# Trade Log

Interactive setups and alerts logged during TradingView agent sessions.
Not a trade journal — no P&L, no sizing. Ideas and conditions only.

**This is a template.** Copy it to `trade-log.md` (already gitignored — see
`.gitignore`) and let your agent append real entries there. Everything below
is synthetic/illustrative, not a real trade.

---

## 2026-01-15 09:30 — EXAMPLE:BTCUSD / 4H

**Setup:** Ascending triangle forming since the 01-10 low, resistance flat around 45,200, rising trendline support currently near 43,800.
**Condition:** Breakout and hold above 45,200 on volume targets the prior swing high near 48,000; a close back below the rising trendline invalidates the pattern.
**Invalidation:** A 4H close below 43,500 breaks the ascending structure entirely.
**Note:** This entry demonstrates the standard format — one-line setup, a clear trigger condition, and an explicit invalidation level. Real entries should be this specific.

---

## 2026-01-18 14:05 — EXAMPLE:ETHUSD / 1D

**Setup:** Descending channel since the 01-05 peak; testing lower channel boundary for the third time.
**Condition:** A reclaim of the mid-channel line reopens the upper boundary; a clean break below the lower boundary on the third test often signals trend continuation.
**Invalidation:** A daily close back above the channel's upper boundary invalidates the descending bias.
**Note:** Shows how to reference chart-drawing context (a channel, in this case) without needing exact price levels if the setup is more about structure than specific numbers.

---

## How to use this template

1. Copy this file to `trade-log.md` in the repo root (matches `.gitignore`, stays local-only)
2. Ask your agent to log setups as you work through charts — see `CLAUDE.md`'s "Logging setups" section for the exact format contract
3. Delete these two example entries once you have real ones
