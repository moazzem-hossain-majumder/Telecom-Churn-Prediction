# Data

All files here are produced from the [Cell2Cell telecom churn dataset](https://www.kaggle.com/datasets/jpacse/datasets-for-churn-telecom)
(51,047 rows, 58 columns), originally from the Teradata Center for Customer
Relationship Management at Duke University.

| File | Description |
|---|---|
| `cell2cell.csv` | Raw, unmodified source dataset. |
| `X_cleaned.csv` / `y_cleaned.csv` | Full cleaned training/validation feature set and target, after outlier capping, imputation, and duplicate removal (notebook 1). |
| `X_version1_all_features.csv` / `y_version1_all_features.csv` | Cleaned train/val data, **all 57 features** kept. |
| `X_version2_no_corr.csv` / `y_version2_no_corr.csv` | Cleaned train/val data with the three highly-correlated features (`ReceivedCalls`, `UniqueSubs`, `HandsetModels`) removed (54 features). |
| `X_test_version1_all_features.csv` / `y_test_version1_all_features.csv` | Held-out test split (20%), all features, cleaned/imputed the same way as training data. |
| `X_test_version2_no_corr.csv` / `y_test_version2_no_corr.csv` | Held-out test split, correlated features removed. |
| `X_test_no_impute_version1_all_features.csv` / `y_test_no_impute_version1_all_features.csv` | Held-out test split, all features, **rows with missing values dropped instead of imputed**. |
| `X_test_no_impute_version2_no_corr.csv` / `y_test_no_impute_version2_no_corr.csv` | Held-out test split, correlated features removed, missing rows dropped instead of imputed. |

The 80/20 train/test split is stratified on `Churn` and performed once,
before any cleaning, so the held-out test set is never touched during
feature engineering, model selection, or cross-validation. See
`notebooks/01_data_cleaning_and_eda.ipynb` for exactly how each file above
was generated.
