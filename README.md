# Capstone-Projects
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
