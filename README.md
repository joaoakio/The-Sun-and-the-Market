# Sunspots & Equity Returns

This repository investigates whether **monthly sunspot activity** contains **incremental predictive information** for **next-month S&P 500 returns**, **conditional on standard financial controls**.

> **Math rendering note (GitHub):** equations in this README use `$...$` (inline) and `$$...$$` (display), which GitHub renders natively. :contentReference[oaicite:1]{index=1}

---

## Research question and timing convention

Let $P^{EOM}_t$ denote the S&P 500 level at the **end of month** $t$ (last trading day). Define the **next-month log return**:

$$
r_{t+1} = \log\!\left(P^{EOM}_{t+1}\right) - \log\!\left(P^{EOM}_{t}\right).
$$

All predictors are measured with information available **at the end of month $t$** and are used to forecast **$r_{t+1}$** (strict $t \rightarrow t+1$ timing). This convention is enforced throughout to avoid look-ahead bias.

---

## Models (nested pair used everywhere)

We estimate and compare the same **nested pair** of predictive regressions:

**Model A (controls only):**
$$
r_{t+1} = \alpha + \sum_{j=1}^{N}\beta_j X_{j,t} + \varepsilon_{t+1}.
$$

**Model B (controls + sunspots):**
$$
r_{t+1} = \alpha + \sum_{j=1}^{N}\beta_j X_{j,t} + \gamma\,SN_t + \varepsilon_{t+1}.
$$

The key quantity is **$\gamma$**, the incremental contribution of sunspots after controlling for $X_{j,t}$.

**Inference:** OLS with **HAC / Newey–West** standard errors (monthly returns can be heteroskedastic and mildly autocorrelated).

---

## Data sources and variables

### Market data (Yahoo Finance via `yfinance`)
- S&P 500 index: `^GSPC` (daily prices → EOM levels → monthly log returns)
- VIX: `^VIX` (EOM level)
- 10-year yield proxy: `^TNX` (EOM level)
- 13-week T-bill proxy: `^IRX` (EOM level)

### Sunspots (SILSO, monthly series)
- Monthly mean total sunspot number (SILSO “SN_m_tot_V2.0” monthly series)

### Predictors (all dated at $t$, used to forecast $r_{t+1}$)
- `ret_m_l1`: previous month log return ($r_t$)
- `rv_m_l1`: realized volatility within month $t$ from daily S&P log returns  
  $$
  rv_t=\sqrt{\sum_{d\in t} r_d^2}
  $$
- `vix_l1`: VIX level at EOM $t$
- `tbill_l1`: short-rate proxy at EOM $t$ (from `^IRX`)
- `term_slope_l1`: term slope at EOM $t$ (from `^TNX - ^IRX`)
- `sn_lag1` (used as $SN_t$): sunspots in month $t$ (aligned to forecast $t{+}1$)

---

## Methodology (what the notebook does)

1. **Data & alignment:** build an end-of-month panel and enforce strict $t \rightarrow t+1$ timing.
2. **Initial exploration:** plots, correlations, and scatter checks (sanity + intuition, not proof).
3. **In-sample regressions:** estimate Model A and Model B with HAC standard errors.
4. **Stability diagnostics:** subsample splits and rolling-window estimates of $\gamma$ and its HAC $p$-value.
5. **Out-of-sample forecasting:** expanding-window test comparing Model A vs Model B using MSFE and OOS $R^2$:
   $$
   R^2_{\text{OOS}} = 1 - \frac{\text{MSFE}_B}{\text{MSFE}_A}.
   $$
   Plus a **Clark–West** test for nested models.
6. **Allocation lens:** translate OOS forecasts from Model B into a simple monthly long/cash rule and compare to buy-and-hold.

---

## Results snapshot (from the current run)

### In-sample (HAC / Newey–West)
- Estimated $\gamma$ (sunspots) is **positive**, but **not statistically robust** at conventional levels (HAC $p$-value above 5% in the full sample).

### Stability
- Subsample split suggests **instability**: $\gamma$ changes sign/magnitude across eras.
- Rolling estimates show $\gamma$ and its HAC $p$-value vary materially over time (limited persistence).

### Out-of-sample forecasting
- Adding sunspots **does not improve** forecast accuracy:
  - $\text{MSFE}_B > \text{MSFE}_A$
  - $R^2_{\text{OOS}} < 0$ (worse than controls-only)
  - Clark–West test is **not supportive** (one-sided $p$ not small)

### Allocation lens
- A simple monthly long/cash rule driven by the augmented forecast **does not outperform** buy-and-hold over the evaluated OOS window (lower annualized mean log return and Sharpe in the current run).

---

## Interpretation (practical takeaway)

Under the chosen specification, sunspots do **not** show reliable **incremental predictive value** beyond standard controls, especially where it matters most: **out-of-sample performance**. A reasonable decision is to treat sunspots as an **exploratory feature** rather than a standalone signal, unless additional robustness checks (pre-registered hypotheses, alternative assets, structural breaks, economic mechanism) strengthen the evidence.

---




