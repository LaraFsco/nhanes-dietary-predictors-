# Portfolio 2 — Results
## NHANES Dietary Predictors of Glycaemic Status
**Author:** Lara Francesca Scofano
**Date:** February 2026
**Status:** In progress — updated as scripts are completed

---

## Dataset

- **Source:** NHANES 2017–2018 (National Health and Nutrition Examination Survey, US CDC)
- **Final sample:** ~2,800 participants after exclusions
- **Exclusions applied:** pregnant women, participants missing HbA1c or fasting glucose, extreme dietary outliers
- **Key variables:** HbA1c (%), fasting glucose (mg/dL), BMI, waist circumference, dietary intake (carbohydrates, sugar, fibre, fat, total kcal), age, sex

---

## Script 02 — Exploratory Analysis

### Glycaemic Status Classification

Participants classified by HbA1c using standard clinical thresholds:

| Status | HbA1c threshold | N | % |
|---|---|---|---|
| Normal | < 5.7% | — | — |
| Prediabetic | 5.7–6.4% | — | — |
| Diabetic | ≥ 6.5% | — | — |

*(Fill in counts from script 02 console output)*

### Key Observations from EDA

- HbA1c and fasting glucose distributions are right-skewed, with a long tail toward higher values — consistent with a population where a minority have overt diabetes
- BMI distribution peaks around 27–30 kg/m², reflecting the overweight range as modal in this US sample
- Carbohydrate intake is highly variable across participants (wide distribution), suggesting heterogeneous dietary patterns

### Correlation Heatmap — Key Findings

| Variable pair | Direction | Strength | Interpretation |
|---|---|---|---|
| HbA1c ↔ fasting glucose | Positive | Strong | Both measure glycaemic control — expected |
| BMI ↔ waist circumference | Positive | Strong | Both measure adiposity — tautological |
| BMI ↔ HbA1c | Positive | Moderate | Adiposity associated with worse glycaemic control |
| Dietary carbs ↔ HbA1c | Positive | Weak | Dietary carbohydrate intake has limited direct association with HbA1c |
| Fibre ↔ HbA1c | Positive | Weak | Counterintuitive — likely reverse causation (see Limitations) |
| kcal ↔ carb/fat/sugar | Positive | Strong | Calories are composed of macronutrients — expected |

**Interpretation:** Dietary variables show only weak correlations with glycaemic outcomes in this cross-sectional dataset. Body composition (BMI, waist circumference) shows stronger associations. This pattern is explored further in the regression analysis.

---

## Script 03 — Regression Analysis

### Linear Regression — Predictors of HbA1c (%)

**Model:** `HbA1c ~ carb_avg_z + sugar_avg_z + fiber_avg_z + fat_avg_z + BMI_z + age + C(sex)`
All dietary variables standardised (z-scores) for comparability of coefficients.

| Predictor | Coefficient | 95% CI | p-value | Interpretation |
|---|---|---|---|---|
| Intercept | 5.016 | [4.881, 5.151] | <0.001 | Baseline HbA1c (male, average predictors) |
| Sex (Female) | -0.127 | [-0.217, -0.038] | 0.005 | Females have 0.13% lower HbA1c than males ✅ |
| Carbohydrate intake | -0.113 | [-0.233, 0.008] | 0.067 | Trend toward lower HbA1c — not significant ❌ |
| Sugar intake | +0.043 | [-0.049, 0.136] | 0.357 | No association ❌ |
| Fibre intake | +0.062 | [0.004, 0.120] | 0.035 | Significant but counterintuitive — likely reverse causation ⚠️ |
| Fat intake | -0.004 | [-0.063, 0.054] | 0.882 | No association ❌ |
| BMI | +0.244 | [0.202, 0.287] | <0.001 | **Strongest predictor** — 1 SD higher BMI = +0.24% HbA1c ✅ |
| Age | +0.018 | [0.015, 0.020] | <0.001 | Each additional year = +0.018% HbA1c ✅ |

**R² = 0.147** — the model explains 14.7% of the variance in HbA1c. BMI and age alone account for the majority of this; dietary variables add minimal explanatory power beyond body composition.

#### Interpretation

The dominant predictors of HbA1c in this dataset are **BMI and age**, not dietary intake. Across the five dietary variables tested, none showed a statistically significant positive association with HbA1c after adjustment. This is a substantive finding, not a null result:

- It suggests that in a cross-sectional population sample, a single snapshot of dietary intake is a poor predictor of chronic glycaemic state
- Body composition (BMI) likely captures longer-term metabolic dysregulation that dietary recall cannot
- The fibre coefficient (+0.062, p=0.035) is likely explained by **reverse causation**: participants with higher HbA1c may have been advised by clinicians to increase fibre intake, inflating the observed association

---

## Limitations

- **Cross-sectional design:** diet and HbA1c measured at the same time — causality cannot be established
- **24-hour dietary recall:** a single-day snapshot is a noisy estimate of habitual diet; two recall days were available but averaging introduces its own assumptions
- **Reverse causation:** participants with known high glucose may have already modified their diet (particularly increasing fibre), masking the true diet–outcome relationship
- **Missing confounders:** physical activity was not included in this model (PAQ file downloaded but not merged at modelling stage) — this is a meaningful omission since activity independently predicts HbA1c and correlates with BMI. Future iteration should include it.
- **Residual confounding:** other unmeasured variables not captured in NHANES — gut microbiome composition, sleep quality, stress, medication use — may drive the observed associations
- **No postprandial data:** fasting glucose and HbA1c reflect chronic glycaemic state; they do not capture individual postprandial glucose variability, which is ZOE's primary outcome of interest

*"These limitations are the exact gaps that ZOE's PREDICT study design was built to address: continuous glucose monitoring, longitudinal dietary tracking, and microbiome profiling."*

---

## Next Steps

- [ ] Script 03: logistic regression (odds of prediabetes/diabetes) — Odds Ratios + forest plot
- [ ] Script 03: BMI-stratified sensitivity analysis
- [ ] Script 04 (if time): interaction term — carbohydrate × BMI group

