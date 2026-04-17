# Rainman Signal Fusion

Multi-source market signal fusion engine. Aggregates macro, price, geopolitical, and conflict data from four independent feeds, synthesizes them with Claude AI, and sizes positions using Kelly criterion.

---

## Signal Sources

| Source | Data | Cadence |
|--------|------|---------|
| FRED | Macroeconomic indicators (CPI, unemployment, yield curve, M2) | Daily |
| Finnhub | Price feeds, earnings, analyst ratings, news sentiment | Real-time |
| GDELT | Global event database - geopolitical events, media tone | Daily |
| ACLED | Armed conflict location and event data | Weekly |

---

## Architecture

```
Signal Collectors (one per source)
    |
    v
Signal Normalizer (common schema, z-score normalization)
    |
    v
Claude Haiku (fast per-signal scoring: bullish / neutral / bearish + confidence)
    |
    v
Claude Opus (cross-signal synthesis: macro regime, risk-on/off, key contradictions)
    |
    v
Kelly Criterion Sizer (position size as fraction of portfolio)
    |
    v
Output: signal report + sizing recommendation
```

---

## AI Orchestration

Two-model design keeps costs and latency low:

- **Claude Haiku**: Scores each signal independently. Fast, cheap, runs in parallel across all sources.
- **Claude Opus**: Reads all Haiku scores plus raw context, identifies the macro regime, resolves contradictions between signals, and produces the final synthesis.

The Opus synthesis prompt is constrained: it must cite which signals conflict, explain which it weights more heavily and why, and flag its own uncertainty explicitly.

---

## Position Sizing

Uses fractional Kelly criterion:

```
f* = (p * b - q) / b   (full Kelly)
position = f* * kelly_fraction   (fractional Kelly, default 0.25)
```

Where `p` = estimated win probability from synthesis confidence, `b` = expected payoff ratio from price targets, `q` = 1 - p.

Full Kelly is never used directly. The fractional multiplier (default 0.25) limits drawdown from model error.

---

## Status

Under active development. Signal collectors and normalization layer in progress.
