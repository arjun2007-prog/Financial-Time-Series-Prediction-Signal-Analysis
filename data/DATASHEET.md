# Datasheet: `cleaned_data_with_returns.csv`

## 1. Dataset Overview

| | |
|---|---|
| **Instrument** | Nifty 50 Index |
| **Frequency** | Daily (trading days only) |
| **Date range** | 2014-01-02 to 2024-12-30 |
| **Rows** | 2,670 |
| **Columns** | 8 |
| **Granularity** | One row = one trading day, index-level OHLCV |
| **Missing values** | None (0 nulls across all columns) |

## 2. Schema

| Column | Type | Unit | Description | Derived? |
|---|---|---|---|---|
| `Date` | string (ISO 8601, `YYYY-MM-DD`) | — | Trading date | No — source field |
| `Open` | float64 | index points | Opening level of the index for the day | No — source field |
| `High` | float64 | index points | Highest level during the day | No — source field |
| `Low` | float64 | index points | Lowest level during the day | No — source field |
| `Close` | float64 | index points | Closing level of the index for the day | No — source field |
| `Volume` | int64 | shares/contracts traded | Daily traded volume | No — source field |
| `simple_return_o2c` | float64 | unitless (ratio) | Same-day open-to-close simple return | **Yes** |
| `log_return_o2c` | float64 | unitless (nats) | Same-day open-to-close log return | **Yes** |

## 3. Derived Field Definitions

**`simple_return_o2c`** (arithmetic/simple return, open-to-close):
```
simple_return_o2c_t = (Close_t / Open_t) - 1
```

**`log_return_o2c`** (continuously-compounded/log return, open-to-close):
```
log_return_o2c_t = ln(Close_t / Open_t)
```

Both are computed **within the same trading day** — they do not reference any prior-day value, so every row has a valid, non-null return (no leading NaN, unlike close-to-close returns).

## 4. Relationship Between the Two Return Columns

```
log_return_o2c = ln(1 + simple_return_o2c)
simple_return_o2c = exp(log_return_o2c) - 1
```
Values are nearly identical for small moves and diverge slightly on large-move days (log return is always ≤ simple return in magnitude for positive moves, and the asymmetry grows with move size).

## 5. Summary Statistics (`o2c` returns, full sample)

| Statistic | `simple_return_o2c` | `log_return_o2c` |
|---|---|---|
| Mean | -0.000705 | -0.000741 |
| Std dev | 0.008491 | 0.008486 |
| Min | -0.068180 | -0.070616 |
| Max | 0.093065 | 0.088986 |

## 6. Known Characteristics / Caveats

- **Index-level data**: no split/dividend adjustment needed (unlike individual constituent stocks).
- **No missing rows within trading days**: gaps in `Date` correspond only to weekends/NSE holidays, not data quality issues (assumed pre-validated as "cleaned").
- **Extreme values**: max/min single-day moves (~9%) likely correspond to known volatility events (e.g., COVID-19 crash, Mar 2020) — not treated as outliers or removed.
- **Return sign convention**: positive = index closed higher than it opened that day; negative = index closed lower than it opened.
- **Scope of `o2c` returns**: captures only the *intraday* session move. It does **not** capture the overnight gap (prior close → today's open). If overnight or full close-to-close returns are needed, they are separate derived fields not included in this file.

