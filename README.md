# Forecast Accuracy Is Not Enough — code and results

Code, configuration and results for the article:

> Mununga Kawika, D., & Ferchichi, O. (2026). *Forecast Accuracy Is Not Enough: Linking Statistical, Machine Learning and Deep Learning Forecasts to Safety-Stock Requirements in Multi-SKU Retail.* Manuscript under review.

The study compares ten forecasting methods (naive and exponential-smoothing benchmarks, ARIMA, SARIMA, Random Forest and a multi-seed LSTM) on the 20 top-selling FOODS items of Walmart store CA_1 in the M5 dataset. It traces their errors through to safety stock and to an out-of-sample inventory simulation.

## Repository layout

```
dataset/brutes/       M5 raw files (not included, see below)
dataset/propres/      train/test panels written by notebook 01 (not included, regenerated)
models/               fitted models written by notebook 01 (not included, regenerated)
notebook/
  01_main_pipeline.ipynb      SKU selection, features, ARIMA(1,1,1), Random Forest, LSTM (seed 42)
  02_sarima_lstm_seeds.ipynb  SARIMA per SKU, LSTM with 5 seeds, then runs corrections_v4.py
  corrections_v4.py           availability screening, benchmarks, profiles, tests, safety stock, simulation
  extras_v5.py                WRMSSE, Diebold–Mariano tests, MAPE analysis, prices, sensitivity table
  make_figures.py             Figures 1–6 of the article
results_article/      outputs of notebook 01
results_article_v4/   outputs of notebook 02 + corrections_v4.py (tables of the article)
figures/              article figures
```

## Data

The M5 data are not redistributed here. Download `calendar.csv`, `sales_train_validation.csv` and `sell_prices.csv` from the [M5 Forecasting – Accuracy competition on Kaggle](https://www.kaggle.com/competitions/m5-forecasting-accuracy/data) (accepting the competition rules is required) and place them in `dataset/brutes/`.

The result files in `results_article/` and `results_article_v4/` contain the observed daily sales of the 20 selected SKUs alongside the forecasts, so that the tables can be checked without rerunning the models.

## Reproducing the results

Tested with Python 3.12 on macOS (Intel x86_64, CPU only).

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cd notebook
jupyter nbconvert --to notebook --execute --inplace 01_main_pipeline.ipynb      # about 7 min
jupyter nbconvert --to notebook --execute --inplace 02_sarima_lstm_seeds.ipynb  # about 25 min
python extras_v5.py ../results_article_v4 ../dataset/propres
python make_figures.py
```

To check the article tables without TensorFlow or statsmodels, run only the evaluation step on the saved forecasts (numpy, pandas and scipy are enough). `dataset/propres/` must first be filled by notebook 01:

```bash
cd notebook
python corrections_v4.py ../results_article_v4/predictions_v4_panel.csv ../dataset/propres ../results_article_v4
python extras_v5.py ../results_article_v4 ../dataset/propres
python make_figures.py
```

## Where each result comes from

| Article | File in `results_article_v4/` |
| --- | --- |
| Section 3.3, availability screening | `availability_by_sku.csv`, `summary_v4.json` |
| Section 4.1, demand profiles | `demand_profiles_v4.csv` |
| Figure 2, example forecasts | `predictions_v4_panel.csv` |
| Section 4.1, prices | `price_summary.csv` |
| Table 4 and WRMSSE, accuracy | `metrics_global_v4.csv`, `metrics_by_sku_v4.csv`, `lstm_seeds_available_days.csv`, `wrmsse.csv`, `wrmsse_alt_weights2014.csv`, `sku_weights.csv` |
| Figure 3, error distribution; per-SKU MAE | `forecast_panel_all_methods.csv`, `mae_distribution.csv` |
| Table 5, pairwise comparisons | `statistical_tests_v4.csv`, `bootstrap_mae_v4.csv`, `extra_comparisons_sarima.csv`, `diebold_mariano.csv` |
| Table 6 and Figure 4, profiles | `metrics_by_profile_v4.csv`, `metrics_by_sku_v4.csv` |
| Table 7, sensitivity to service level and lead time | `ss_sensitivity_table.csv` |
| Table 8, Figures 5 and 6, safety stock and simulation | `safety_stock_v4.csv`, `inventory_simulation_summary.csv`, `inventory_simulation_by_sku.csv`, `stock_at_equal_service.csv` |
| Section 5.5, MAPE | `mape_analysis.csv` |
| Section 4.6, robustness | `availability_threshold_sensitivity.csv` |
| Benchmark parameters | `baseline_parameters.csv` |
| Random Forest feature importance | `../results_article/rf_feature_importance.csv` |

## Reproducibility notes

- Random seed 42 for the main runs; the LSTM is also trained with seeds 7, 123, 2024 and 31415.
- TensorFlow results can differ slightly across hardware and library versions, even with fixed seeds.
- All tuning, scaling and SKU selection use the training period (2011–2014) only.

## Citation

Please cite the article and this archive (see `CITATION.cff`, and the DOI badge once it is issued).

## License

Code: MIT (see `LICENSE`). The M5 data remain subject to the terms of the M5 competition.

## Changelog

- **1.1.0** — adds `extras_v5.py` (WRMSSE with M5 weights, per-SKU Diebold–Mariano tests, MAPE analysis, price summary, safety-stock sensitivity table), the full forecast panel `forecast_panel_all_methods.csv`, and Figures 3–5; figures renumbered in article order.
- **1.0.0** — first archived version.
