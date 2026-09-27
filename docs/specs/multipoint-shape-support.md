# Spec: Multi-point shape support in `draw_shape`

## Status
Draft — proposed by a downstream user, not yet reviewed by a maintainer.

## Problem

`draw_shape` cannot create any drawing that needs more than two anchor points.
`parallel_channel` is the concrete case that surfaced this: TradingView's
native "Parallel Channel" tool takes three points (two for the main
trendline, one for the offset that sets channel width), but calling
`draw_shape` with `shape: "parallel_channel"` silently produces a degenerate
channel with zero width — both stored points collapse to the first
coordinate passed in.

## Root cause

`drawShape()` in `src/core/drawing.js` (lines 10–45) hardcodes a points
array built from exactly two parameters, `point` and `point2`:

```js
export async function drawShape({ shape, point, point2, overrides: overridesRaw, text, _deps }) {
  ...
  if (point2) {
    await evaluate(`
      ${apiPath}.createMultipointShape(
        [{ time: ${p1time}, price: ${p1price} }, { time: ${p2time}, price: ${p2price} }],
        { shape: ${safeString(shape)}, overrides: ${overridesStr}, text: ${textStr} }
      )
    `);
  } else {
    await evaluate(`${apiPath}.createShape(...)`);
  }
  ...
}
```

`createMultipointShape(points, options)` is TradingView's public Charting
Library method and is documented to accept as many points as the requested
shape type requires. The MCP wrapper never passes more than two, so any
shape needing a third (or later) point receives an incomplete array. The
underlying library appears to handle this by collapsing the shape to a
single point rather than erroring, which is why the failure is silent
(`success: true` is still returned) instead of throwing.

The `draw_shape` tool schema (`src/tools/drawing.js`, lines 6–15) reinforces
this: `point` and `point2` are the only coordinate parameters exposed, and
the shape description reads "Shape type: horizontal_line, vertical_line,
trend_line, rectangle, text" — i.e. only 1- and 2-point tools are considered
supported surface area.

## Confirmed affected shape

- `parallel_channel` — needs 3 points. Verified via `draw_get_properties`
  against channels drawn manually in the TradingView UI: their stored
  `points` array has 3 entries, `[mainLineStart, mainLineEnd, widthAnchor]`,
  where `widthAnchor.time === mainLineStart.time`.

## Likely affected shapes (untested, same code path)

Any other multi-point Charting Library tool routed through
`createMultipointShape` would hit the same truncation. Candidates worth
verifying before/while implementing:

- `fib_channel`
- pitchfork family (`pitchfork`, `schiff_pitchfork`, `inside_pitchfork`)
- `head_and_shoulders`
- Elliott wave tools (`elliott_impulse_wave`, `elliott_triangle_wave`, etc.)
- `gann_fan` / `gann_box` variants

## Proposed fix

1. **Schema**: replace the fixed `point` / `point2` pair with a `points`
   array parameter (min 1 item), while keeping `point` / `point2` accepted
   as a backward-compatible alias for the common 1- and 2-point case so
   existing callers don't break.

   ```js
   points: z.array(z.object({ time: z.coerce.number(), price: z.coerce.number() }))
     .min(1)
     .optional()
     .describe('Anchor points in order. Most shapes need 1-2; some (e.g. parallel_channel) need 3+.'),
   ```

2. **Core logic**: build the points array from `points` if provided,
   otherwise fall back to `[point, point2].filter(Boolean)` for backward
   compatibility. Use `createShape` only for the true single-point case;
   route everything else (including today's 2-point shapes) through
   `createMultipointShape` as it already does.

3. **Validation**: before calling into the page, assert the points array
   length is reasonable per shape where known (at minimum: reject 0 points,
   reject a `parallel_channel` call with fewer than 3 points with an
   actionable error instead of silently degrading). A small
   `shape -> requiredPointCount` map (even partial, covering the shapes
   above) turns today's silent corruption into a clear validation error for
   unmapped/underspecified calls.

4. **Docs**: update the `draw_shape` tool description and README's shape
   list to state which shapes need how many points, once verified.

## Non-goals

- Does not change the existing `point` / `point2` behavior for
  `horizontal_line`, `vertical_line`, `trend_line`, `rectangle`, `text` —
  those stay 1- or 2-point and should continue working unmodified.
- Does not attempt to fix `draw_shape`'s handling of an *update* to an
  existing shape's points (separate issue: the internal `LineDataSource`
  multi-point creation flow expects an interactive click sequence to
  finalize; forcing points onto an unfinished shape via internal APIs
  throws `Assertion failed` and corrupts the shape). That's a different,
  harder problem — this spec only covers *creating* a shape correctly the
  first time via the already-used public `createMultipointShape` API.

## Test plan

Following the existing pattern in `tests/sanitization.test.js`:

- `drawShape` with a 3-point `points` array for `parallel_channel` calls
  `createMultipointShape` with all 3 points present in the generated
  expression.
- `drawShape` with legacy `point`/`point2` still produces the same
  2-point `createMultipointShape` call as today (regression check).
- `drawShape` with `shape: "parallel_channel"` and fewer than 3 points
  (via either param style) throws a validation error rather than calling
  into the page.

## Verification

Once implemented, redraw the case that surfaced this: a `parallel_channel`
with the main trendline points `{time: 1788696000, price: 89.74}` →
`{time: 1789333200, price: 83.47}` and a width anchor at
`{time: 1788696000, price: 85.44}`. Confirm via `draw_get_properties` that
all 3 points round-trip correctly (not collapsed to one), and via
screenshot that the channel renders with the expected fill and both rails
visible.
