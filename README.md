# Predicting Clinical Trial Enrollment Failure

Capstone project using the AACT database (ClinicalTrials.gov) to predict whether a
trial will be terminated for low enrollment, and how much eligibility criteria matter.
The proposal is in `docs/`.

## Data setup

### 1. Create a free AACT account
Sign up at https://aact.ctti-clinicaltrials.org/sign_up. After confirming your email,
your database username and password are shown on your AACT account page.
Each team member should use their own account.

### 2. Install
```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env      # then put your AACT username/password in .env
```

### 3. Run the pipeline
```bash
.venv/bin/python scripts/01_test_connection.py   # check login, see status counts and table list
.venv/bin/python scripts/02_extract.py           # pull cohort tables to data/raw/*.parquet
.venv/bin/python scripts/03_build_dataset.py     # build data/processed/trials.parquet
```

Step 2 downloads roughly a few hundred MB and can take 10–30 minutes.
To keep everyone on the **same data**, one person runs step 2 and shares
`data/raw/` (e.g. on a shared drive) instead of everyone extracting on a different day.
The extraction date is recorded in `data/raw/SNAPSHOT.txt`.

## Cohort and label

- **Cohort:** interventional trials with `start_date` from 2008-01-01 to 2020-12-31
  (change `START`/`END` in `scripts/02_extract.py`).
- **label = 1:** status TERMINATED and `why_stopped` is about enrollment/accrual.
- **label = 0:** status COMPLETED.
- **label = NaN:** other stop reasons, withdrawn, still running, unknown. These rows are kept
  for the statistical and competing-risks analysis but excluded from the classifier.

`stop_reason` is a first-pass keyword classifier (v0) in `scripts/03_build_dataset.py`.
The labeling workstream will validate it against hand labels and replace it.

## Leakage warning

These columns describe the trial's **final** state, not what was known at the start.
Do not use them as model features as-is:

| Column | Problem |
|---|---|
| `enrollment`, `enrollment_type` | becomes the *actual* count after the trial ends (tiny for terminated trials) |
| `completion_date`, anything in `calculated_values` about duration | known only at the end |
| `n_facilities`, `n_countries` | sites are added/removed while recruiting |

## Layout

```
src/db.py                   AACT connection (reads .env)
scripts/01..03              data pipeline
data/raw/                   extracted tables (git-ignored)
data/processed/             analysis dataset (git-ignored)
notebooks/                  exploration
docs/                       proposal
```
