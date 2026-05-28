# Triagegeist Project Writeup

## Project Title

Emergency Department Triage Acuity Prediction With Chief Complaint Text and Clinical Features

## Problem Statement

This project builds an AI-powered analytical system for emergency department triage. The goal is to predict `triage_acuity` from information available at triage time, with special attention to identifying high-acuity patients (`triage_acuity` 1 or 2).

The clinical risk is asymmetric: missing a high-acuity patient is more harmful than over-prioritizing a low-acuity patient. Therefore, model selection prioritizes high-acuity recall and severe undertriage rate rather than accuracy alone.

## Data Sources

The workflow uses four input tables:

- `train.csv`: labeled triage records.
- `test.csv`: unlabeled records for submission.
- `chief_complaints.csv`: raw chief complaint text and complaint system categories.
- `patient_history.csv`: comorbidity and medical history indicators.

Rows are linked by `patient_id`. The final submission file contains `patient_id` and predicted `triage_acuity`.

## Dataset Citation And Terms Compliance

Competition requirement: all datasets used must be cited and described in the notebook, and the team must confirm that dataset use complies with access terms.

Current project data:

- Source used in this notebook: the Triagegeist CSV dataset supplied with this project workspace in the repository `data/` folder.
- The supplied files are `train.csv`, `test.csv`, `chief_complaints.csv`, `patient_history.csv`, and `sample_submission.csv`.
- If this notebook is submitted as a public Kaggle Notebook, the attached Kaggle Dataset or competition data page for these CSV files should be cited directly in the notebook metadata or writeup.

Recommended public alternatives if replacing or externally validating the current data:

- MIMIC-IV-ED, available through PhysioNet. Access is credentialed and requires signing the relevant Data Use Agreement. Citation: Johnson et al., `MIMIC-IV-ED`, PhysioNet, Version 2.2, 2023, DOI `10.13026/5ntk-km72`.
- NHAMCS Emergency Department public-use files, available from CDC/NCHS. NHAMCS public-use data should be cited to CDC/NCHS documentation for the specific year(s) used.

Compliance statement for this submission draft:

- The current notebook uses the provided Triagegeist project CSV files.
- No restricted MIMIC-IV-ED data is included in this repository.
- If MIMIC-IV-ED is used later, only credentialed users who have signed the DUA should access it, and derived outputs must comply with PhysioNet terms.
- If NHAMCS is used later, the notebook should cite the exact public-use file year(s), documentation, and CDC/NCHS source page.
- Any future external validation dataset should be documented in the notebook before model evaluation.

## EDA Summary

The target is an ordered 5-class label where lower values indicate higher urgency. In the training data, acuity 1/2 patients make up about 20.83% of records.

Key EDA findings:

- Vital signs and clinical severity features are strongly associated with acuity.
- `NEWS2`, `GCS`, oxygen saturation, respiratory rate, hypotension, and altered mental status are important high-risk signals.
- Chief complaint text is highly predictive but also highly templated, which creates shortcut-learning risk.
- Train/test distributions are broadly aligned for inspected numeric and categorical features.
- `disposition` and `ed_los_hours` are post-triage outcomes and are excluded from all modeling features.

## Preprocessing And Feature Engineering

The preprocessing pipeline merges raw tables, removes leakage variables, cleans invalid clinical values, and creates modeling-ready train/test feature tables.

Feature engineering includes:

- Derived vital signs: `mean_arterial_pressure`, `pulse_pressure`, `shock_index`.
- Clinical risk flags: `high_news2`, `low_gcs`, `low_spo2`, `tachypnea`, `fever`, `hypotension`, `severe_pain`.
- Mental status signal: `altered_mental_status_flag`.
- History aggregates: `comorbidity_count`, `high_risk_history_flag`.
- Complaint text features: cleaned complaint text, character count, word count.
- Complaint red flags: chest pain, stroke-like symptoms, trauma, bleeding, pregnancy, severe headache.
- Data quality flags: missingness indicators, invalid-value indicators, and `bp_inconsistent`.

## Modeling Approach

The notebook compares feature groups through ablation studies:

- Core clinical features.
- Core plus derived clinical flags.
- Core plus patient history.
- Core plus complaint structured features.
- Core plus complaint text.
- Full structured features.
- Full model without nurse/site identifiers.
- Full model without dominant NEWS2/GCS rule features.

Models compared:

- Logistic Regression.
- LightGBM.
- XGBoost.
- CatBoost.

Logistic Regression is used for the full ablation sweep because it is fast and interpretable. Boosting models are evaluated on the strongest candidate feature sets.

## Validation Strategy

Two internal validation strategies are used:

- Stratified random holdout: fast sanity check.
- Stratified grouped holdout by `chief_complaint_raw_clean`: stress-test against repeated complaint templates.

The grouped split is especially important for text-based models because random validation can overestimate performance when the same complaint template appears in both train and validation data.

Primary metrics:

- High-acuity recall.
- Severe undertriage rate.

Supporting metrics:

- Accuracy.
- Macro F1.
- Weighted F1.
- Mean absolute error.
- Quadratic weighted kappa.
- Confusion matrix.

## Final Model

The selected final model is:

- Model: Logistic Regression.
- Feature variant: `core + complaint text`.

This model is selected because it ties for the best grouped-validation safety score while using fewer features than full-feature alternatives. It also avoids reliance on nurse/site identifiers.

## Submission

The selected model is refit on the full processed training data and used to predict the processed test data. The final output is written to:

```text
submission.csv
```

The submission contains:

```text
patient_id,triage_acuity
```

## Future Unseen Data

The notebook includes an inference wrapper for future unseen datasets. Any future data should go through the same preprocessing and feature engineering pipeline before prediction.

If a future labeled dataset is available, it should be used as external validation and evaluated with the same high-acuity recall and undertriage metrics.

## Limitations

- The chief complaint text appears highly templated, so very high text-model performance may not generalize to real-world free-text complaints.
- Validation is internal. External hospital/time validation would be required before any clinical deployment claim.
- `disposition` and `ed_los_hours` are excluded from training because they are post-triage outcomes.
- This system is a competition submission candidate and analytical prototype, not a deployable clinical triage device.

## Reproducibility

The notebook is organized to run end-to-end:

1. Load raw data.
2. Run EDA and leakage review.
3. Apply preprocessing and feature engineering.
4. Train and compare models.
5. Select the final model.
6. Generate `submission.csv`.
7. Optionally run future unseen data inference.
