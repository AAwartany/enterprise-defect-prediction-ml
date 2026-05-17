# AI-Driven Defect Prediction in Enterprise IT

Companion code repository for the manuscript **"AI-driven defect prediction in enterprise IT: a machine learning framework using historical JIRA data"** (PeerJ Computer Science submission).

## Overview

This repository contains the machine learning pipeline used to predict high-risk software defects from JIRA metadata in an enterprise IT environment. The pipeline trains and evaluates four supervised classifiers (Random Forest, Gradient Boosting, Logistic Regression, Support Vector Machine), applies SMOTE for class-imbalance handling, and produces SHAP-based feature importance analysis on the best-performing model.

## What's in this repository

| File | Description |
|------|-------------|
| `MDPI_Paper_Real_Analysis_v3_NoPriority.ipynb` | Full Colab notebook implementing the analysis pipeline |
| `results.txt` | Model performance metrics on the held-out test set |
| `data_dictionary.md` | Description of the JIRA feature schema (no actual data values) |
| `README.md` | This file |
| `LICENSE` | MIT License |

## What's NOT in this repository

**The raw JIRA dataset is not included.** The dataset comprises proprietary defect records from a single enterprise IT organization and cannot be redistributed due to organizational confidentiality constraints.

Researchers wishing to replicate the methodology on their own JIRA instances are welcome to use the pipeline code with their own data. The data dictionary describes the expected schema.

## How to use

1. Open `MDPI_Paper_Real_Analysis_v3_NoPriority.ipynb` in Google Colab
2. Export your JIRA defects to CSV with the columns described in `data_dictionary.md`
3. Run all cells; upload your CSV when prompted
4. The notebook produces: Table 3 metrics, SHAP beeswarm plot, confusion matrix

Typical runtime: 10–15 minutes for ~5,000 records on standard Colab.

## Methodology highlights

The pipeline deliberately excludes the Priority field from the feature set on grounds of label leakage, as Priority is structurally correlated with the Severity-derived prediction target. Historical patterns are engineered as team- and component-level aggregates rather than individual-level metrics, to avoid using model outputs as a basis for individual performance attribution.

## Reproducibility

All results in the manuscript can be regenerated from this notebook applied to a JIRA dataset matching the schema in `data_dictionary.md`. The pipeline uses:

- Python 3.11
- scikit-learn 1.4
- imbalanced-learn (SMOTE)
- shap (TreeExplainer)
- pandas, numpy, matplotlib, seaborn

All dependencies are installed by the first cell of the notebook.

Random seed is fixed (`RANDOM_STATE = 42`) for reproducibility within identical datasets.

## Citation

If this code is useful in your research, please cite the corresponding manuscript:

> Awartani, A.I. (2026). AI-driven defect prediction in enterprise IT: a machine learning framework using historical JIRA data. *PeerJ Computer Science* (in review).

## Contact

For questions about the methodology, the feature engineering pipeline, or to discuss collaboration on replication studies with other enterprise JIRA datasets, please contact the author through the corresponding-author email on the manuscript.

## License

MIT License — see `LICENSE` file.
