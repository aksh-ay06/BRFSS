# Can State-Level Industrial Pollution Improve Prediction of Self-Reported Cancer Prevalence?

An analysis combining **BRFSS 2023** survey data (433,323 adults) with **EPA P2 Mapping** facility-level pollution records to investigate whether state-aggregated pollution metrics are associated with — and predictive of — self-reported cancer.

## Data Sources

| Dataset | Records | Description |
|---------|---------|-------------|
| **BRFSS 2023** | 433,323 respondents | CDC Behavioral Risk Factor Surveillance System — complex survey design with stratification (`_STSTR`), clustering (`_PSU`), and survey weights (`_LLCPWT`) |
| **EPA P2 Mapping Data** | 69,416 facilities | Facility-level pollution aggregated to state level: TRI releases, GHG emissions, NEI criteria air pollutants, RCRA hazardous waste, DMR water pollution |

99.2% of BRFSS respondents (429,700) were successfully matched to state-level EPA pollution data.

## Approach

1. **Design-aware weighted EDA** using `samplics` to produce correct population estimates from the complex survey design
2. **State-level EPA aggregation** — facility pollution summed per state, log-transformed, and merged into respondent records
3. **Survey-weighted logistic regression** (`samplics.SurveyGLM`) for statistical inference with design-correct standard errors
4. **XGBoost classification** comparing baseline (demographics only) vs. pollution-augmented models using PSU-grouped cross-validation
5. **SHAP-based interpretation** of feature importance

## Key Findings

### Weighted Cancer Prevalence

The design-weighted cancer prevalence across the BRFSS 2023 sample is **7.92%** (95% CI: 7.76%–8.08%).

Prevalence increases sharply with age — the dominant risk factor — and varies by state, race, sex, and comorbidities.

### Survey-Weighted Logistic Regression

The pollution-augmented model (fit on 392,747 complete-case respondents) reveals statistically significant associations for all three pollution variables after adjusting for age, sex, race, smoking, arthritis, heart disease, education, and urbanicity:

| Pollution Variable | Odds Ratio | 95% CI | p-value |
|--------------------|-----------|--------|---------|
| NEI CAP Emissions (log) | **1.22** | 1.18–1.26 | < 0.001 |
| TRI Releases (log) | **0.79** | 0.77–0.82 | < 0.001 |
| GHG Emissions (log) | **0.94** | 0.91–0.96 | < 0.001 |

- **NEI criteria air pollutant emissions** show a positive association: each unit increase in log-NEI is associated with 22% higher odds of self-reported cancer.
- **TRI releases** and **GHG emissions** show inverse associations, likely reflecting confounding by state-level characteristics (e.g., industrialized states may have younger populations, better screening access, or other demographic profiles).
- Adding pollution variables causes the urban/rural odds ratio to attenuate from 0.63 to 0.91, suggesting pollution partially mediates the urban-rural cancer prevalence gap.
- Smoking (`_RFSMOK3`) loses significance in the pollution-augmented model (p = 0.86), suggesting collinearity with state-level pollution or demographic adjustments.

### XGBoost Predictive Performance

Models were evaluated using 5-fold PSU-grouped stratified cross-validation and a held-out test set (86,198 respondents):

| Model | CV AUPRC | CV ROC-AUC | Test AUPRC | Test ROC-AUC | Lift over baseline |
|-------|----------|------------|------------|--------------|-------------------|
| XGB baseline (demographics) | 0.2193 | 0.7946 | 0.2176 | 0.7992 | 2.77x |
| XGB + pollution | 0.2208 | 0.7944 | 0.2176 | 0.7989 | 2.77x |

State-level pollution features provide **negligible predictive lift** over individual-level demographics. Both models achieve nearly identical test performance, with a 2.77x lift in AUPRC over the baseline prevalence rate.

### SHAP Feature Importance

The top predictors of self-reported cancer (by mean |SHAP value|) are dominated by individual-level features:

1. **Age group** — by far the strongest predictor
2. **Arthritis status** — strong comorbidity signal
3. **Heart disease (MI/CHD)**
4. **Physical health days**
5. **Education level**

Pollution variables rank low in SHAP importance, confirming that state-level aggregation is too coarse to meaningfully improve individual-level prediction.

## Limitations

- **Self-reported outcome**: Cancer status may be subject to recall bias
- **State-level pollution only**: The BRFSS public-use file identifies respondents only at the state level. County- or ZIP-level linkage (available in restricted BRFSS) would provide sharper exposure estimates
- **Cross-sectional design**: Cannot infer causation — confounding by state-level healthcare access, demographics, and screening rates is likely
- **Population coverage**: BRFSS excludes institutionalized populations and those without telephones

## Conclusion

While state-level pollution metrics show statistically significant associations with self-reported cancer in the survey-weighted logistic model, they provide **no meaningful predictive improvement** over individual-level demographics in the XGBoost framework. Age remains the overwhelmingly dominant predictor. Finer-grained geographic pollution linkage (county FIPS from restricted BRFSS) would be a natural next step to better isolate environmental exposure effects.

## Technical Stack

- **Python 3.10** with `samplics` (survey statistics), `xgboost`, `shap`, `scikit-learn`
- `StratifiedGroupKFold` with PSU-based groups to prevent data leakage across survey clusters
- Survey weights normalized and applied throughout EDA, regression, and ML evaluation

## Repository Structure

```
BRFSS2023.ipynb     # Main analysis notebook
data/               # BRFSS 2023 CSV/Parquet + EPA P2 Mapping Excel (not tracked)
pyproject.toml      # Project dependencies (managed with uv)
```
