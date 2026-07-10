# 🔌 AI-Based Demand Forecasting for Cable Raw Materials

**Elsewedy Electric AI Innovation Competition | Top 5 Finalist 🏆**

An end-to-end AI system that forecasts demand for critical cable manufacturing raw materials — polymer layers (XLPE, PE, PVC, LSF), shielding materials (GSW, GST, copper/aluminum screens), and blocking materials (mica tape, water-blocking tape) — to help optimize inventory, reduce costs, and prevent supply chain disruptions.

## 🎯 Key Results

| Metric | Value |
|--------|-------|
| **Dynamic Selection Accuracy** | R² = 0.984 |
| **Dynamic Selection MAE** | ~1.28 tons |
| **Best Single Model (Gradient Boosting)** | R² ≈ 0.985 |
| **Dataset Size** | 612 records (17 materials × 36 months) — also validated on a 5-year expanded dataset |
| **Estimated Annual Savings** | $1.4M - $2.7M |

---

## 📓 Notebooks Overview

This repository contains 5 notebooks representing the full pipeline — from synthetic data generation to production-ready inference with adaptive model selection.

### 1.`ElSewedy_Dataset.ipynb`
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1mYLbWn1NT5v7cYm2p5qLGzGLALyPomtS?usp=sharing)

**Synthetic Data Generation**

Generates a realistic dataset simulating procurement history across 17 cable raw materials sourced from 8 global suppliers.

- Models material-specific demand patterns (growth rates, seasonality)
- Simulates supplier behavior (lead times, performance scores, pricing)
- Incorporates real-world business rules: quarterly ordering for non-metals, annual contracts for metals
- Two versions generated: **612-record baseline** (36 months) and an **expanded 5-year dataset** for robustness testing
- Outputs: `elsewedy_materials_dataset.csv` (20 columns)

---

### 2. `ElSewedy_Data_Visualization.ipynb`
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1vFegSGpiFLUAtH2a64nAgS24LRp6FyTP?usp=sharing)

**Exploratory Data Analysis**

Deep-dive analysis into demand patterns, supplier performance, and inventory health before modeling.

- Demand trends by material type and voltage level
- Seasonality detection (Q2/Q4 spikes)
- Supplier performance vs. risk analysis
- Stock coverage and safety stock health checks
- Feature correlation heatmaps
- Outputs: 13+ visualizations (`elsewedy_data_analysis.png`, `elsewedy_deep_analysis.png`)

---

### 3. `ElSewedy_Forecasting.ipynb`
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/19Yv_0uJ10MQ1mT5KMlw0Zn9rZ4qnXpPs?usp=sharing)

**Core Forecasting Models (Traditional ML + Deep Learning)**

Trains and evaluates 6 forecasting models on 31 engineered features (temporal lags, rolling statistics, risk scores, inventory ratios).

**Traditional ML (Strong Performers):**
| Model | R² Score | MAE (TON) |
|-------|----------|-----------|
| **Gradient Boosting** ⭐ | **~0.985** | **~1.28** |
| XGBoost | ~0.98 | ~1.5 |
| Random Forest | ~0.97 | ~1.6 |

**Deep Learning (Underperformed on this dataset size):**
| Model | R² Score | MAE (TON) |
|-------|----------|-----------|
| GRU | ~0.64 | ~7.1 |
| TCN | ~0.54 | ~8.0 |
| LSTM | ~0.47 | ~8.4 |

> 💡 **Key Finding:** On a 612-record tabular dataset, tree-based ensemble methods (Gradient Boosting, XGBoost) significantly outperformed deep learning architectures. This directly motivated the meta-learning approach in Notebook 4.

- Feature engineering pipeline (lag features, moving averages, cyclical encoding)
- Time-based train/test split to prevent data leakage
- 6-month ahead demand forecasting with safety stock recommendations
- Outputs: `elsewedy_demand_forecast.csv`, trained model files

---

### 4. `Dynamic_Model_Selection_Meta_Learning.ipynb`
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/Notebooks/Dynamic_Model_Selection_Meta_Learning.ipynb)

**Meta-Learning for Adaptive Model Selection**

Since traditional ML models clearly outperformed deep learning on this dataset, this notebook implements a **meta-learning layer** that selects the best-performing model *per material* rather than forcing one global model on all 17 materials.

**How it works:**
- Trains a per-material champion model selector (evaluated on R² per material)
- For each material, compares candidate models and picks the best performer
- Result: **Gradient Boosting selected for ~16 of 17 materials**, **XGBoost selected for Aluminum Screen** (its demand volatility pattern favored XGBoost)
- Deep learning models (LSTM/TCN/GRU) were **not selected for any material** in this run — confirming tree-based models are better suited to this tabular, moderate-size dataset

**Performance of Dynamic Selection vs. Fixed Models:**
| Approach | R² Score | MAE (TON) |
|----------|----------|-----------|
| **Dynamic Selection** ⭐ | **0.984** | **~1.28** |
| Best Fixed ML Model | ~0.985 | ~1.28 |
| Best Fixed DL Model | ~0.64 | ~7.1 |
| Avg. ML Models | ~0.97 | — |
| Avg. DL Models | ~0.55 | — |

**Improvement of Dynamic Selection over average DL models: ~34% R² gain**

- Per-material selection frequency analysis
- Prediction error distribution (tight clustering around zero)
- Cumulative absolute error tracking across test samples
- Risk Score vs. Demand Volatility mapping per selected model

---

### 5. `ElSewedy_Demand_Inference.ipynb`
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lqZmz0DevW27Q9-zjAmAixOiMWVj2QC9?usp=sharing)

**Production Inference Pipeline**

The deployment-ready notebook that loads the meta-learned model selector and generates live forecasts, safety stock levels, and risk alerts for new data.

- Loads serialized models (Gradient Boosting primary, XGBoost for select materials)
- Applies the dynamic model-selection logic from Notebook 4 automatically per material
- Generates rolling 6-month forecasts per material
- Calculates dynamic safety stock: `Safety Stock = Lead_Time_Demand × (1 + Risk_Score/10 × 0.5)`
- Flags supply chain risk levels (Low/Medium/High)
- Produces business-ready output tables for procurement teams

---

## 🗂️ Repository Structure

```
elsewedy-demand-forecasting/
│
├── notebooks/
│   ├── README.md
│   ├── ElSewedy_Dataset.ipynb
│   ├── ElSewedy_Data_Visualization.ipynb
│   ├── ElSewedy_Forecasting.ipynb
│   ├── Dynamic_Model_Selection_Meta_Learning.ipynb
│   └── ElSewedy_Demand_Inference.ipynb
│
├── data/
│   ├── README.md
│   ├── elsewedy_materials_dataset.csv          # 612-record baseline
│   └── elsewedy_materials_dataset_expanded.csv # 5-year validation set
│
├── LICENSE
└── README.md
```

---

## 🛠️ Tech Stack

- **Data Processing:** Pandas, NumPy
- **Traditional ML:** Scikit-learn, XGBoost
- **Deep Learning:** TensorFlow / Keras
- **Visualization:** Matplotlib, Seaborn
- **Environment:** Google Colab

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/Fatma3598/elsewedy-demand-forecasting.git
cd elsewedy-demand-forecasting

# Install dependencies
pip install pandas numpy scikit-learn xgboost tensorflow matplotlib seaborn

# Run notebooks in order:
# 1. ElSewedy_Dataset.ipynb                          → generates raw data
# 2. ElSewedy_Data_Visualization.ipynb                → explore patterns
# 3. ElSewedy_Forecasting.ipynb                       → train 6 candidate models
# 4. Dynamic_Model_Selection_Meta_Learning.ipynb      → per-material model selection
# 5. ElSewedy_Demand_Inference.ipynb                  → generate production forecasts
```

**Or run directly in Google Colab:**

| Notebook | Open in Colab |
|----------|----------------|
| 1. Dataset Generation | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/notebooks/ElSewedy_Dataset.ipynb) |
| 2. Data Visualization | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/notebooks/ElSewedy_Data_Visualization.ipynb) |
| 3. Forecasting Models | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/notebooks/ElSewedy_Forecasting.ipynb) |
| 4. Meta-Learning Selection | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/notebooks/Dynamic_Model_Selection_Meta_Learning.ipynb) |
| 5. Demand Inference | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Fatma3598/elsewedy-demand-forecasting/blob/main/notebooks/ElSewedy_Demand_Inference.ipynb) |

---

## 📊 Methodology Summary

```
Raw Data (20 cols, 612 records)
        ↓
Feature Engineering (31 features)
        ↓
Multi-Model Training (6 candidate models: 3 ML + 3 DL)
        ↓
Meta-Learning Selection (per-material best model: Gradient Boosting / XGBoost)
        ↓
Production Inference (forecasts + safety stock + risk alerts)
```

**Why Meta-Learning?** Rather than assuming one model fits all 17 materials, the system evaluates each material's demand pattern and volatility, then assigns the model that performs best for that specific material — avoiding the weaker deep learning results while still allowing flexibility.

---

## 👥 Team

**FCIS Cab Smart AI**  
Faculty of Computers and Information Sciences, Ain Shams University

🏆 **Top 5 Finalist** — Elsewedy Electric AI Innovation Competition

**Team Members:**
- Fatma Elwasify
- Ahmed Darweesh
- Radwa El-Hosseiny
- Mona Abdelazim
- Myvel Nabil Ramsis
- Omar Salama
- Ali Ahmed Shawky

**Notebooks & Modeling by:**
- **Fatma Elwasify** — fatma.gamal@cis.asu.edu.eg
- **Ahmed Darweesh** — ahmed.darweesh@cis.asu.edu.eg

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
