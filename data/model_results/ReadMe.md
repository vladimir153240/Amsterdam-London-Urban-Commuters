# Model Results

## Overview

This folder contains **estimated MNL model coefficients** from Phase 4 of the analysis. These files store the regression outputs (beta coefficients, standard errors, t-statistics, p-values) for interpretation in Phase 5 (Hypothesis Testing) and Phase 6 (Scenario Testing).
The files can be accessed via: https://drive.google.com/drive/folders/1ezucTysqFbE8G4zvq4N5OdpL4HZbJFzE.
---

## Files in This Folder

### **Amsterdam Model: `amst_final_beta.csv`**

**Size**: ~10 KB | **Rows**: Variable | **Columns**: 8

**Content**: Multinomial Logit model estimation results for Amsterdam commuting mode choice

**Choice Set**: 3 alternatives
1. Car (driver)
2. Public Transport (bus/tram/metro)
3. Bicycle

**Baseline**: Car (driver) is the baseline category

#### Reading the Results

```python
import pandas as pd

results_amst = pd.read_csv('amst_final_beta.csv')

# Show all variables with p < 0.05
significant = results_amst[results_amst['p_value'] < 0.05]
print(significant[['variable', 'coef', 'p_value']])

# Odds ratio (relative to car)
results_amst['odds_ratio'] = np.exp(results_amst['coef'])
```

#### Interpretation Guide

**Coefficient Sign**:
- **Positive (+)**: Increases probability of choosing PT or Bicycle (vs. Car)
- **Negative (-)**: Decreases probability of choosing PT or Bicycle (increases Car choice)

---


## Notes

1. Coefficients are log-odds
2. Baseline is car driver — Positive coef = higher PT/Bike probability
3. Weights applied — Results are population-representative
4. Significance verified — All reported effects meet α = 0.05    

