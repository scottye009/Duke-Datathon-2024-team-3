# Duke Datathon 2024 - Team 3

Predicting ICU length of stay and assessing floor readiness for sepsis patients who underwent surgical interventions during their ICU admission.

## Poster

[View Poster (PDF)](Research%20symposium-%20Predictive%20Model%20of%20ICU%20Length-of-Stay%20Following%20Surgery%20for%20Soft%20Tissue%20Infection%20in%20Sepsis.pdf)

## Overview

This project uses the MIMIC-IV clinical database to study sepsis patients in the ICU who required surgical procedures (amputation, debridement, or incision & drainage). We build predictive models for length of stay (LOS) and define a composite "floor readiness" metric to identify patients ready for ICU discharge.

## Dataset

- **Source:** MIMIC-IV via Google BigQuery (`physionet-data.mimiciv_*`)
- **Initial cohort:** 625 patients with sepsis and limb-related surgical procedures
- **Final cohort:** 432 ICU stays (after filtering patients who died during their stay with a single ICU admission)
- **Features include:**
  - Demographics (age, gender)
  - ICU admission severity markers (vasopressor use within 24hr, mechanical ventilation within 6hr, FiO2 > 60%)
  - Initial lab values (sodium, WBC, creatinine, pH, lactate, base excess, glucose, CRP)
  - Last lab values prior to discharge
  - Procedure timing and frequency

## Floor Readiness Criteria

A patient is classified as "floor ready" when all of the following are met:
- pH >= 7.3
- Lactate <= 3
- |Base excess| <= 3
- Creatinine <= 1.6 or < 1.5x initial value
- Off mechanical ventilation
- FiO2 <= 60%
- Off vasopressors
- Sodium 135-145
- Hemoglobin >= 7.0
- WBC 4.5-12
- CRP <= 20
- Glucose 60-180

## Analysis Pipeline

### 1. Data Construction (`05_Data_Construction.ipynb`)
- Extracts cohort from MIMIC-IV via BigQuery
- Joins ICU stays, sepsis diagnoses, and procedure codes
- Engineers temporal features (days between admission and procedures)
- Merges vasopressor, ventilation, and lab data
- Computes floor readiness labels

### 2. Linear Regression (`10_Linear_Regression.ipynb`)
- VIF-based feature selection to remove multicollinearity
- MinMax-scaled OLS regression for inference
- Significant predictors of LOS (p < 0.05): sepsis status, procedure frequency, days to first surgery, initial vasopressor need, initial sodium, initial creatinine, last base excess, last CRP
- Test R-squared: 0.40

### 3. Super Learner Ensemble (`12_Linear_Regression_Super_Learner.R`)
- Ensemble of Random Forest, Elastic Net (glmnet), Bayesian GLM, and SVM
- Includes diagnostic plots (residuals, Q-Q, leverage)
- Evaluated with R-squared, MSE, RMSE, and MAE on held-out test set

### 4. Clustering Analysis (`15_Clustering.ipynb`)
- K-Means and K-Medoids clustering on clinical features
- Dimensionality reduction with PCA and t-SNE (2D and 3D)
- TableOne summaries stratified by cluster
- Random Forest regression for LOS prediction (MSE: 80.7)
- Feature importance analysis

## Key Findings

- Days between ICU admission and first surgery is the strongest predictor of LOS
- Number of procedures, initial creatinine, and last CRP are significant drivers
- Only ~22% of patients met all floor readiness criteria at discharge
- Cluster analysis reveals distinct patient subgroups with different clinical trajectories

## Requirements

**Python:**
- pandas, numpy, matplotlib, seaborn
- scikit-learn, statsmodels, scipy
- tableone, pyclustering
- plotly
- google-cloud-bigquery (for data extraction)

**R:**
- tidyverse, ggplot2
- SuperLearner, caTools
- glmnet, randomForest, arm

## Usage

1. Data extraction requires access to MIMIC-IV on Google BigQuery (PhysioNet credentialed access)
2. Run `05_Data_Construction.ipynb` to build the cohort and export CSVs
3. Run `10_Linear_Regression.ipynb` or `12_Linear_Regression_Super_Learner.R` for predictive modeling
4. Run `15_Clustering.ipynb` for unsupervised analysis

## License

MIT
