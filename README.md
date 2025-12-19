
# Sunspots & Equity Returns (Monthly, Predictive Regression + OOS Tests)

This repository contains a research notebook that evaluates whether **monthly sunspot activity** contains **incremental predictive information** for **next-month** S&P 500 returns, **conditional on** a set of standard financial controls.

> **One-sentence summary:** After controlling for standard predictors, sunspots show **weak / unstable** in-sample association and **do not improve** out-of-sample forecasts in this implementation.

---

## Research question and timing convention

Let $P^{EOM}_t$ be the S&P 500 level at the **end of month $t$** (last trading day). Define the **next-month log return**:

```math
r_{t+1} \equiv \log(P^{EOM}_{t+1}) - \log(P^{EOM}_{t}).
```

All predictors are measured using information available **at the end of month $t$** and are used to forecast **$r_{t+1}$** (strict $t \rightarrow t+1$ timing, no look-ahead).

The core predictive regression is:

```math
r_{t+1} = \alpha + \sum_{j=1}^{N}\beta_j X_{j,t} + \gamma\,SN_t + \varepsilon_{t+1},
```

where $X_{1,t},\ldots,X_{N,t}$ are financial controls observed at $t$, and $SN_t$ is the sunspot measure for month $t$.

**Incremental value** means: does adding $SN_t$ improve forecasting performance beyond the control set ${X_{j,t}}_{j=1}^{N}$? In practice, we prioritize **out-of-sample** metrics (MSFE, OOS $R^2$, Clark–West), not just in-sample $p$-values.

---

## Data sources

* **Equities:** S&P 500 index level from Yahoo Finance (`^GSPC`), sampled daily and resampled to end-of-month.
* **Sunspots:** SILSO “Monthly mean total sunspot number” (SN_m_tot_V2.0).
* **Controls (Yahoo Finance proxies):**

  * `^VIX` (VIX)
  * `^TNX` (10Y yield proxy)
  * `^IRX` (13-week T-bill proxy)

**Sunspots data details (SILSO):**

* Monthly mean is the arithmetic mean of daily totals within each calendar month.
* Missing values may appear as `-1` and are treated as missing.
* A provisional marker may apply to the most recent months.
* License: **CC BY-NC 4.0** (attribution required; non-commercial).
  (See SILSO documentation in the notebook/comments.)

---

## Variables (controls and signal)

All predictors are measured at month $t$ and used to forecast $r_{t+1}$.

**Controls ($X_{j,t}$):**

* `ret_m_l1` : previous month log return $r_t$ (short-horizon dynamics).
* `rv_m_l1`  : realized volatility within month $t$ computed from daily log returns
  $rv_t = \sqrt{\sum_{d\in t} r_d^2}$ (risk regime proxy).
* `vix_l1`   : VIX end-of-month level (implied volatility / risk appetite).
* `tbill_l1` : short-rate proxy from `^IRX` (cash/discounting environment).
* `term_slope_l1` : term slope proxy $(TNX_t - IRX_t)$ (macro/curve regime).

**Signal ($SN_t$):**

* `sn_lag1` : monthly sunspot number aligned so that sunspots observed in month $t$ are used to forecast $r_{t+1}$.

---

## Methodology overview

### 1) In-sample predictive regressions (HAC / Newey–West)

We estimate one nested pair, aligned to strict $t \rightarrow t+1$ timing:

**Model A (controls only):**

```math
r_{t+1} = \alpha + \sum_{j=1}^{N}\beta_j X_{j,t} + \varepsilon_{t+1}.
```

**Model B (controls + sunspots):**

```math
r_{t+1} = \alpha + \sum_{j=1}^{N}\beta_j X_{j,t} + \gamma\,SN_t + \varepsilon_{t+1}.
```

We report **HAC / Newey–West** standard errors (monthly data can be heteroskedastic and mildly autocorrelated). The coefficient of interest is $\gamma$.

### 2) Stability diagnostics

A credible predictive relationship should be **stable** across time. We evaluate stability via:

* coarse **subsample splits** (e.g., first vs second half), and
* a **rolling window** estimate of $\gamma$ (and its HAC $p$-value) to visualize regime dependence.

### 3) Out-of-sample (OOS) forecast comparison

We use an **expanding-window** procedure to compare one-step-ahead forecasts from Model A vs Model B.

We evaluate:

* MSFE for each model, and
* **OOS $R^2$**:

```math
R^2_{\text{OOS}} = 1 - \frac{\text{MSFE}_{B}}{\text{MSFE}_{A}}.
```

We also report the **Clark–West** test (HAC) for nested model comparison.

### 4) Allocation “investable lens” (simple monthly rule)

We translate the **OOS** augmented forecast into a simple rule:

* decision made at end of month $t$ using $\hat r_{t+1\mid t}$
* position held during month $t+1$

```math
pos_t = \mathbf{1}\{\hat r_{t+1\mid t} > 0\}, \qquad
r^{strat}_{t+1} = pos_t \cdot r_{t+1}.
```

Because the notebook uses **log returns**, wealth is computed as:

```math
W_T = \exp\left(\sum_{t\le T} r_t\right),
```

normalized to start at 1. This section is intentionally minimal (no trading frictions) to ask whether the signal is strong enough to show up under a basic implementation.

---

## Results (from the current notebook run)

**In-sample (Model B):**

* $\hat\gamma$ is positive but **not statistically robust** under HAC (above conventional 5% thresholds).

**Stability:**

* Subsample estimates of $\gamma$ show **sign / magnitude drift** across eras (suggesting regime dependence rather than a persistent effect).

**Out-of-sample:**

* Adding sunspots yields **negative OOS $R^2$** and Clark–West is **not supportive**, indicating **no incremental forecast benefit** relative to controls alone.

**Allocation lens:**

* The forecast-driven monthly rule does **not** outperform buy-and-hold over the evaluated OOS window in this implementation.

> Interpretation: Under this specification and dataset, sunspots look like an **exploratory feature** rather than a standalone signal. Any claim of predictability would require stronger OOS evidence and stability.

---

## License / attribution

* Sunspot data: SILSO (CC BY-NC 4.0). You must provide attribution and use non-commercially.
* Market data: Yahoo Finance via `yfinance` (used for prototyping / educational analysis).

````




