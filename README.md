Run down - Bulleted version for the GitHub project card, plus a short summary:

---

**Summary (2–3 sentences)**

Predicts county-level acute-care hospital bed capacity across New Mexico's 33 counties using HRSA Area Health Resources Files data, then uses the gap between predicted and actual capacity — the residual — as a provision-vs-need signal for where state health investment should be directed. Framed through System Evaluation Theory (Renger, 2022), treating the mismatch as a measurement within a reinforcing loop of funding, recruitment, and access rather than a root cause. Co-authored with Joseph Ikogho.

---

**Bullets**

**Problem**
- New Mexico ranks 5th in land area; rural residents travel hours to reach acute care
- 8 of 33 NM counties have zero short-term general hospital beds
- Goal: identify where state investment closes the largest provision-vs-need gaps

**Data**
- HRSA/NCHWA Area Health Resources Files (AHRF) 2024–2025, county-level CSV
- National: 3,235 rows × 4,352 columns; NM subset: 33 counties
- Cleaning by missingness *mechanism*: structural (contiguous-county slots) dropped; MNAR suppression (infant mortality, small-count privacy) excluded rather than imputed
- Documented anomaly: Sandoval County reports zero acute beds despite Rio Rancho (pop. ~156k) — consistent with satellite-campus reporting under parent-system certification in Bernalillo County

**Target (y)**
- `stgh_beds_per_1k` — short-term general (acute care) beds per 1,000 residents, 2023
- Chosen over all-bed measures so psychiatric/VA/long-term beds don't mask absent emergency care
- `allhosp_beds_per_1k` retained for sensitivity analysis (definition-change robustness)
- Distribution is zero-inflated: 8 true zeros, continuous to 6.31

**Features**
- Final X: pct_65plus, median family income, poverty rate, log1p(density), CBSA rurality (one-hot)
- Dropped for collinearity: median_age (r = 0.98 with pct_65plus), population (r = 0.91 with density)
- Frame: 33 rows × 10 columns, zero missing values

**EDA findings**
- Bed-access medians nearly identical across rural/micropolitan/metropolitan — rurality alone doesn't predict access
- Rural counties bimodal: cluster of zeros plus small-denominator inflation (Union, 6.31)
- Zero-bed counties span the full density range — no single variable separates served from unserved
- No feature exceeds |0.3| correlation with target; weak linear signal expected at n=33

**Methods**
- Linear Regression and Random Forest on continuous target; GaussianNB on median-split binary
- All models scored against Dummy baselines — a null result is explicitly reportable

**Stack**
- Python, pandas, scikit-learn, matplotlib, plotly, geopandas
- Google Colab; notebook certified via Restart & Run All

**Limitations**
- Facility reporting artifacts (Sandoval/Presbyterian Rust)
- Mixed source vintages (median age 2020; population and income 2023)
- n = 33 limits statistical power
- Tribal/IHS facilities incompletely captured in AHRF# Capstone-Projects
Capstone projects for Data-Science-21 Cohort
The predictors or drivers that one would expect for medical access in New Mexico are not present in the data.  This finding is relevant for policy driven solutions.  Please review the following notations.

**Null result.** No signal: county-level need drivers (population density, 
poverty rate, elderly share) do not predict short-term general hospital bed 
provision in New Mexico. Every model tested — linear regression (6 and 3 
feature), Ridge, and Random Forest — performed worse than a Dummy mean 
predictor (R² -0.338). Because the null holds across linear, regularized, and 
non-linear model families, this reflects the data rather than the choice of 
algorithm.

Interpretation: if capacity were allocated according to measurable population 
need, these variables would carry signal. They do not. Provision appears driven 
by other factors — facility ownership and reporting structures, historical 
siting, institutional inertia — rather than by county need. This is consistent 
with the provision-vs-need mismatch the project set out to examine.

Limitations: n = 33; target is zero-inflated (8 counties with zero acute beds); 
small-denominator inflation (Union) and reporting artifacts (Sandoval, 
Valencia) add noise to the target.

Next: binary classification (GaussianNB) on a median split of the target — 
adequate vs. under-provisioned — to test whether a coarser target retains 
signal the continuous one loses.
