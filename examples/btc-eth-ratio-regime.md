# Example: BTC/ETH Ratio as a Market Regime Indicator

**Use case:** relative-strength analysis — tracking which of the two largest
crypto assets is leading, using the BTC/ETH ratio as a regime signal.

This is a widely-used public technique, not a proprietary strategy: the
BTC/ETH ratio is one of the most-watched relative-value charts in crypto.
Rising BTC/ETH means Bitcoin is outperforming Ethereum (a "Bitcoin-led"
regime); falling BTC/ETH means Ethereum is outperforming (often associated
with broader altcoin strength). The 60-day rolling correlation between the
two assets has spent most of the post-2020 era above 0.85, with notable
drops during ETH-specific catalysts (the Merge, ETF approval news) — those
correlation breaks are often where the ratio's trend actually shifts.

## Workflow

```
# 1. Load the ratio directly — TradingView supports symbol division
chart_set_symbol({ symbol: "BINANCE:BTCUSD/BINANCE:ETHUSD" })
chart_set_timeframe({ timeframe: "1D" })

# 2. Pull recent price action for the ratio itself
data_get_ohlcv({ summary: true })

# 3. Add RSI on the ratio — a common way to gauge overbought/oversold
#    regime extremes rather than raw price extremes
indicator_add({ indicator: "Relative Strength Index" })
data_get_study_values({})
# RSI > 70 on the ratio: BTC richly outperforming, pullback (ETH catch-up) more likely
# RSI < 30 on the ratio: ETH richly outperforming, BTC catch-up more likely

# 4. Mark a regime boundary once you've identified one visually
draw_shape({
  shape: "horizontal_line",
  point: { time: <unix_timestamp>, price: <ratio_level> },
  text: "Regime boundary"
})

# 5. Screenshot for reference / logging
capture_screenshot({ region: "chart" })
```

## What to log

If this setup informs a real decision, log it in your local `trade-log.md`
(gitignored — see `trade-log.example.md` for the template):

```
## 2026-03-01 10:00 — BTCUSD/ETHUSD / 1D
**Setup:** Ratio RSI diverging from price at a multi-month high, testing
prior regime boundary.
**Condition:** A clean break and hold above the boundary confirms
continued BTC-led regime; rejection favors rotation toward ETH.
**Invalidation:** A close back inside the prior range invalidates the
regime call.
**Note:** Correlation context matters — check for any ETH-specific
catalyst (network upgrade, ETF news) that could be driving a temporary
correlation break rather than a genuine regime shift.
```

## Notes

- This example demonstrates the *workflow*, not a specific trade call — the
  timestamps/levels above are placeholders, not real analysis.
- The BTC/ETH ratio is public market data; no proprietary indicator or
  paid script is required to reproduce this.
