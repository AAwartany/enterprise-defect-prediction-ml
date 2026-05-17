# Data Dictionary — JIRA Feature Schema

This document describes the expected schema of the JIRA export CSV used as input to the analysis pipeline. Researchers replicating the methodology on their own JIRA instances should export defect data with the columns described below.

The raw dataset used in the manuscript is not included in this repository due to organizational confidentiality constraints. The schema below allows replication on any JIRA instance with a similar configuration.

## Required columns (exact names expected by the pipeline)

| Column | Type | Description |
|--------|------|-------------|
| `Key` | string | JIRA issue identifier (e.g., `PROJ-1234`). Used only as a unique row ID. |
| `Severity` | string | Defect severity level. Used as the source of the prediction target. Expected values: Critical, High, Severe, Medium, Normal, Low, Minor |
| `Created` | datetime | Timestamp when the defect was logged |
| `Resolved` | datetime | Timestamp when the defect was resolved (may be null for open defects) |
| `Updated` | datetime | Timestamp of the most recent update |
| `Status` | string | Current lifecycle status (e.g., Open, In Progress, Resolved, Closed) |
| `Priority` | string | JIRA Priority field (high/medium/low). **NOTE: This is deliberately excluded from the feature matrix due to label leakage with Severity, but is kept in the input schema for inspection** |
| `Component/s` | string | Component(s) the defect belongs to (comma-separated if multiple) |
| `Issue Type` | string | JIRA issue type (e.g., Bug, Defect, Incident) |
| `Bug Category` | string | Custom field categorizing the defect (e.g., UI Issue, Development Issue, Business Requirement) |
| `Defect Type` | string | Custom field describing the technical nature of the defect (e.g., UI, Functional, Data, Integration) |
| `Reporter` | string | Person who logged the defect. Used only in aggregate form (Reporter History Count); not used as individual identifier. |
| `Linked Issues` | string | Comma-separated list of linked issue keys; used for derived feature `linked_issues_count` |
| `IT Impacted Applications` | string | Comma-separated list of impacted applications; used for derived feature `impacted_apps_count` |
| `Re-open counter` | integer | Number of times the defect was re-opened after resolution |
| `Labels` | string | Comma-separated tags |
| `Assignee` | string | **NOT used as a feature.** Excluded from analysis to avoid individual performance attribution. |
| `Developer` | string | **NOT used as a feature.** Same reason as Assignee. |
| `Bug Severity` | string | Alternative severity field; may duplicate `Severity` in some JIRA configurations |

## Derived features computed by the pipeline

The pipeline engineers these features from the raw columns above:

| Derived feature | Source | Description |
|-----------------|--------|-------------|
| `high_risk` (target) | Severity | Binary label: 1 if Severity ∈ {Critical, High, Severe}, else 0 |
| `days_to_resolution` | Resolved − Created | Defect lifecycle duration in days |
| `defect_age_days` | Updated − Created | Days since the defect was logged |
| `created_month` | Created | Month (1–12) the defect was logged |
| `created_dow` | Created | Day of week (0=Mon, 6=Sun) |
| `created_quarter` | Created | Calendar quarter (1–4) |
| `reopen_count` | Re-open counter | Numeric count of re-openings |
| `linked_issues_count` | Linked Issues | Count of comma-separated linked items |
| `has_linked_issues` | Linked Issues | Binary: 1 if non-null, 0 otherwise |
| `impacted_apps_count` | IT Impacted Applications | Count of impacted applications |
| `labels_count` | Labels | Count of tags |
| `component_high_risk_rate` | Component/s × Severity | Component-level aggregate: fraction of historical defects classified high-risk |
| `component_total_defects` | Component/s | Component-level aggregate: total defect count |
| `reporter_history_count` | Reporter | Number of prior defects logged by the same reporter (used as team-level aggregate) |

## Categorical encodings

The pipeline one-hot encodes the following categorical features (drop_first=True):

- `Issue Type`
- `Bug Category`
- `Defect Type`
- `component_grouped` (top-15 components, others grouped as "Other")

## Notes on adaptation to other JIRA instances

JIRA configurations vary considerably across organizations. To adapt this pipeline:

1. Map your local column names to the names in the table above. If columns differ, edit the `df.rename({...})` step in the notebook.
2. If your organization uses different Severity values, update the `high_risk_severities` list in the notebook accordingly.
3. Custom fields like `Bug Category` and `Defect Type` are organization-specific. The pipeline will handle whatever string values appear in your data.
4. The Priority exclusion is recommended regardless of organization, as Priority and Severity are structurally correlated in most JIRA configurations.

## Dataset characteristics in the manuscript

For reference, the dataset used in the manuscript had the following characteristics after filtering:

- **Total records:** ~4,821 (after dropping null severity, null Created date, etc.)
- **High-risk class:** 26.49%
- **Time span:** Multiple years of release cycles (specific dates withheld)
- **Source:** A single multi-product enterprise IT organization
- **Unique components:** Withheld for confidentiality
- **Unique reporters:** Withheld for confidentiality
