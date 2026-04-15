# Partial-Moments-Momentum-Trading-Strategy
Implemented PMM strategy (Gao, Leung, Satchell 2022)
# Partial Moment Momentum

A replication of the trading strategy from Gao, Leung and Satchell (2022), "Partial Moment Momentum", Journal of Banking and Finance 135.

---

## What this is

Standard momentum strategies buy past winners and sell past losers. They work, but they treat upside and downside volatility as the same thing — which means they get hit hard during market crashes. This paper proposes a fix: weight the long and short legs differently based on whether recent market variance came from up days or down days.

The result is a strategy that holds more of the winner portfolio when the market has been trending up, and more of the loser portfolio when it has been trending down — dynamically adjusting before momentum crashes rather than reacting after.

---

## The strategies

**WML** — plain 11x1 momentum. Buy decile 10 (winners), short decile 1 (losers), hold one month, skip the most recent month to avoid reversal.

**VM** — volatility-managed momentum from Barroso and Santa-Clara (2015). Scales both legs equally by realized variance so the portfolio targets a fixed annualized volatility of 12%.

**PMM** — the paper's main contribution. Splits realized variance into upside (RPM+) and downside (RPM-) partial moments and uses that split to assign unequal weights to the long and short legs:

```
phi1 = 2 * sigma_tar / sqrt(RV_t) * RPM+ / RV_t    (long weight)
phi2 = 2 * sigma_tar / sqrt(RV_t) * RPM- / RV_t    (short weight)
```

**PMMC** — same as PMM but caps total leverage at 200%.

---

## Data

All sourced from the Kenneth R. French Data Library via `pandas_datareader`:

| Dataset | Used for |
|---|---|
| `10_Portfolios_Prior_12_2` | Monthly winner / loser decile returns |
| `F-F_Research_Data_Factors_daily` | Daily market returns for RPM computation |
| `F-F_Research_Data_Factors` | Monthly RF, market factor, SMB, HML |
| `F-F_Research_Data_5_Factors_2x3` | Five-factor spanning tests |

Sample period: July 1927 to December 2018.

---

## Results

| Strategy | Ann. Return | Sharpe | MDD |
|---|---|---|---|
| WML | 14.17% | 0.52 | -95.60% |
| VM | 17.19% | 0.86 | -46.13% |
| PMM | 18.82% | 0.89 | -40.74% |
| PMMC | 15.42% | 0.84 | -40.74% |

PMM improves on both plain momentum and volatility-managed momentum across return, Sharpe, skew, and drawdown. The numbers sit slightly below the paper's reported figures (PMM SR 1.23) because the replication uses pre-formed French decile portfolios rather than the individual CRSP stock data used in the original.

---

## Notebook structure

| Cell | Contents |
|---|---|
| 1 | Imports and global constants |
| 2 | Data download and parquet cache |
| 3 | Logger utility |
| 4 | EDA: momentum deciles |
| 5 | EDA: daily market returns |
| 6 | WML portfolio construction |
| 7 | RPM+/RPM- computation (Numba JIT) |
| 8 | VM, PMM, PMMC strategy returns |
| 9 | Performance table (replicates Table 1) |
| 10 | Market state analysis (replicates Table 2) |
| 11 | Dynamic leverage plots (replicates Fig 1 and Fig 3) |
| 12 | Factor regressions (replicates Tables 7 and 8) |
| 13 | Spanning tests (replicates Table 10) |
| 14 | Summary and cumulative performance chart |

---

## Setup

```bash
conda activate base
pip install pandas numpy numba numexpr joblib pandas_datareader matplotlib seaborn scipy
jupyter notebook pmm_strategy.ipynb
```

The notebook caches all downloaded data to `cache/` on first run. Subsequent runs skip the download.

---

## Reference

Gao, Y., Leung, H., Satchell, S. (2022). Partial moment momentum. *Journal of Banking and Finance*, 135, 106361.
