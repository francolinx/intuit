# Claude Master Handoff — Intuit SMB Underwriting Hackathon

## One-line mission
Build a small-business lending system that decides who to fund, predicts each applicant's probability of default (PD), forecasts when approved loans default over time, answers causal counterfactual questions, and produces the required four submission files exactly.

---

## Essential upload set for Claude

### If Claude upload limit is tight, upload these first
1. `claude_master_handoff.md` — this file.
2. `train.csv` — historical data with labels only for prior-approved matured loans.
3. `validation.csv` — calibration/tuning set; outcomes available for prior-approved matured loans, but declined rows have blank outcomes.
4. `test.csv` — scored application set; all outcome columns blank.
5. `data_dictionary.csv` — field names, groups, types, and whether features are intervenable.
6. `intervention_queries.csv` — the 900 causal what-if queries for Deliverable C.
7. `cohort_week_definitions.csv` — maps cohort week 1–13 to calendar date ranges for Deliverable B.
8. `submission_B_template.csv` — required 169-row B grid; overwrite prediction columns only.

### Strongly recommended if there is room
9. `submission_D_writeup_template.md` — required writeup headers and formatting rules.
10. `validate_submission.py` — final format validator.
11. `requirements.txt` — minimal dependency note; mostly `numpy` and `pandas`.
12. `manifest.json` — validator expected counts.
13. `applicant_ids.txt` — expected applicant IDs for validator.
14. `query_ids.txt` — expected query IDs for validator.

### Optional / skip if upload-limited
- `hackathon-brief.pdf` — useful for context but the important rules are summarized here.
- `README.md` and `README(1).md` — useful, but this handoff contains the necessary instructions.
- `.gitignore` — not needed for modeling.
- `._train.csv`, `._validation.csv`, `._test.csv` — AppleDouble/macOS metadata files; ignore.

---

## What the challenge is
You are a small-business lender. Each row is a loan application. The submission must act like a real lender:

1. Predict default risk for applicants.
2. Choose which applicants to approve.
3. Forecast how defaults accumulate over time in the approved portfolio.
4. Answer causal what-if questions about interventions on applicant features.
5. Defend the methodology in a short technical writeup.

This is not just a Kaggle classifier. The key difficulty is selective labels: we only observe true outcomes for loans the previous lender approved and that matured. Declined and immature loans have blank outcomes.

---

## Required final submission files
Submit exactly four files, flat in one folder, with these exact names:

```text
submission_A_decisions.csv
submission_B_trajectory.csv
submission_C_counterfactuals.csv
submission_D_writeup.pdf
```

Wrong names, missing IDs, bad ranges, or non-monotone B trajectories can make the submission fail validation.

---

## Dataset inventory and shapes

| File | Shape / Count | Purpose |
|---|---:|---|
| `train.csv` | 85,340 × 44 | Historical applications. Outcomes only for prior-approved matured loans. |
| `validation.csv` | 4,489 × 44 | Same columns; use for tuning, calibration, and sanity checks. |
| `test.csv` | 8,817 × 44 | Scored applicants; all outcome fields withheld. |
| `data_dictionary.csv` | 44 fields | Field type, group, intervenability, notes. |
| `intervention_queries.csv` | 900 rows | Queries for Deliverable C. |
| `cohort_week_definitions.csv` | 13 rows | Calendar ranges for cohort weeks. |
| `submission_B_template.csv` | 169 rows | 13 × 13 grid for Deliverable B. |

Deliverable A must include validation + test applicants: 4,489 + 8,817 = 13,306 rows.

---

## Important actual dataset facts

From the loaded files:

### `train.csv`
- Rows: 85,340
- Prior-approved matured rows with labels: 51,722
- Blank outcome rows: 33,618
- Labeled default rate: about 17.45%
- `prior_decision = 1`: 51,722 rows
- `prior_decision = 0`: 33,618 rows
- Date range: 2024-01-01 to 2025-06-29

### `validation.csv`
- Rows: 4,489
- Prior-approved matured rows with labels: 2,551
- Blank outcome rows: 1,938
- Labeled default rate: about 20.62%
- Date range: 2025-06-30 to 2025-09-28

### `test.csv`
- Rows: 8,817
- All outcomes blank
- `prior_decision = 1`: 4,964 rows
- `prior_decision = 0`: 3,853 rows
- Date range: 2025-06-30 to 2025-09-28

### Cohort weeks
Deliverable B uses 13 weekly cohorts:

| cohort_week | start_date | end_date |
|---:|---|---|
| 1 | 2025-06-30 | 2025-07-06 |
| 2 | 2025-07-07 | 2025-07-13 |
| 3 | 2025-07-14 | 2025-07-20 |
| 4 | 2025-07-21 | 2025-07-27 |
| 5 | 2025-07-28 | 2025-08-03 |
| 6 | 2025-08-04 | 2025-08-10 |
| 7 | 2025-08-11 | 2025-08-17 |
| 8 | 2025-08-18 | 2025-08-24 |
| 9 | 2025-08-25 | 2025-08-31 |
| 10 | 2025-09-01 | 2025-09-07 |
| 11 | 2025-09-08 | 2025-09-14 |
| 12 | 2025-09-15 | 2025-09-21 |
| 13 | 2025-09-22 | 2025-09-28 |

---

## Column groups

The 44 columns fall into these groups:

### Business identity
- `business_id`
- `applicant_id`
- `sector`
- `geography_region`
- `vintage_years`
- `employee_count_bucket`

### Self-reported
- `stated_annual_revenue`
- `stated_time_in_business`
- `requested_amount`
- `intended_use_of_funds`

### Bank-feed derived
These are often null when `has_linked_bank_feed = False`.
- `has_linked_bank_feed`
- `observed_monthly_revenue_avg_3mo`
- `observed_revenue_trend_3mo`
- `observed_revenue_volatility`
- `observed_cash_balance_p10`
- `observed_overdraft_count_3mo`
- `payroll_regularity_score`

### Bureau credit
- `aggregate_credit_utilization`
- `recent_inquiries_count_6mo`
- `existing_debt_obligations`
- `owner_personal_credit_band`
- `days_since_last_external_decline`

### Platform engagement
- `account_age_days`
- `platform_active_months`
- `bookkeeping_recency_days`
- `invoice_payment_delinquency_rate`
- `prior_loans_count`
- `prior_loans_default_count`
- `prior_loans_amount_total`

### Application context
- `application_timestamp`
- `application_channel`
- `multi_lender_inquiry_count_30d`
- `days_since_last_inquiry_elsewhere`
- `repeat_application_count`
- `requested_amount_to_observed_revenue`

### Prior underwriter
- `prior_underwriter_score`
- `prior_decision`
- `prior_approved_amount`

### Outcome — do not use as features for prediction
- `default_flag`
- `days_to_default`
- `days_to_full_repayment`
- `repayment_status`
- `final_recovered_amount`
- `observation_status`

---

## Intervenable features for Deliverable C
Only these features are marked intervenable in `data_dictionary.csv`:

```text
stated_annual_revenue
stated_time_in_business
requested_amount
observed_monthly_revenue_avg_3mo
observed_revenue_trend_3mo
observed_revenue_volatility
observed_cash_balance_p10
observed_overdraft_count_3mo
payroll_regularity_score
aggregate_credit_utilization
recent_inquiries_count_6mo
existing_debt_obligations
owner_personal_credit_band
invoice_payment_delinquency_rate
application_channel
multi_lender_inquiry_count_30d
```

---

## Loan economics
All funded loans use fixed terms:

- Loan amount = `requested_amount`
- Term = 60 days
- Repayment = daily ACH draws
- APR = 35% annualized
- Origination fee = 3%, collected up front

Approximate non-default gross revenue per dollar funded:

```text
origination fee + interest ≈ 0.03 + 0.35 × 60 / 365 ≈ 0.0875
```

So a fully repaid loan earns roughly 8.75% before considering losses and recovery.

Default definition:

A funded loan defaults if any of these happen:
1. 3 consecutive missed daily ACH draws.
2. 6 total missed draws over the life of the loan.
3. Outstanding balance remains above zero at day 90.

`days_to_default`, when present, is the first day 1–90 on which the loan met a default condition.

---

## Deliverable A — `submission_A_decisions.csv`

### Required rows
One row per applicant in `validation.csv` + `test.csv`.

### Required columns
```text
applicant_id
decision
predicted_pd
pd_lower_90
pd_upper_90
```

### Rules
- `decision`: 1 = approve requested amount; 0 = decline.
- `predicted_pd`: default probability in [0, 1]. Required for everyone, including declined applicants.
- `pd_lower_90 <= predicted_pd <= pd_upper_90` for every row.
- All PD fields must be in [0, 1].

### Modeling idea
Build a default-risk model on rows where outcomes are known, then calibrate it on validation labeled rows.

Important: labels are selective. Outcomes exist only for loans the prior lender approved and that matured. A model trained only on these rows learns the risk among prior-approved loans, not the whole applicant population. Correct for this in the method and writeup.

### Approval policy idea
Do not simply approve low PD. Approve when expected value is positive.

Simple formula:

```text
Expected Profit ≈ (1 - PD) × performing_profit - PD × expected_loss_given_default
```

A practical hackathon policy:
1. Estimate PD for validation + test.
2. Estimate recovery / LGD from train labeled defaulted rows if possible.
3. Tune an approval cutoff on validation.
4. Use a conservative threshold because a default can wipe out many performing-loan gains.

---

## Deliverable B — `submission_B_trajectory.csv`

### Required rows
Exactly the same 169 rows as `submission_B_template.csv`:
13 cohort weeks × 13 loan ages.

### Required columns
```text
cohort_week
loan_age_weeks
cumulative_default_rate
cdr_lower_90
cdr_upper_90
```

### Rules
- Do not change the 13 × 13 grid.
- `cumulative_default_rate`, `cdr_lower_90`, `cdr_upper_90` must be in [0, 1].
- `cdr_lower_90 <= cumulative_default_rate <= cdr_upper_90`.
- Within each cohort, `cumulative_default_rate` must be non-decreasing as `loan_age_weeks` increases.

### Fast winning method
Use a timing curve:

1. Train/estimate final PD for each applicant.
2. Choose approved applicants from Deliverable A.
3. For historical defaulted loans, compute what fraction of defaults occur by week 1, week 2, ..., week 13 using `days_to_default`.
4. For each cohort week in validation + test, calculate the average PD of your approved applicants.
5. Multiply average PD by the historical cumulative timing curve.
6. Apply monotonicity:

```python
cdr = np.maximum.accumulate(cdr)
```

Interpretation:

```text
Predicted default-by-week-a rate = final PD level × historical fraction of defaults observed by day 7a
```

---

## Deliverable C — `submission_C_counterfactuals.csv`

### Required rows
One row per `query_id` in `intervention_queries.csv`.

### Required columns
```text
query_id
predicted_pd_cf
pd_cf_lower_90
pd_cf_upper_90
```

### Rules
- `predicted_pd_cf`, `pd_cf_lower_90`, `pd_cf_upper_90` must be in [0, 1].
- `pd_cf_lower_90 <= predicted_pd_cf <= pd_cf_upper_90`.

### Core idea
Each query says:

```text
For applicant_id X, set feature_name to intervention_value. What is the new PD?
```

This is meant to be causal: `do(feature = value)`, not only a naive spreadsheet edit.

### Practical hackathon-safe counterfactual method
For each query:
1. Look up applicant row from validation + test. If not present there, search train too.
2. Copy the row.
3. Set `feature_name = intervention_value`.
4. Recompute any engineered features that logically depend on the changed feature.
   - Example: if `requested_amount` or `observed_monthly_revenue_avg_3mo` changes, recompute `requested_amount_to_observed_revenue` when possible.
5. Predict PD with the calibrated A model.
6. Use a wider uncertainty interval than A because causal counterfactuals are harder than observational prediction.

### Causal framing for the writeup
Say clearly:

- Observational prediction asks: “Given what we see, what risk do businesses like this usually have?”
- Interventional prediction asks: “If this one feature were forced to a new value, what would risk be?”

Do not overclaim causal certainty. Say the model is a structured approximation using intervenable features, dependency-aware recomputation, and wider uncertainty for weaker causal identification.

---

## Deliverable D — `submission_D_writeup.pdf`

Use `submission_D_writeup_template.md`. Required sections, in order:

1. Problem framing & assumptions violated
2. Methodology
3. Causal reasoning & counterfactual methodology
4. Calibration & uncertainty quantification
5. Limitations & what we'd do differently

Format:
- Max 4 pages of body content, excluding references.
- Minimum 11 pt font.
- Minimum 0.75 inch margins.
- No executive summary.
- No marketing fluff.
- Section 3 on causality is weighted most heavily.

Writeup angle:

```text
We treated this as lending under selective labels, not generic classification. We modeled default risk, converted calibrated PDs into profit-aware decisions, modeled timing separately using empirical default curves, handled counterfactuals as interventions rather than naive row edits where possible, and reported uncertainty because default risk and causal effects are not known exactly.
```

---

## Recommended modeling pipeline for Claude

### Step 1 — Load and classify columns
Use `data_dictionary.csv` to separate:
- features
- outcomes to exclude
- intervenable features
- categorical vs numeric fields

Exclude all outcome columns from features:

```python
OUTCOME_COLS = [
    "default_flag",
    "days_to_default",
    "days_to_full_repayment",
    "repayment_status",
    "final_recovered_amount",
    "observation_status",
]
```

Also do not use `applicant_id` as a predictive feature. Be cautious with `business_id`; it may not transfer cleanly and could leak identity/memorization, so likely exclude it from model features.

### Step 2 — Feature engineering
Useful features:

```text
requested_amount / stated_annual_revenue
requested_amount / observed_monthly_revenue_avg_3mo
existing_debt_obligations / stated_annual_revenue
existing_debt_obligations / observed_monthly_revenue_avg_3mo
prior_default_rate = prior_loans_default_count / max(prior_loans_count, 1)
has_prior_default = prior_loans_default_count > 0
missing_bank_feed indicators
month / week from application_timestamp
is_prior_declined = prior_decision == 0
prior_approved_amount / requested_amount
```

Keep missingness indicators for bank-feed columns because missingness is meaningful.

### Step 3 — Train PD model
Good models:
- Best practical: CatBoost / LightGBM / XGBoost if installable.
- Safe minimal: sklearn `HistGradientBoostingClassifier`, `RandomForestClassifier`, or logistic regression with preprocessing.

Training label:
- Use rows with `default_flag` not null.
- `y = default_flag`.

Calibration:
- Use labeled rows in validation if allowed.
- If building from train only, calibrate using a split from train and sanity-check on validation labeled rows.
- Use isotonic or Platt scaling if available; otherwise use validation bin calibration.

Selective-label handling:
- Include `prior_underwriter_score`, `prior_decision`, and prior-approved information carefully.
- Consider training a prior approval/propensity model and weighting approved-labeled samples inversely by approval probability.
- At minimum, discuss selective labels honestly in writeup.

### Step 4 — Profit-aware decisions
Approximate economics:

```python
performing_profit_rate = 0.03 + 0.35 * 60 / 365  # about 0.0875
```

Estimate average loss given default from labeled defaults if `final_recovered_amount` is usable:

```python
lgd = 1 - recovered_amount / prior_approved_amount
```

If recovery values are missing or unreliable, use a conservative assumed LGD, e.g., 0.65–0.90, and tune threshold on validation.

Decision rule:

```python
expected_profit = requested_amount * ((1 - pd) * performing_profit_rate - pd * lgd)
decision = (expected_profit > 0).astype(int)
```

Tune a safety margin / cutoff on validation.

### Step 5 — Build B trajectory
- Assign validation + test applications to cohort weeks by `application_timestamp` and `cohort_week_definitions.csv`.
- Use only applicants approved by our policy.
- For each cohort, average the approved applicants' PDs.
- Create default timing curve from training defaulted rows using `days_to_default`.
- Multiply cohort mean PD by the timing curve.
- Clip to [0, 1].
- Apply cumulative max within cohort.
- Add 90% intervals.

### Step 6 — Build C counterfactuals
- Join each query to applicant features.
- Apply intervention.
- Recompute dependent ratio features.
- Predict calibrated counterfactual PD.
- Intervals should usually be wider than A.

### Step 7 — Validate output files
Final folder must contain exactly:

```text
submission_A_decisions.csv
submission_B_trajectory.csv
submission_C_counterfactuals.csv
submission_D_writeup.pdf
```

Run:

```bash
pip install -r requirements.txt
python validate_submission.py path/to/submission_folder
```

If using the uploaded flat files instead of an `expected_ids/` folder, put `manifest.json`, `applicant_ids.txt`, and `query_ids.txt` in an `expected_ids/` folder next to `validate_submission.py`, or run with:

```bash
python validate_submission.py path/to/submission_folder --expected-ids path/to/expected_ids
```

Keep fixing until result is `PASS`.

---

## Schema checklist for generated files

### A checklist
- [ ] 13,306 rows exactly.
- [ ] All validation + test `applicant_id`s included exactly once.
- [ ] Columns: `applicant_id`, `decision`, `predicted_pd`, `pd_lower_90`, `pd_upper_90`.
- [ ] `decision` only 0 or 1.
- [ ] PD values in [0, 1].
- [ ] Interval order valid.

### B checklist
- [ ] 169 rows exactly.
- [ ] Same grid as `submission_B_template.csv`.
- [ ] Columns: `cohort_week`, `loan_age_weeks`, `cumulative_default_rate`, `cdr_lower_90`, `cdr_upper_90`.
- [ ] Values in [0, 1].
- [ ] Interval order valid.
- [ ] `cumulative_default_rate` non-decreasing within each cohort.

### C checklist
- [ ] 900 rows exactly.
- [ ] One row per `query_id`.
- [ ] Columns: `query_id`, `predicted_pd_cf`, `pd_cf_lower_90`, `pd_cf_upper_90`.
- [ ] Values in [0, 1].
- [ ] Interval order valid.

### D checklist
- [ ] PDF named `submission_D_writeup.pdf`.
- [ ] Uses required five sections in order.
- [ ] Max 4 pages body.
- [ ] Minimum 11 pt font and 0.75 inch margins.
- [ ] Section 3 carefully distinguishes observational vs interventional prediction.

---

## Suggested implementation file names
Ask Claude to produce code like this:

```text
00_explore_data.py
01_train_model.py
02_make_submission_A.py
03_make_submission_B.py
04_make_submission_C.py
05_writeup_draft.md
run_all.py
```

Or, if time is short, one notebook/script:

```text
make_submission.py
```

That script should create a folder like:

```text
submission/
  submission_A_decisions.csv
  submission_B_trajectory.csv
  submission_C_counterfactuals.csv
  submission_D_writeup.pdf
```

---

## Prompt to give Claude after uploading files
Copy/paste this to Claude:

```text
You are helping me win the Intuit SMB Underwriting Hackathon. I uploaded the essential files and a master handoff. Build a practical, valid, high-scoring solution.

Goals:
1. Read train.csv, validation.csv, test.csv, data_dictionary.csv, intervention_queries.csv, cohort_week_definitions.csv, and submission_B_template.csv.
2. Build a calibrated default probability model while acknowledging selective labels: outcomes are observed only for prior-approved matured loans.
3. Generate Deliverable A: submission_A_decisions.csv for validation + test applicants, with approve/decline decisions, predicted_pd, and 90% PD intervals.
4. Generate Deliverable B: submission_B_trajectory.csv using the exact 13 × 13 template grid, with monotone cumulative default forecasts by cohort week and loan age.
5. Generate Deliverable C: submission_C_counterfactuals.csv for all intervention queries, treating them as do(feature=value) interventions; recompute dependent features where logically necessary.
6. Draft Deliverable D: a concise technical writeup under the required section headers, with special focus on causal reasoning and calibration.
7. Make sure outputs pass validate_submission.py.

Use a fast but defensible approach: gradient boosting or a strong sklearn baseline, feature engineering around revenue/debt/loan amount/prior defaults/bank-feed missingness, profit-aware approval policy, empirical default timing curve for B, and wider uncertainty intervals for C. Prioritize valid files first, then improve model quality.

Do not use outcome columns as features. Do not change required file names or schemas. Make B non-decreasing within each cohort. Explain every tradeoff clearly.
```

---

## Fast action plan for the hackathon

### First 45 minutes
- Load data.
- Build minimal PD model.
- Generate valid A/B/C files.
- Run validator.

### Next 90 minutes
- Improve features.
- Tune approval threshold based on expected profit.
- Improve calibration.
- Make B timing curve more realistic.

### Final 60 minutes
- Fill D writeup with honest, strong methodology.
- Validate again.
- Submit exactly four files.

---

## Winning narrative
The strongest story is:

```text
We did not treat this as generic classification. We treated it as lending under selective labels and incomplete observability. Our system estimates calibrated PD, converts it into an expected-profit funding policy, separates final default risk from default timing, approximates counterfactual interventions with dependency-aware feature updates, and uses uncertainty intervals to avoid false precision.
```
