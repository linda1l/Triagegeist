# Triagegeist: Emergency Department Triage Acuity Prediction

This repository contains our team project for the Triagegeist emergency triage challenge. The project builds a machine learning workflow to predict five-level emergency department triage acuity from information available at or around triage.

The model uses structured triage data, patient history, and chief complaint text. The workflow focuses on leakage control, high-acuity patient detection, and reproducible submission generation.

## Project Overview

Emergency department triage requires rapid assessment of patient urgency. In this project, we predict `triage_acuity` using triage-stage information such as:

- demographics
- arrival characteristics
- vital signs
- clinical scores
- pain variables
- mental status
- chief complaint text
- patient history

Post-triage outcome variables such as `disposition` and `ed_los_hours` are excluded from all predictive features because they would not be available when triage acuity is assigned.

## Repository Contents

```text
Triagegeist/
├── README.md
├── Triagegeist.ipynb
├── Emergency Department Triage Acuity Prediction.pdf
└── submission.csv
```

The competition data files are not redistributed in this repository. They should be accessed through the official Kaggle competition input.

## Data

The workflow expects the following competition files:

```text
train.csv
test.csv
chief_complaints.csv
patient_history.csv
sample_submission.csv
```

On Kaggle, these files should be attached through the official Triagegeist competition input. The expected path is:

```text
/kaggle/input/competitions/triagegeist/
```

The notebook also includes path handling so the data directory can be adjusted if Kaggle uses a different input path.

## Methodology

The project follows these main steps:

1. Data loading and integrity checks
2. One-to-one merging of chief complaint and patient history tables using `patient_id`
3. Leakage exclusion for post-triage outcome variables
4. Exploratory analysis of target imbalance, missingness, clinical variables, categorical variables, text fields, and patient history
5. Feature engineering and preprocessing
6. Model comparison across multiple feature variants and algorithms
7. Safety-focused validation and final submission generation

## Feature Engineering

Feature engineering includes:

* missingness indicators
* invalid-value indicators
* `missing_vital_count`
* pulse pressure
* mean arterial pressure
* shock index
* clinical risk flags
* `comorbidity_count`
* `high_risk_history_flag`
* complaint red-flag indicators
* TF-IDF features from cleaned chief complaint text

Clinical cleaning converts implausible values in selected vital-sign and scoring fields to missing values while retaining indicator features.

## Models

The notebook evaluates:

* Logistic Regression
* LightGBM
* XGBoost
* CatBoost

Feature variants include core clinical features, derived clinical flags, patient history features, structured complaint features, TF-IDF complaint text features, full structured inputs, full inputs without operational identifiers, and a version excluding dominant clinical-rule features.

The final submission uses a small soft-voting ensemble of the three highest-ranked Logistic Regression candidates.

## Validation

High-acuity cases are defined as acuity levels 1 and 2. Model selection prioritizes:

* high-acuity recall
* severe undertriage

Supporting metrics include:

* accuracy
* macro F1
* weighted F1
* mean absolute error
* quadratic weighted kappa

Because chief complaint text is repetitive, final model selection uses five-fold `StratifiedGroupKFold` grouped by cleaned chief complaint text. This prevents identical cleaned complaint strings from appearing in both training and validation within the same fold. It does not remove all semantic overlap between clinically similar complaint phrases.

## Results

The strongest single model is tuned Logistic Regression with the compact core clinical + complaint text feature set.

Five-fold grouped cross-validation performance:

```text
Accuracy:             0.9999 ± 0.0001
QWK:                  1.0000 ± 0.0001
High-acuity recall:   1.0000 ± 0.0000
Severe undertriage:   0.0000 ± 0.0000
```

These scores are treated as internal validation evidence only. Repeated complaint templates and rule-like structure may limit transfer to external emergency department text.

The final ensemble generated `submission.csv` with 20,000 predictions.

## How to Run

1. Open `Triagegeist.ipynb` in Kaggle.
2. Attach the official Triagegeist competition data as notebook input.
3. Confirm the input files are available under:

```text
/kaggle/input/competitions/triagegeist/
```

4. Run the notebook from top to bottom.
5. The final prediction file will be written as:

```text
/kaggle/working/submission.csv
```

## Reproducibility

The notebook documents the workflow from data loading to submission generation. The same preprocessing rules are applied to training and test data. Leakage exclusions, grouped cross-validation settings, feature variants, model comparisons, and final prediction generation are included in the notebook.

## Limitations

This project is a competition proof of concept, not a deployable triage system.

External validation is required because:

* acuity labels reflect recorded triage decisions, not an independent clinical gold standard;
* complaint templates may be specific to this dataset or hospital documentation style;
* grouped validation reduces exact-template memorisation but does not establish safety under prospective use;
* performance may differ across hospitals, languages, documentation systems, and triage protocols.

## Team

Team: machine now learning

Members:

* Linda Zhang
* Jojojo123
* soda444

## License and Data Use

The code and project materials in this repository may be shared for competition review and reproducibility.

The original Triagegeist competition data is not redistributed here. Users should access the data through the official Kaggle competition page and comply with the competition rules.
