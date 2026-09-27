# Example: BTC vs. Global Liquidity — A Macro Correlation Framework

**Use case:** macro-driven analysis — comparing Bitcoin's price action
against global liquidity conditions rather than trading purely on
crypto-native technicals.

**Attribution:** this framework is publicly popularized by Benjamin Cowen
(Into The Cryptoverse). His core argument, made repeatedly on public
streams and in published analysis: Bitcoin's price is better explained by
*global net liquidity* — the combined balance sheets of the major central
banks (Fed, ECB, BOJ, PBOC, BOE), minus money parked in the Fed's reverse
repo facility and the U.S. Treasury General Account — than by raw M2 money
supply figures alone. Cowen has pointed out that M2 hit record highs in
2014, 2018, and 2022, yet Bitcoin suffered major drawdowns in each of those
years, which is why he distinguishes "M2" from "net liquidity actually
flowing into markets." This example demonstrates the general workflow for
building that kind of comparison in TradingView — it does not reproduce any
of his specific paid tools or proprietary indicators.

## Workflow

```
# 1. Load BTC on a macro-appropriate timeframe
chart_set_symbol({ symbol: "BITSTAMP:BTCUSD" })
chart_set_timeframe({ timeframe: "1W" })

# 2. Add a liquidity proxy as a comparison overlay. TradingView hosts
#    public FRED (Federal Reserve Economic Data) series directly as
#    symbols — e.g. FRED:WM2NS (US M2, weekly) or FRED:WALCL (Fed
#    balance sheet). A true "global net liquidity" series (multi-central-
#    bank, netted against RRP/TGA) isn't a single public ticker — it's
#    typically built by combining several FRED/central-bank series, which
#    is exactly why Cowen's version is a distinct, hand-built indicator
#    rather than a stock chart overlay.
chart_manage_indicator({
  action: "add",
  indicator: "FRED:WALCL" # Fed balance sheet, as an available starting proxy
})

# 3. Pull both series and eyeball the lag/lead relationship — Cowen's
#    public commentary generally treats liquidity as leading price by
#    several weeks, not moving in lockstep
data_get_ohlcv({ summary: true })
data_get_study_values({})

# 4. Mark divergence points where BTC and the liquidity proxy visibly
#    decouple — these are the moments worth investigating further
draw_shape({
  shape: "vertical_line",
  point: { time: <unix_timestamp> },
  text: "Liquidity/price divergence"
})

# 5. Screenshot for reference / logging
capture_screenshot({ region: "chart" })
```

## What to log

```
## 2026-02-10 08:00 — BTCUSD vs FRED:WALCL / 1W
**Setup:** BTC price flat-to-down over 8 weeks while the liquidity proxy
has been expanding — a potential lag before price catches up, per the
general framework.
**Condition:** Continued liquidity expansion with BTC price starting to
turn up would confirm the lag thesis; continued BTC weakness despite
expansion would argue liquidity isn't the dominant driver right now.
**Invalidation:** A reversal in the liquidity proxy itself removes the
basis for the thesis regardless of BTC price action.
**Note:** This is a macro overlay, not a timing tool — treat it as context
for a multi-week view, not an entry signal.
```

## Notes

- The exact timestamps/levels above are placeholders — this file
  demonstrates the workflow, not a live analysis.
- A proper "global net liquidity" index (multiple central banks, netted
  against RRP/TGA) requires combining several data series; this example
  uses a single public FRED series as an accessible starting point, not a
  full reproduction of Cowen's specific methodology.
- For further public reading on this framework, search "Benjamin Cowen
  global liquidity Bitcoin" — his Into The Cryptoverse channel and
  associated publications cover the methodology in detail.
