# 🌽 MaizeIQ — XGBoost Maize Yield Prediction System
### One Acre Fund Dataset | 2016–2022 | East & West Africa

---

## Project Overview

A complete, interpretable XGBoost maize yield prediction system with SHAP + LIME explanations, deployed as a farmer-facing Streamlit web app.

**Model performance (on 74,967 rows):**
- Test RMSE: **388 kg/ha**
- Test MAE:  **299 kg/ha**
- Test R²:   **0.874**

---

## Project Structure

```
.
├── phase1_eda.py           # EDA, yield distribution, correlations, MI scores
├── phase2_cleaning.py      # Data cleaning, imputation, outlier handling, renaming
├── phase3_features.py      # Feature engineering, PCA, feature ranking
├── phase4_5_train.py       # Stratified split + XGBoost training with Optuna
├── phase6_xai.py           # SHAP + LIME explanations, quality metrics
├── app.py                  # Streamlit multi-page web application
├── requirements.txt        # All Python dependencies
└── README.md               # This file
```

---

## Quick Start

### 1. Install dependencies
```bash
pip install -r requirements.txt
```

### 2. Place your dataset
Copy `One_Acre_Fund_MEL_maize_survey_data_2016-2022.csv` into the project folder.

### 3. Run the pipeline (in order)
```bash
python phase1_eda.py          # EDA plots → eda_*.png
python phase2_cleaning.py     # → cleaned_data.csv
python phase3_features.py     # → scaler.joblib, pca.joblib, feature_names.joblib
python phase4_5_train.py      # → model.joblib (RMSE ~388, R² ~0.87)
python phase6_xai.py          # → shap_*.png, lime_*.png, explainer joblibs
```

### 4. Launch the Streamlit app
```bash
streamlit run app.py
```
Open your browser to **http://localhost:8501**

---

## Top 5 Farmer-Controllable Features (by Mutual Information)
| Rank | Feature | MI Score | Pearson |r| |
|------|---------|---------|----------|
| 1 | Phosphorus Applied (kg/ha) | 0.292 | 0.598 |
| 2 | Management Score | 0.291 | 0.606 |
| 3 | Nitrogen Applied (kg/ha) | 0.251 | 0.535 |
| 4 | Planting Density (plants/m²) | 0.112 | 0.371 |
| 5 | Planting Day of Year | 0.074 | varies |

---

## App Pages
| Page | Description |
|------|-------------|
| 🌾 Predict Yield | Large yield metric card, confidence interval, SHAP driver chart, growth stage |
| 🔍 Explain Prediction | SHAP waterfall, LIME bar chart, plain-language summary, what-if insights |
| 📋 Past Forecasts | History table, yield trend chart, CSV download |
| 💡 Tips | Agronomic guidance for each controllable feature |

---

## Key Design Decisions

### Data Cleaning
- Dropped 5 columns with |Pearson r| < 0.05 vs yield: `year`, `lat`, `avg_season_tavg`, `weeding`, `elevation_m`
- Zero yield values imputed with column mean
- IQR capping for outliers in fertilizer and yield columns
- All columns renamed to human-readable labels (see `RENAME_MAP` in `phase2_cleaning.py`)

### Model
- XGBoost with `tree_method='hist'` for speed on 75k rows
- Optuna TPE sampler, 50 trials, optimising RMSE
- Stratified 80/20 split by yield quartile

### XAI
- SHAP TreeExplainer (exact for tree models, no approximation)
- LIME LimeTabularExplainer with 500-row background sample
- All plots use human-readable feature names throughout

---

## Deploying to Streamlit Cloud

1. Push this repository to GitHub (include `requirements.txt`)
2. Go to [share.streamlit.io](https://share.streamlit.io) → New app
3. Select your repo, set **Main file path** to `app.py`
4. Upload model artifacts (`model.joblib`, `scaler.joblib`, `feature_names.joblib`, `cleaned_data.csv`) via the Streamlit Cloud file uploader or store them in the repo

> **Note:** For production, store artifacts in cloud storage (S3, GCS) and load via `smart_open` or `fsspec`.

---

## Column Reference (Raw → Human-Readable)
| Raw Column | Human-Readable Label |
|-----------|---------------------|
| `yield_kg_ha` | Yield (kg/ha) |
| `N_kg_ha` | Nitrogen Applied (kg/ha) |
| `P_kg_ha` | Phosphorus Applied (kg/ha) |
| `K_kg_ha` | Potassium Applied (kg/ha) |
| `pl_m2` | Planting Density (plants/m²) |
| `plant_doy` | Planting Day of Year |
| `mgmt_score` | Management Score |
| `fert_adequacy` | Fertilizer Adequacy |
| `NPK_total` | Total NPK (kg/ha) |
| `season_prec` | Season Precipitation (mm) |
| `drought_stress` | Drought Stress Index |
| `soil_fertility_idx` | Soil Fertility Index |
| `soil_quality_idx` | Soil Quality Index |
| ... | (see RENAME_MAP in phase2_cleaning.py) |
