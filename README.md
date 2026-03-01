# NHANES Dietary Predictors of Glycaemic Status

**A portfolio project demonstrating regression analysis on real public health data**

---

## The Question

Do dietary patterns predict glycaemic outcomes (HbA1c, fasting glucose) in a US population sample, after adjusting for BMI, age and sex?

This question matters because popular discourse links sugar and carbohydrate intake directly to diabetes risk — but observational evidence is more ambiguous. This project uses NHANES data to test that relationship rigorously, with appropriate statistical adjustment.

---

## Why This Dataset

**NHANES (National Health and Nutrition Examination Survey)** is the gold-standard US population health survey, conducted by the CDC. It combines interview, physical examination and laboratory data on a nationally representative sample.

- **NHANES 2017–2018** cycle: ~9,000 participants nationally
- **Variables available:** HbA1c, fasting glucose, BMI, waist circumference, two-day dietary recall (macronutrients, calories), demographics
- **Final analytic sample:** ~2,800 adults after exclusions (pregnant women, missing HbA1c or dietary data, extreme dietary outliers)

This is a real, messy, publicly available dataset — not simulated. The data download, merging of six separate XPT files, and cleaning decisions are fully documented in Script 01.

---

## Results

### Linear Regression — Predictors of HbA1c

| Predictor | Coefficient | 95% CI | p-value |
|---|---|---|---|
| BMI | **+0.244** | [0.202, 0.287] | <0.001 |
| Age | +0.018 | [0.015, 0.020] | <0.001 |
| Sex (Female) | −0.127 | [−0.217, −0.038] | 0.005 |
| Fibre intake | +0.062 | [0.004, 0.120] | 0.035 |
| Carbohydrate intake | −0.113 | [−0.233, 0.008] | 0.067 |
| Sugar intake | +0.043 | [−0.049, 0.136] | 0.357 |
| Fat intake | −0.004 | [−0.063, 0.054] | 0.882 |

All dietary variables standardised (z-scores). **R² = 0.147**

**Finding:** BMI and age are the dominant predictors of HbA1c. Dietary variables — including sugar and carbohydrate intake — add minimal explanatory power after adjusting for body composition.

### Logistic Regression — Odds of Prediabetes/Diabetes

Binary outcome: HbA1c ≥ 5.7% (prediabetic or diabetic) vs normal.

| Predictor | Odds Ratio | 95% CI |
|---|---|---|
| BMI | **3.07** | [2.63, 3.58] |
| Age | 1.07 | [1.06, 1.08] |
| Fibre intake | 1.24 | [1.08, 1.42] |
| Fat intake | 1.09 | [0.95, 1.26] |
| Sugar intake | 1.09 | [0.97, 1.23] |
| Carbohydrate intake | 0.90 | [0.75, 1.08] |
| Sex (Female) | 0.63 | [0.51, 0.77] |

Each 1 SD increase in BMI is associated with a **3× higher odds** of prediabetes/diabetes. No dietary variable shows a significant independent association with glycaemic status after BMI adjustment.

### Stratified Sensitivity Analysis — ORs by BMI Group

The logistic model was re-run separately within normal-weight, overweight and obese subgroups to test whether dietary signals differ by adiposity. See `figures/03_stratified_forest_plot.png`.

**Finding:** Within each BMI stratum, dietary ORs remain close to 1.0 and confidence intervals are wide — consistent with no meaningful dietary signal independent of body composition.

---

## Biological Interpretation

The finding that BMI dominates over dietary predictors in a cross-sectional study is mechanistically coherent:

- **HbA1c reflects chronic glycaemic state** (average over ~3 months) — it integrates metabolic burden accumulated over years, not just current diet
- **BMI is a proxy for insulin resistance** — adipose tissue, particularly visceral fat, drives systemic insulin resistance via inflammatory cytokines (TNF-α, IL-6) and free fatty acid release
- **A single dietary recall is a noisy proxy** for habitual dietary patterns — day-to-day variation in intake is large, attenuating any true diet–HbA1c relationship toward zero

The fibre coefficient (+0.062, p=0.035) is counterintuitive but likely reflects **reverse causation**: participants with known elevated glucose may have been clinically advised to increase dietary fibre, inflating the observed positive association.

---

## Limitations

- **Cross-sectional design:** diet and HbA1c measured at the same time — causality cannot be established
- **24-hour dietary recall:** a single-day snapshot is a noisy estimate of habitual diet; day-to-day variability is large
- **Reverse causation:** participants already diagnosed with high glucose may have modified their diet (especially fibre), masking the true relationship
- **Missing confounders:** physical activity was not included in this model — activity independently predicts HbA1c and correlates with BMI
- **Residual confounding:** medication use (particularly metformin), gut microbiome composition and sleep quality are not captured in this analysis

These limitations reflect the exact gaps that longitudinal studies with continuous glucose monitoring (e.g., ZOE PREDICT) were designed to address.

---

## Project Structure

```
Portfolio 2/
├── notebooks/
│   ├── 01_data_download_and_merge.py   # Download XPT files, merge, initial clean
│   ├── 02_exploratory_analysis.py      # Distributions, heatmap, glycaemic classification
│   └── 03_regression_analysis.py       # Linear + logistic regression, stratified analysis
├── data/
│   └── nhanes_clean.csv                # Final analytic dataset (output of script 01)
├── figures/
│   ├── 02_histogram.png                # HbA1c and BMI distributions
│   ├── 02_heatmap.png                  # Correlation matrix
│   ├── 02_boxplot.png                  # HbA1c by glycaemic status
│   ├── 03_linear_regression.png        # Coefficient plot — linear regression
│   ├── 03_logistic_regression.png      # Forest plot — odds ratios
│   └── 03_stratified_forest_plot.png   # ORs by BMI group
├── RESULTS.md                          # Detailed results with interpretation
└── README.md
```

---

## Methods

| Step | Tool | Description |
|---|---|---|
| Data download | `nhanes` Python package | Six XPT files: demographics, HbA1c, glucose, BMI, dietary recall day 1 and 2 |
| Merging | `pandas.merge()` on SEQN | Inner join across all six files |
| Cleaning | exclusion criteria, outlier removal | Pregnant women, missing outcomes, extreme kcal values |
| Standardisation | `sklearn.StandardScaler` | Continuous predictors converted to z-scores for comparability |
| Linear regression | `statsmodels.formula.api.ols` | OLS with formula interface |
| Logistic regression | `statsmodels.formula.api.logit` | Binary outcome (HbA1c ≥ 5.7%) |
| Visualisation | `matplotlib`, `seaborn` | Coefficient plot, forest plots, correlation heatmap |

---

## How to Run

```bash
cd "Portfolio 2/notebooks"
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn nhanes
python 01_data_download_and_merge.py   # downloads data, outputs nhanes_clean.csv
python 02_exploratory_analysis.py      # generates figures/02_*
python 03_regression_analysis.py       # generates figures/03_* and data/03_model_summary.csv
```

---

**Portfolio project by Lara F. Scofano**
*Completed March 2026*
