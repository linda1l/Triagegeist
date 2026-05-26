# EDA and Preprocessing Summary

Generated outputs:
- `data/processed/train_processed.csv`
- `data/processed/test_processed.csv`
- `data/processed/feature_report.csv`
- `data/processed/preprocessing_report.csv`

Target:
- `triage_acuity` is retained only in train.
- High acuity is acuity 1 or 2; observed train high-acuity rate is 20.83%.

Leakage and row alignment:
- `patient_id` is preserved in processed outputs only for row alignment/submission bookkeeping.
- Modeling code must drop `patient_id` from the feature matrix.
- `disposition` and `ed_los_hours` are post-triage outcome variables and are excluded from processed model features.

Preprocessing additions:
- Invalid clinical values are set to missing and paired with explicit invalid indicators.
- Blood pressure inconsistency is flagged with `bp_inconsistent`.
- Chief complaint red-flag keyword features are included.
- `altered_mental_status_flag` is included.

Retrospective outcomes:
- Outcome analyses are post-hoc consistency checks only and must not be used for model training.
