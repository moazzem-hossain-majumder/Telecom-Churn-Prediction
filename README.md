# Telecom Churn Prediction (Cell2Cell)

**CSE437 Group 8 — BRAC University**
Md. Shihab Sharar · Moazzem Hossain Majumder · Fahim Montasir · Nafisa Islam · Sabbir Ahmed · Md. Golam Rabiul Alam

Predicting customer churn for a US telecom carrier using the classic
**Cell2Cell** benchmark dataset, with a strong focus on avoiding the
data-leakage and over-optimistic-metric problems that show up repeatedly
in prior published work on this dataset. Full write-up is in
[`report/Churn_Prediction_Report.pdf`](report/Churn_Prediction_Report.pdf).

## Problem & Motivation

Acquiring a new telecom customer costs 5–10x more than retaining an
existing one, so predicting who is about to churn is a high-value,
long-standing business problem. A literature review of 20 papers on
Cell2Cell and related telecom churn datasets found accuracies ranging
from the 50s to nearly 99%, but very few papers report calibration,
significance testing, or a leakage-free evaluation protocol. This
project builds a single, consistent, leakage-controlled pipeline and
compares seven baseline models under it — showing that once cross-
validation and a truly held-out test set are used properly, realistic
F1 scores on this dataset are far lower (~0.37–0.41) than the inflated
numbers (up to 88%) reported elsewhere.

## Dataset

- **Source:** Cell2Cell, Teradata Center for CRM at Duke University
  (via Kaggle), 51,047 customer records, 58 raw features.
- **Target:** `Churn` (Yes/No), ~29% positive class (moderately imbalanced).
- Features cover usage (minutes, calls), billing/revenue, equipment,
  demographics, service area, and customer-care interactions.

## Methodology

1. **Split once, early.** An 80/20 stratified train/test split is done
   *before* any cleaning; the 20% held-out test set is touched exactly
   once per model, at final evaluation — never during EDA, feature
   engineering, or cross-validation.
2. **Cleaning & imputation** (notebook 1): outlier capping at the 99th
   percentile (train-set thresholds only), KNN imputation for
   demographic numerics, median imputation for other numerics, mode
   imputation for categoricals, `ServiceArea` collapsed from 748 to 15
   categories, redundant/duplicate rows removed.
3. **Two feature-set versions** are produced: all 57 features, and a
   54-feature version with three highly-correlated columns
   (`ReceivedCalls`, `UniqueSubs`, `HandsetModels`) dropped.
4. **Ablation study** (notebook 2): every model is run once per
   logical feature group (All Features, Usage, Call Quality,
   Demographics, Equipment, Service, Behavioral, Selected) using
   stratified 5-fold CV, with SMOTE applied inside every fold. The
   feature group with the best mean validation F1 is then used for
   that model's single final evaluation on the held-out test set.
5. **Models compared:** Logistic Regression, Random Forest, AdaBoost,
   XGBoost, CatBoost, SVC (linear/poly/RBF), and an MLP neural network,
   all run through the same shared training/evaluation function so
   results are directly comparable. SHAP is used to explain the best
   CatBoost model.

## Results

Final held-out test-set performance (best feature group per model, by
validation F1):

| Model | Feature Group | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| Logistic Regression | All Features | 0.627 | 0.346 | 0.490 | **0.405** | 0.614 |
| AdaBoost | Selected Features | 0.450 | 0.283 | 0.729 | 0.408 | 0.572 |
| Random Forest | Service Features | 0.535 | 0.301 | 0.596 | 0.400 | 0.579 |
| CatBoost | Equipment Features | 0.557 | 0.301 | 0.534 | 0.385 | 0.570 |
| SVC (RBF) | Usage Features | 0.437 | 0.267 | 0.669 | 0.381 | 0.486 |
| XGBoost | Equipment Features | 0.577 | 0.296 | 0.457 | 0.359 | 0.560 |

*(MLP was also run through the ablation study — Equipment Features
scored best on validation F1 — see the notebook for full details.)*

**Takeaway:** under a rigorous, leakage-controlled evaluation, F1
scores across all seven models land around 0.35–0.41, well below the
80–99% figures often reported in prior Cell2Cell studies. Equipment
and usage-related features consistently matter most, while precision
is the shared weak point across every model — a large number of false
positives remains an open problem for future work (e.g. SMOTE
variants, cost-sensitive thresholds, temporal/RNN-based models).

## Repository Structure

```
Telecom-Churn-Prediction/
├── README.md                                  <- you are here
├── requirements.txt                            <- Python dependencies
├── report/
│   └── Churn_Prediction_Report.pdf             <- full written report (IEEE format)
├── notebooks/
│   ├── 01_data_cleaning_and_eda.ipynb          <- EDA, outlier capping, imputation, train/test split
│   └── 02_model_training_and_evaluation.ipynb  <- ablation study, 7 baseline models, SHAP, results table
└── data/
    ├── README.md                               <- description of every data file
    ├── cell2cell.csv                           <- raw source dataset
    └── ... cleaned / split / version1 & version2 CSVs
```

## Reproducing

The notebooks were originally run in Google Colab with data mounted
from Google Drive. To run locally instead:

1. `pip install -r requirements.txt`
2. Update the `dataset_path` variable at the top of each notebook to
   point at the local `data/` folder (and remove/skip the
   `google.colab.drive.mount(...)` cell).
3. Run `notebooks/01_data_cleaning_and_eda.ipynb` first (regenerates
   the cleaned/versioned CSVs in `data/`), then
   `notebooks/02_model_training_and_evaluation.ipynb`.

## Citation

If you build on this work, please cite the accompanying report:

> Sharar, M.S., Majumder, M.H., Montasir, F., Islam, N., Ahmed, S., Alam, M.G.R.
> *Improving Churn Prediction in the Telecommunication Industry Using Machine Learning.*
> CSE437, BRAC University.
