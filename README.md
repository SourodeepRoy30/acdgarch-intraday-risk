# Intensity-Adjusted Intraday Volatility and Market Risk Forecasting
### An ACD-GARCH Framework on High-Frequency BP plc Tick Data

[![Language](https://img.shields.io/badge/language-R-276DC3?style=flat-square)](https://www.r-project.org/)
[![Data](https://img.shields.io/badge/tick%20quotes-1.27M-blue?style=flat-square)]()
[![Dissertation](https://img.shields.io/badge/MSc%20Dissertation-Distinction%2080%25-gold?style=flat-square)]()

---

## Overview

This repository contains the full research code for my MSc Data Science and Analytics dissertation at the **University of Leeds** (Distinction, 80%).

The central question is: **does adjusting high-frequency returns for trading activity intensity produce better out-of-sample volatility and risk forecasts?**

Standard GARCH models treat all time intervals equally. But in high-frequency markets, a 5-minute window containing 30 trades carries fundamentally different information than one containing 3 trades. The **Autoregressive Conditional Duration (ACD)** model captures this variation in trading intensity, and we use it to construct *intensity-adjusted returns* — a denoised return series that isolates genuine price volatility from activity-driven noise.

We then fit GARCH models on both the raw and intensity-adjusted series, compare out-of-sample volatility forecasts, and evaluate the resulting VaR and ES risk estimates against realised returns using formal statistical tests.

---

## Key Results

All models trained on **April 2024 only** and evaluated out-of-sample on **May and June 2024**.

### Volatility Forecasting

| Metric | May | June |
|---|---|---|
| QLIKE improvement (IA vs Raw) | **+4.96%** | **+5.94%** |
| MSE improvement (IA vs Raw) | **+75.7%** | **+80.4%** |
| DM-HAC t-statistic (QLIKE) | **33.96** | **31.80** |
| DM p-value | < 0.001 | < 0.001 |
| Win rate (% of 5-min buckets where IA has lower loss) | 96.6% | 97.2% |

The intensity-adjusted model achieves lower QLIKE loss in over **96% of individual 5-minute intervals** across both test months. The Diebold-Mariano test with Newey-West HAC standard errors confirms this improvement is statistically significant well beyond conventional levels.

### Market Risk (VaR/ES Backtesting)

| Test | Result |
|---|---|
| Kupiec unconditional coverage | Both models conservative; formally rejected at 1% |
| Christoffersen independence | **Passes for all specifications** — no clustering of breaches |
| FHS ES cover ratio (5%) | **1.01–1.06** — accurate conditional tail estimation |
| FZ0 DM-HAC t-statistic (1%) | **36.4** (May+Jun combined, p < 0.001) |
| FZ0 DM-HAC t-statistic (5%) | **16.3** (May+Jun combined, p < 0.001) |
| FZ0 win rate at 1% | **90–97%** of 5-min intervals |

The IA-mapped model produces lower FZ0 joint VaR-ES loss in all 12 evaluation configurations. The conservative VaR coverage is a consequence of the single-month training design (April was more volatile than May-June), not a structural model failure — confirmed by the independence tests passing throughout.

---

## Methodology

### Data
- **Asset:** BP plc ordinary shares (BP.L), London Stock Exchange
- **Source:** Refinitiv Tick History
- **Period:** April–June 2024
- **Volume:** 1,270,000+ individual tick quotes
- **Cleaning:** Zero-return filtering, overnight gap removal, exchange hours restriction (08:00–16:30 BST)

### Step 1 — ACD Model (Notebooks 02–04)
Trade durations (time between successive transactions) are modelled using a **Weibull ACD(2,2)** specification. The ACD model captures the intraday U-shaped pattern in trading activity (high at open and close, low at midday) and produces a conditional intensity measure ψ_t for each trade.

### Step 2 — Intensity-Adjusted Returns (Notebook 05)
Raw tick returns are scaled by the ACD intensity to produce two adjusted series:

- **IA-sum:** event-level adjusted returns aggregated to 5-minute buckets (used for volatility track)
- **IA-inv:** raw 5-minute returns multiplied by √ψ (invertible; used for risk track)

The adjustment reduces the activity-volatility correlation by **31–44%** at the 5-minute level.

### Step 3 — GARCH Estimation (Notebook 06)
Three GARCH specifications (sGARCH, eGARCH, GJR-GARCH) with Student-t innovations are fitted on **April only**. Model selection via BIC selects **sGARCH(1,1)-t** for both tracks. April parameters are frozen and applied to May and June using `ugarchfilter()` — no look-ahead leakage. Forecast quality evaluated via QLIKE, MSE, Mincer-Zarnowitz regression, and DM-HAC tests.

### Step 4 — VaR/ES Backtesting (Notebook 07)
Risk forecasts constructed at two levels:

- **1% parametric:** analytical Student-t multipliers applied to frozen-parameter GARCH sigma
- **5% FHS (Filtered Historical Simulation):** empirical quantile of April standardised residuals scaled by OOS sigma (McNeil & Frey 2000)

IA-model VaR/ES forecasts are mapped back to the raw return scale via the invertible ACD scaling: VaR_raw = VaR_IA / √ψ.

Evaluation tests: **Kupiec (1995)** unconditional coverage, **Christoffersen (1998)** independence and conditional coverage, **FZ0 joint loss** (Fissler & Ziegel 2016), **DM-HAC** (Diebold & Mariano 1995 with Newey-West HAC), and **Clopper-Pearson** binomial confidence intervals.

---

## Repository Structure

```
acdgarch-intraday-risk/
│
├── data/
│   ├── raw/                    # Original tick data (not tracked — too large)
│   └── processed/              # Intermediate .rds files (not tracked — reproducible from code)
│
├── notebooks/
│   ├── 01_data_loading.Rmd           # Load and inspect raw tick data
│   ├── 02_data_cleaning.Rmd          # Filter, clean, align tick quotes
│   ├── 03_diurnal_adjustment.Rmd     # Intraday seasonality removal
│   ├── 04_acd_model.Rmd              # Weibull ACD(2,2) estimation
│   ├── 05_intensity_adjustment.Rmd   # Construct IA return series
│   ├── 06_garch_volatility_models.Rmd # GARCH fitting, OOS forecasting, QLIKE/MSE
│   └── 07_var_es_backtesting.Rmd     # VaR/ES construction, backtesting, FZ0
│
└── README.md
```

Notebooks must be run **in order (01 → 07)**. Each notebook saves its outputs to `data/processed/` for the next notebook to load.

---

## Reproducing the Results

### Requirements

R version 4.3 or later. Install required packages:

```r
install.packages(c(
  "data.table",   # fast data manipulation
  "ggplot2",      # visualisation
  "rugarch",      # GARCH estimation
  "ACDm",         # ACD model estimation
  "sandwich",     # Newey-West HAC standard errors
  "lmtest",       # coeftest with robust SEs
  "zoo",          # rolling window functions
  "rprojroot"     # project-relative file paths
))
```

### Running

1. Clone the repository:
```bash
git clone https://github.com/SourodeepRoy30/acdgarch-intraday-risk.git
cd acdgarch-intraday-risk
```

2. Place the raw tick data file in `data/raw/` (see Data note below).

3. Open the project in RStudio by double-clicking `acdgarch-intraday-risk.Rproj`.

4. Run notebooks 01 through 07 in order. Each notebook uses `rprojroot::find_rstudio_root_file()` to anchor file paths — no manual path editing required.

> **Data note:** The raw tick data was sourced from Refinitiv Tick History via the University of Leeds library subscription and cannot be redistributed. Contact the author for details on obtaining equivalent BP.L tick data.

---

## Statistical Tests Reference

| Test | Purpose | Reference |
|---|---|---|
| Weibull ACD(2,2) | Model trade duration dynamics | Engle & Russell (1998) |
| ARCH-LM (Ljung-Box on r²) | Pre-GARCH heteroskedasticity check | Engle (1982) |
| Mincer-Zarnowitz regression | Forecast efficiency (α=0, β=1) | Patton (2011) |
| QLIKE loss | Volatility forecast comparison | Patton (2011) |
| Diebold-Mariano (HAC) | Forecast superiority test | Diebold & Mariano (1995) |
| Kupiec LR test | Unconditional VaR coverage | Kupiec (1995) |
| Christoffersen LR test | VaR hit independence | Christoffersen (1998) |
| FZ0 loss | Joint VaR-ES evaluation | Fissler & Ziegel (2016) |
| Filtered Historical Simulation | Non-parametric tail estimation | McNeil & Frey (2000) |
| Clopper-Pearson CI | Binomial hit-rate confidence interval | Clopper & Pearson (1934) |

---

## Author

**Sourodeep Roy**  
MSc Data Science and Analytics — University of Leeds (Distinction, 80%)  
[GitHub](https://github.com/SourodeepRoy30) · [LinkedIn](https://www.linkedin.com/in/sourodeeproy/)


