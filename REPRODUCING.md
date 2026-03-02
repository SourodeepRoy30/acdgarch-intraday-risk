# Reproducing the Analysis

## Prerequisites
- R 4.4.x
- Run `renv::restore()` in RStudio to install all package dependencies

## Data
The raw tick data (`data/raw/BPGBGBX_Ticks_2020.04.01_2020.06.30.csv`) is
tracked via Git LFS and will download automatically when you clone the repo.
A one-day sample (April 1, 2020) is also included for quick pipeline checks.
See `data/DATA_ACCESS.md` for full data description.

## Running the pipeline
Run notebooks in order from the project root:

| Notebook | Produces |
|---|---|
| `notebooks/01_setup.Rmd` | Environment verification |
| `notebooks/02_data_preparation.Rmd` | `data/processed/events_*.rds` |
| `notebooks/03_cleaning_seasonality.Rmd` | `data/processed/events_*_clean.rds` |
| `notebooks/04_acd_model.Rmd` | `data/processed/acd_model.rds` |
| `notebooks/05_intensity_adjusted_returns.Rmd` | `data/processed/returns_5min.rds` |
| `notebooks/06_garch_volatility_models.Rmd` | `data/processed/garch_forecasts.rds` |
| `notebooks/07_var_es_backtesting.Rmd` iles are gitignored — run the notebooks to generate them.
Raw data is tracked via Git LFS and downloads automatically on clone.
