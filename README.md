# Sunspots & Equity Returns (Out-of-sample test)

This repository contains a self-contained notebook that evaluates whether **monthly sunspot activity** provides **incremental predictive information** for **next-month S&P 500 returns**, **conditional on** a standard set of financial controls.

The focus is not on storytelling or in-sample significance alone, but on **out-of-sample (OOS) forecasting performance** under a clean timing convention (strict lag; no look-ahead).

---

## Research question

Do sunspots add incremental forecasting power for next-month equity returns once we control for standard market predictors?

We test the nested predictive regression:

- **Model A (controls only):**  
  \( r_{t+1} = \alpha + \sum_{j=1}^N \beta_j X_{j,t} + \varepsilon_{t+1} \)

- **Model B (controls + sunspots):**  
  \( r_{t+1} = \alpha + \sum_{j=1}^N \beta_j X_{j,t} + \gamma\,SN_t + \varepsilon_{t+1} \)

**Incremental value** is assessed primarily via **OOS performance** (MSFE, OOS \(R^2\), Clark–West).

---

## Data sources

- **S&P 500 index**: Yahoo Finance ticker `^GSPC` (daily prices aggregated to end-of-month).
- **Controls** (Yahoo Finance):
  - `^VIX` (VIX level)
  - `^TNX` (10-year yield proxy)
  - `^IRX` (13-week T-bill proxy)
- **Sunspots**: SILSO monthly series (Sunspot Index and Long-term Solar Observations).

> Notes: Yahoo Finance is used for prototyping convenience. SILSO data is under CC BY-NC 4.0; please cite SILSO if you reuse the sunspot series.

---

## Method overview (what the notebook does)

1. **Data & alignment (strict lag):** predictors at month \(t\) are used to forecast \(r_{t+1}\).
2. **Initial exploration:** time-series plots, simple correlations, scatter.
3. **In-sample regressions:** OLS with **HAC / Newey–West** standard errors.
4. **Stability diagnostics:** subsample splits and rolling-window estimates of \(\gamma\) (plus rolling HAC p-values).
5. **Out-of-sample test:** expanding-window forecasts comparing Model A vs Model B using:
   - MSFE
   - OOS \(R^2 = 1 - \text{MSFE}_B / \text{MSFE}_A\)
   - Clark–West test (nested-model adjustment)
6. **Allocation lens (simple):** monthly long/cash rule based on OOS forecasts (illustrative “investable lens”).

---

## Key results (as implemented)

- **In-sample:** \(\gamma\) tends to be positive but is not robust at conventional HAC significance levels.
- **Stability:** \(\gamma\) is not stable across subsamples/rolling windows.
- **Out-of-sample:** adding sunspots does **not** improve forecast accuracy (OOS \(R^2\) is negative; Clark–West not supportive).
- **Allocation simulation:** a simple forecast-driven long/cash rule does not beat buy-and-hold over the tested OOS window.

**Conclusion:** within this dataset and model design, sunspots appear to be an exploratory feature at best and do not deliver reliable incremental predictive value beyond standard controls.

