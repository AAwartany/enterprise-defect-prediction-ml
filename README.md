# AI-driven defect prediction in enterprise IT: a machine learning framework using historical JIRA data

Companion code repository for the manuscript **"AI-driven defect prediction in enterprise IT: a machine learning framework using historical JIRA data"** (PeerJ Computer Science submission).

## Description

This repository contains the machine learning pipeline used to predict high-risk software
defects from JIRA metadata in an enterprise IT environment. The pipeline trains and evaluates
four supervised classifiers (Random Forest, Gradient Boosting, Logistic Regression, and Support
Vector Machine), applies SMOTE to handle class imbalance, and produces SHAP-based feature
importance analysis on the best-performing model (Random Forest). All results reported in the
manuscript — the Table 3 performance metrics, the confusion matrix, and the SHAP beeswarm plot —
can be regenerated from the notebook applied to a JIRA dataset matching the documented schema.

## Dataset Information

- **Nature and source.** The dataset comprises proprietary defect records extracted from the
  JIRA instance of a single, multi-product enterprise IT organization. It **cannot be
  redistributed** due to organizational confidentiality constraints, so the raw data is not
  included in this repository. Researchers can apply the pipeline to their own JIRA export by
  matching the schema below.
- **Schema.** The full feature schema (field names, types, and descriptions, with no actual
  data values) is documented in `data_dictionary.md`.
- **Size.** 4,821 defect records after filtering (removing rows with null Severity or null
  Created date).
- **Time span.** Multiple years of release cycles; exact dates are withheld for confidentiality.
- **Features.** 35 features after engineering and one-hot encoding; see `data_dictionary.md` for
  the raw columns and the derived features.
- **Target variable.** `high_risk`, a binary label derived from the Severity field: 1 if
  Severity is in {Critical, High, Severe}, otherwise 0.
- **Class balance.** 26.49% high-risk vs. 73.51% low-risk. The imbalance is addressed with
  SMOTE on the training split only (see Methodology).

> The Priority field is deliberately **excluded** from the feature matrix to prevent label
> leakage, since Priority is structurally correlated with the Severity-derived target. It is
> retained in the input schema for inspection only. Reporter/component information is used only
> in aggregate form; individual identifiers such as Assignee and Developer are excluded to avoid
> individual performance attribution.

## Code Information

| File | Description |
|------|-------------|
| `MDPI_Paper_Real_Analysis_v3_NoPriority.ipynb` | Full notebook implementing the analysis pipeline (see stages below) |
| `data_dictionary.md` | Description of the JIRA feature schema and derived features (no actual data values) |
| `results.txt` | Model performance metrics, confusion matrix, and SHAP feature ranking on the held-out test set |
| `LICENSE` | MIT License |
| `.gitignore` | Standard Python/Jupyter ignore rules |
| `README.md` | This file |

The notebook is organized into the following stages:

1. **Setup** — installs (`shap`, `imbalanced-learn`) and imports all dependencies (first cell).
2. **Data loading** — prompts for and loads the JIRA CSV export.
3. **Preprocessing** — drops records with null Severity/Created, parses datetimes, handles
   missing values.
4. **Feature engineering** — derives lifecycle, temporal, and count features; computes
   component- and reporter-level historical aggregates; one-hot encodes categorical fields
   (`drop_first=True`); and excludes the Priority field.
5. **Class-imbalance handling** — applies SMOTE to the training split only.
6. **Model training** — trains Random Forest, Gradient Boosting, Logistic Regression, and SVM.
7. **Evaluation** — reports the Table 3 metrics (precision, recall, F1, AUC-ROC) and the
   confusion matrix on the held-out 20% test set.
8. **Interpretability** — generates the SHAP (TreeExplainer) beeswarm plot for the best model.

> **Recommended before resubmission:** rename the notebook from
> `MDPI_Paper_Real_Analysis_v3_NoPriority.ipynb` to a neutral name (e.g.
> `defect_prediction_pipeline.ipynb`). The current filename references a different journal, which
> can confuse PeerJ reviewers. If you rename it, update the reference in the table above and in
> the Usage Instructions below.

## Usage Instructions

1. Open `MDPI_Paper_Real_Analysis_v3_NoPriority.ipynb` in Google Colab (or any Jupyter
   environment).
2. Export your JIRA defects to CSV with the columns described in `data_dictionary.md`.
3. Run all cells; upload your CSV when prompted.
4. The notebook produces: the Table 3 performance metrics, the confusion matrix, and the SHAP
   beeswarm plot.

Typical runtime is 10-15 minutes for ~5,000 records on a standard Colab instance.

## Requirements

- Python 3.11
- scikit-learn 1.4
- imbalanced-learn (provides SMOTE) — installed by the first notebook cell
- shap (TreeExplainer) — installed by the first notebook cell
- pandas, numpy, matplotlib, seaborn — provided by the standard Colab environment

All runtime dependencies are installed automatically by the first cell of the notebook, so no
manual setup is required to run it in Colab.

For a fully version-pinned local environment, generate a `requirements.txt` from the same
environment used to run the notebook:

```bash
pip freeze > requirements.txt
```

and install with:

```bash
pip install -r requirements.txt
```

## Methodology

1. **Preprocessing** — remove records with null Severity or Created date, parse datetime fields,
   handle missing values.
2. **Feature engineering** — construct lifecycle features (e.g. `days_to_resolution`,
   `defect_age_days`), temporal features (`created_month`, `created_dow`, `created_quarter`),
   count features (`linked_issues_count`, `impacted_apps_count`, `labels_count`, `reopen_count`),
   and component-/reporter-level historical aggregates (`component_high_risk_rate`,
   `component_total_defects`, `reporter_history_count`). Categorical fields are one-hot encoded
   with `drop_first=True`. The Priority field is excluded to avoid label leakage, and aggregates
   are computed at the team/component level rather than the individual level to avoid using model
   outputs for individual performance attribution.
3. **Class-imbalance handling** — SMOTE applied to the training set only.
4. **Train/test split** — 80/20 split (`test_size=0.2`).
5. **Model training** — Random Forest, Gradient Boosting, Logistic Regression, and SVM.
6. **Evaluation** — precision, recall, F1, and AUC-ROC on the held-out test set (Table 3), plus a
   confusion matrix. Random Forest was the best-performing model (F1 = 0.570, AUC-ROC = 0.806).
7. **Interpretability** — SHAP (TreeExplainer) feature-importance analysis on the best model.

**Reproducibility.** The random seed is fixed (`RANDOM_STATE = 42`) throughout, so results are
deterministic for an identical input dataset matching the schema in `data_dictionary.md`.

## Citations

If this code is useful in your research, please cite the corresponding manuscript:

> Awartani, A.I. (2026). AI-driven defect prediction in enterprise IT: a machine learning
> framework using historical JIRA data. *PeerJ Computer Science* (in review).

*(No third-party dataset citation applies: the underlying data is proprietary enterprise JIRA
data and is not drawn from an external curated source.)*

## License & Contribution Guidelines

- **License.** Released under the MIT License — see the `LICENSE` file.
- **Contributing.** Contributions and replication studies on other enterprise JIRA datasets are
  welcome. Please open an issue to discuss a proposed change, or submit a pull request against the
  `main` branch. For questions about the methodology or the feature-engineering pipeline, contact
  the author (see below).

## Contact

For questions about the methodology, the feature-engineering pipeline, or collaboration on
replication studies with other enterprise JIRA datasets, please contact the author through the
corresponding-author email on the manuscript.
