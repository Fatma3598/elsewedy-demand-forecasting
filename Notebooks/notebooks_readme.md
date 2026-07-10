# 📓 Notebooks

This folder contains the 5 notebooks representing the complete pipeline, from synthetic data generation to production-ready inference with adaptive model selection.

Run them in order:

| # | Notebook | Purpose |
|---|----------|---------|
| 1 | `ElSewedy_Dataset.ipynb` | Generates the synthetic dataset (612 records baseline + 5-year expanded version) |
| 2 | `ElSewedy_Data_Visualization.ipynb` | Exploratory data analysis — demand trends, seasonality, supplier risk, inventory health |
| 3 | `ElSewedy_Forecasting.ipynb` | Trains and evaluates 6 candidate models (Random Forest, Gradient Boosting, XGBoost, LSTM, TCN, GRU) |
| 4 | `Dynamic_Model_Selection_Meta_Learning.ipynb` | Meta-learning layer that selects the best model per material (R² = 0.984) |
| 5 | `ElSewedy_Demand_Inference.ipynb` | Production inference pipeline — generates forecasts, safety stock, and risk alerts |

For detailed descriptions of each notebook's methodology and results, see the main [README](../README.md).

## Requirements

```bash
pip install pandas numpy scikit-learn xgboost tensorflow matplotlib seaborn
```

## Expected Inputs/Outputs

- Notebooks 1-2 read/write from `../data/`
- Notebook 3 outputs trained models and `elsewedy_demand_forecast.csv`
- Notebook 4 outputs the per-material model selector
- Notebook 5 consumes the selector to produce final business outputs