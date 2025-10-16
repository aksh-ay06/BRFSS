# BRFSS 2023 — Risk Modeling & Insights ( work in progress )

End-to-end analysis of the CDC **Behavioral Risk Factor Surveillance System (BRFSS) 2023** microdata to build interpretable risk models and actionable insights.

## Overview
- Large-scale survey preprocessing (types, missing codes, codeframes).
- Class-imbalance handling (e.g., SMOTE/SMOTENC) and multicollinearity control.
- Model suite: Logistic Regression, Random Forest, XGBoost, CatBoost (+ Optuna tuning).
- Transparent evaluation (ROC-AUC, PR-AUC, F1/Recall @ tuned threshold, calibration).
- Explainability with SHAP for feature importance and per-prediction rationale.

## Repo Structure
```
BRFSS/
├─ BRFSS2023.ipynb   # EDA → preprocessing → modeling → interpretation
└─ README.md
```

## Environment
```bash
python -m venv .venv && source .venv/bin/activate
pip install -U pip wheel
pip install pandas numpy pyarrow scipy scikit-learn imbalanced-learn shap             matplotlib plotly xgboost catboost optuna jupyter
```

## Data
Download 2023 BRFSS microdata and documentation from CDC. Keep raw files outside the repo if possible and set a path in the notebook:
```python
DATA_DIR = "/path/to/brfss2023"
```

## How to Run
```bash
jupyter lab   # open BRFSS2023.ipynb and Run All
```

## Results
- **Target/outcome:** `CHCOCNC1` (self-reported cancer diagnosis; Yes=1, No=0)
- **Best model:** `Logistic Regression` (slightly higher AUPRC; equal ROC-AUC to XGB)
- **Test ROC-AUC / PR-AUC:** `0.799 / 0.217` (LogReg) — `0.799 / 0.215` (XGB)
- **Operating threshold & rationale:** Threshold chosen to achieve ~0.75 **recall** on validation.
  - LogReg `thr ≈ 0.0936` → Precision `0.184`, Recall `0.741`
  - XGB `thr ≈ 0.0844` → Precision `0.180`, Recall `0.773`
- **Base prevalence (weighted, test):** `~7.9%` → **AUPRC lift** ≈ `2.75×` (LogReg), `2.74×` (XGB)
- **Top drivers (model-based):** `_AGE_G` (older age bands), `_DRDXAR2` (arthritis diagnosis status), `_RACEPRV` (race category); also signal from `_SEX` and `_CHLDCNT`.


## Notes & Assumptions
- Survey **raking weights** retained for reporting; model training may be unweighted—report population metrics with weights where relevant.
- Self-reported measures and telephone survey design introduce known biases; use results as decision support, not causal claims.

## Extensions
- Weighted training and weighted metrics.
- Threshold tuning vs. cost matrix by stakeholder scenario.
- Package to FastAPI for batch/online scoring + SHAP dashboards.

## License
Analysis code for research/education. Verify CDC terms before redistributing data.
