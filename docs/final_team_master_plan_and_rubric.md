# Intuit SMB Underwriting Hackathon — Final Team Master Plan & Rubric

**Team:** Smoke Labs  
**Purpose:** Align the team on the best-of-all-worlds strategy from Claude, Kimi Swarm, Gemini Deep Think, NotebookLM, and our own red-team review.  
**Current recommendation:** Do **not** rebuild the submission. Claude’s current version is valid and strong. Only make low-risk final tuning if it clearly improves validation economics or trajectory quality.

---

## 1. Executive Decision

We have a strong, validator-passing submission built by Claude:

- `submission_A_decisions.csv`
- `submission_B_trajectory.csv`
- `submission_C_counterfactuals.csv`
- `submission_D_writeup.pdf`

The current submission passes the validator:

```text
RESULT: PASS  (0 errors, 0 warning(s))
Your submission is correctly formatted and ready to upload.
```

**Strategic posture:**

> Preserve the validated Claude pipeline as the base. Run only last-mile, low-risk ablations: threshold sensitivity for A, risk-bucket timing for B, C group diagnostics, and interval sanity checks. If those improve results without breaking validation or writeup coherence, update. Otherwise submit the current version.

---

## 2. What the Challenge Really Rewards

This is not a generic classifier contest. It is a lender simulation with four deliverables:

| Deliverable | File | What it tests |
|---|---|---|
| A | `submission_A_decisions.csv` | Calibrated PD + profitable approve/decline policy |
| B | `submission_B_trajectory.csv` | Default timing curve for our approved loans |
| C | `submission_C_counterfactuals.csv` | Causal/interventional counterfactual PDs |
| D | `submission_D_writeup.pdf` | Technical explanation, especially causal reasoning |

Core traps:

1. **Selective labels / reject inference:** labels exist only for historically approved loans.
2. **Profit is not AUC:** a good classifier can still approve an unprofitable book.
3. **Timing matters:** B is cumulative default trajectory, not just final PD.
4. **Causal ≠ perturbation:** C asks for `do(feature=value)`, not naive row edits.
5. **Calibration matters:** every A/B/C point estimate needs a 90% interval.
6. **Validation safety matters:** wrong file names, IDs, schemas, ranges, or monotonicity can kill the submission.

---

## 3. Current Claude Submission Diagnostics

### 3.1 Data facts

| Item | Value |
|---|---:|
| Train rows | 85,340 |
| Validation rows | 4,489 |
| Test rows | 8,817 |
| A rows required | 13,306 |
| Train prior approved | 51,722 |
| Train prior declined | 33,618 |
| Train non-default labels | 42,698 |
| Train default labels | 9,024 |
| Train unlabeled rows | 33,618 |
| Validation labeled rows | 2,551 |
| Train labeled default rate | ~17.4% |
| Validation-era default rate | ~20.6% |
| Days to default mean | 43.08 days |
| Days to default median | 37 days |
| Days to default max | 90 days |

### 3.2 A: Lending policy diagnostics

| Metric | Current value |
|---|---:|
| Total A rows | 13,306 |
| Total approvals | 2,075 |
| Approval rate | 15.59% |
| Validation approval rate | 15.64% |
| Test approval rate | 15.57% |
| Mean PD among approved | 6.66% |
| Mean PD among declined | 31.42% |
| Max approved PD | 8.83% |
| Min declined PD | 8.93% |
| Validation AUC | 0.7484 |
| Validation Brier | 0.1354 |
| Validation log loss | 0.4324 |
| Mean A interval width | 0.1279 |

Claude’s writeup states:

- LGD = 0.9065
- Performing-loan return = 0.0875
- Analytic break-even PD ≈ 8.81%
- Deployed cutoff ≈ 8.9%
- Chosen rule is profit-aware, not just AUC-aware

### 3.3 B: Default trajectory diagnostics

| Metric | Current value |
|---|---:|
| B rows | 169 |
| Cohorts | 13 |
| Loan ages | 13 |
| Monotonicity failures | 0 |
| Week-13 CDR range | 6.50%–6.77% |
| Mean B interval width | 0.0217 |

Current B method:

```text
CDR(cohort, week) = approved-cohort mean PD × empirical fraction of defaults realized by week
```

Strength: robust, simple, monotone, preserves day-90 spike.  
Weakness: not highly personalized by risk bucket.

### 3.4 C: Counterfactual diagnostics

| Metric | Current value |
|---|---:|
| C rows | 900 |
| Mean counterfactual PD | 29.01% |
| Mean C interval width | 26.15% |
| Mean absolute PD movement | 2.85 percentage points |

By intervenability:

| Intervenable? | Query count | Mean absolute PD move | Mean interval width |
|---|---:|---:|---:|
| `True` | 726 | 0.0337 | 0.2461 |
| `False` | 174 | 0.0070 | 0.3256 |

This is good: non-intervenable/proxy features already produce small movement but wider uncertainty. Therefore, we should **not** blindly force all non-intervenable features to zero movement unless a new ablation clearly proves it improves C.

---

## 4. Best-of-All-Worlds Synthesis

### 4.1 What we keep from Claude

Claude built the strongest execution backbone:

- 5-member LightGBM ensemble
- Deployment-era recalibration
- Profit-aware decision threshold near break-even
- Clean A/B/C outputs
- Validated schema and ID coverage
- Strong 3-page technical writeup
- Causal humility in C
- Adaptive uncertainty intervals
- Overlap/propensity stress test

**Team decision:** Claude’s current submission is our base. Do not rebuild from scratch.

### 4.2 What we borrow from Kimi Swarm

Kimi’s strongest ideas:

1. **Two-stage propensity thinking:** model the prior lender’s approval function to diagnose overlap and selective labels.
2. **OOD/low-propensity interval widening:** applicants far from historically approved support deserve wider intervals and/or a risk buffer.
3. **Risk-bucket timing for B:** high-risk loans may default earlier; low-risk loans may have more day-90 sweep behavior.
4. **C feature-group logic:** differentiate true levers, financial health, credit, bank-feed behavior, prior history, proxy/context, and colliders.

**Team decision:** Use these as diagnostic and interval logic, not as a full rebuild.

### 4.3 What we borrow from Gemini Deep Think

Gemini’s strongest ideas:

1. **Threshold sensitivity:** 15.6% approval might be slightly conservative; scan around the cutoff.
2. **Calibration is a scoring moat:** intervals should be adaptive, not fixed-width.
3. **B should preserve the day-90 mass:** do not smooth away the outstanding-balance sweep.
4. **C should respect data dictionary intervenability:** but do not overreact by zeroing everything.
5. **Final validation discipline:** PASS beats fancy broken methods.

**Team decision:** Use Gemini as a red-team, not as final authority. Some suggestions are too aggressive or speculative.

### 4.4 What we reject

Do **not** spend final time on:

- Full generative augmentation / GMM pseudo-labeling of declined loans
- Full causal forest / EconML pipeline
- Full Cox/sksurv replacement of the A model
- Full LGD model unless everything else is already locked
- Approval rule based only on `upper_90 < break-even` — too conservative in validation
- Forcing all non-intervenable C deltas to exactly zero without evidence
- Major writeup rewrite that could break clarity or page limits

---

## 5. Final Recommended Plan

### Phase 0 — Preserve the current submission

Before changing anything:

1. Copy the current validated files into a frozen folder:

```text
final_submission_frozen/
  submission_A_decisions.csv
  submission_B_trajectory.csv
  submission_C_counterfactuals.csv
  submission_D_writeup.pdf
```

2. This is our fallback. If any tuning goes wrong, upload the frozen version.

---

### Phase 1 — A: Threshold sensitivity only

Goal: see if the approval cutoff can increase realized validation profit without taking excess default risk.

Run a cutoff scan:

```text
PD cutoff range: 0.0850 to 0.0950
Step: 0.0005 or smaller
Metrics:
- validation approvals
- approved bad rate
- dollar-weighted profit
- per-dollar profit
- test approval count
- approved mean PD
```

Current result from our scan suggests:

| Rule | Validation approvals | Bad rate | Rough fixed-LGD dollar profit |
|---|---:|---:|---:|
| Current cutoff | 599 | 7.01% | ~$218k |
| PD ≤ 0.0900–0.0910 | 632 | 6.80% | ~$265k |
| Upper 90% PD < break-even | 329 | 5.78% | ~$197k |

**Decision rule:**

- If the best nearby cutoff improves validation dollar profit by >5% and approved bad rate remains below break-even, switch.
- If the improvement is tiny, keep current.
- If A changes, recompute B because B depends on approved loans.

---

### Phase 2 — B: Risk-bucket timing test

Current B is safe. Optional improvement:

1. Split historical defaults into risk buckets by predicted PD or `prior_underwriter_score`.
2. Build empirical timing curves for each bucket.
3. For each approved cohort, blend the timing curves using that cohort’s risk mix.
4. Force monotonicity with cumulative max.
5. Preserve week-13 sweep: week 13 final CDR should equal the cohort final mean PD.
6. Compare with current global curve on validation-era timing if possible.

**Decision rule:**

- Keep risk-bucket B only if it is stable, interpretable, validator-safe, and does not create weird cohort artifacts.
- Otherwise keep Claude’s current global timing curve.

---

### Phase 3 — C: Keep current unless diagnostics fail

Current C is defensible:

- Full response-surface perturbation for point estimate
- Deterministic dependent-feature recomputation
- Wider intervals for weakly identified/proxy/structural groups
- Non-intervenable features already have tiny mean movement and wider intervals

Do **not** force zero movement for all non-intervenable features unless:

- those features show large unrealistic PD swings, or
- the official scoring/writer strongly indicates `intervenable=False` means zero causal effect

Final C diagnostics to print:

```text
By feature group:
- count
- mean baseline PD
- mean counterfactual PD
- mean absolute movement
- mean interval width

By intervenable flag:
- count
- mean absolute movement
- mean interval width
```

Decision rule:

- If `intervenable=False` mean movement remains near 0.7 percentage points and intervals are wide, keep.
- If any structural/proxy group has huge swings, shrink those specific groups only.

---

### Phase 4 — Intervals: sanity, not rebuild

Current intervals are strong: validation 90% interval coverage is reported as 93.3% in the writeup.

Run sanity check:

```text
A and C rows with width < 0.005
A and C rows with width > 0.95
Coverage by validation PD bin
Coverage by bank-feed linked vs missing
Coverage by prior-approval propensity bucket if available
```

Decision rule:

- If ultra-narrow intervals occur only at clipped PD≈0 or PD≈1, leave or minimally widen.
- If ultra-narrow intervals occur in normal PD regions, widen them.
- Do not over-widen every interval; calibration score may reward sharpness too.

---

### Phase 5 — D: Only update if outputs changed

Claude’s D is excellent. It is concise, technical, and regulator-aware.

Only edit D if:

- threshold changes materially,
- B changes to risk-bucket timing,
- C group rules change, or
- interval construction changes.

Do not rewrite the whole PDF.

---

## 6. Team Roles for Final Push

| Role | Owner | Responsibility |
|---|---|---|
| Builder | Claude operator | Run threshold/B/C/interval checks and generate final files |
| Validator | Teammate 1 | Run validator after every output change; check row counts/ranges/NaNs |
| Risk red-team | Teammate 2 | Review A threshold, C causal logic, and interval sanity |
| Writeup owner | Teammate 3 | Update PDF only if final outputs change |
| Submitter | Franco | Upload exactly four files; verify portal accepts them |

If there are only 1–2 people, collapse roles:

1. Builder/validator
2. Red-team/submitter

---

## 7. Internal Rubric for Judging Our Final Plan

This is our internal scoring rubric. It is **not** an official scoring formula. It is designed to estimate how likely the submission is to beat other teams.

| Category | Weight | What we reward |
|---|---:|---|
| A: PD + profitable decisions | 30 | Calibration, discrimination, expected-profit approval, sensible approval rate |
| B: trajectory | 15 | Monotonic cohort timing, day-90 spike handling, risk/cohort adjustment |
| C: counterfactuals | 20 | Clear causal distinction, feature-group logic, dependent-feature recomputation, uncertainty |
| Calibration / intervals | 15 | 90% coverage, adaptive width, not too narrow or too wide |
| Validator safety | 10 | Exact files, rows, schemas, IDs, ranges, monotonicity, no NaNs |
| Writeup | 10 | Concise, technical, honest, regulator-defensible |
| **Total** | **100** |  |

### 7.1 Current Claude submission score

| Area | Weight | Score | Reasoning |
|---|---:|---:|---|
| A: PD + decisions | 30 | 27 | AUC 0.7484, strong calibration, profit-aware cutoff, approved mean PD 6.66%; slight risk approval rate is conservative |
| B: trajectory | 15 | 12 | Valid, monotone, preserves day-90 spike; simple global timing curve may miss risk timing heterogeneity |
| C: counterfactuals | 20 | 15.5 | Strong framing and intervals; point estimates still use full perturbation for all groups |
| Calibration / intervals | 15 | 13.5 | Reported 93.3% validation coverage; mild concern about clipped ultra-narrow intervals |
| Validator safety | 10 | 10 | Validator PASS, exact rows/schemas/IDs/ranges |
| Writeup | 10 | 9 | Very strong 3-page technical writeup; dense but credible |
| **Total** | **100** | **87.0–89.0** | Strong A-/A- contender |

Recommended headline grade: **89 / 100 — A-**.

### 7.2 Best-of-all-worlds final plan expected score

Assuming only low-risk improvements are made and validator still passes:

| Area | Weight | Expected score | Reasoning |
|---|---:|---:|---|
| A: PD + decisions | 30 | 28 | Threshold sensitivity may improve validation dollar profit while preserving risk discipline |
| B: trajectory | 15 | 13 | Risk-bucket timing may better capture earlier high-risk defaults and week-13 safe-bucket mass |
| C: counterfactuals | 20 | 16 | Current C is already good; minor group diagnostics/interval sanity improves confidence |
| Calibration / intervals | 15 | 14 | Adaptive/diagnostic intervals are strong if ultra-narrow rows are patched |
| Validator safety | 10 | 10 | Maintain PASS as non-negotiable |
| Writeup | 10 | 9 | Keep strong D; update only if outputs change |
| **Total** | **100** | **90–91** | Best practical final submission |

### 7.3 Why not chase a moonshot?

| Moonshot | Estimated score if perfect | Realistic risk-adjusted score | Why not now |
|---|---:|---:|---|
| Full causal forest for C | 92 | 76 | High implementation risk, weak support/positivity, can break C |
| Full generative reject inference | 91 | 72 | Pseudo-labels can poison calibration and P&L |
| Full Cox/survival replacement | 88 | 78 | Good B, weaker A, dependency/debug risk |
| Upper-bound-only approval | 84 | 82 | Too conservative; loses profitable volume |
| Force zero for all non-intervenable C | 86 | 80 | Too blunt; may under-answer official C queries |

---

## 8. Final Claude Instruction

Paste this into Claude for the final pass:

```text
We synthesized Claude + Kimi + Gemini. Do not rebuild. Use the current validator-passing submission as the base.

Final low-risk checks only:

1. Freeze the current submission as fallback.
2. Run A threshold sensitivity from 0.085 to 0.095.
   - Report validation approvals, bad rate, dollar-weighted profit, per-dollar profit, test approval count.
   - Switch only if a nearby cutoff improves validation profit materially and still stays below break-even risk.
   - If A changes, regenerate B.
3. Test B risk-bucket timing curve by predicted PD bucket or prior_underwriter_score bucket.
   - Preserve day-90 spike.
   - Force monotonicity.
   - Keep only if stable and validator-safe.
4. Keep current C unless diagnostics show large structural/proxy movement.
   - Print intervenable=True vs False diagnostics: count, mean abs movement, mean interval width.
   - Do not force non-intervenable effects to zero unless necessary.
5. Interval sanity:
   - Check rows with interval width < 0.005.
   - Widen only if they are not just clipped extreme PD rows.
6. Update D only if final outputs changed.
7. Rerun validator and show PASS.

Final decision rule: valid + calibrated + profit-aware beats speculative complexity.
```

---

## 9. Final Upload Checklist

Before upload, verify:

### File set

```text
submission_A_decisions.csv
submission_B_trajectory.csv
submission_C_counterfactuals.csv
submission_D_writeup.pdf
```

Only these four should be submitted to the hackathon portal.

### A checks

- [ ] 13,306 rows
- [ ] columns exactly: `applicant_id`, `decision`, `predicted_pd`, `pd_lower_90`, `pd_upper_90`
- [ ] every validation + test applicant included
- [ ] no duplicate applicant IDs
- [ ] no NaNs
- [ ] all PDs in [0,1]
- [ ] `pd_lower_90 <= predicted_pd <= pd_upper_90`
- [ ] final approval rate recorded
- [ ] approved mean PD recorded

### B checks

- [ ] 169 rows
- [ ] columns exactly: `cohort_week`, `loan_age_weeks`, `cumulative_default_rate`, `cdr_lower_90`, `cdr_upper_90`
- [ ] all 13 × 13 cohort-age cells present
- [ ] no duplicates
- [ ] no NaNs
- [ ] all values in [0,1]
- [ ] `cdr_lower_90 <= cumulative_default_rate <= cdr_upper_90`
- [ ] CDR non-decreasing by loan age within every cohort
- [ ] week-13 values equal final cohort expected default rate logic

### C checks

- [ ] 900 rows
- [ ] columns exactly: `query_id`, `predicted_pd_cf`, `pd_cf_lower_90`, `pd_cf_upper_90`
- [ ] every `query_id` present
- [ ] no duplicate query IDs
- [ ] no NaNs
- [ ] all values in [0,1]
- [ ] `pd_cf_lower_90 <= predicted_pd_cf <= pd_cf_upper_90`
- [ ] C group diagnostics saved

### D checks

- [ ] PDF renders cleanly
- [ ] max 4 pages body content
- [ ] minimum 11pt font and 0.75 inch margins
- [ ] exact required headers in order
- [ ] no executive summary / marketing fluff
- [ ] actual diagnostics match final CSVs
- [ ] if A/B/C changed, D updated accordingly

### Validator

Run:

```bash
python validate_submission.py final_submission/ --expected-ids expected_ids/
```

Must show:

```text
RESULT: PASS
```

---

## 10. Final Team Message

> Our moat is not one fancy model. Our moat is disciplined risk modeling: calibrated PDs, economic approval decisions, selective-label awareness, monotone timing forecasts, causal humility in counterfactuals, and validator-safe execution. We are not trying to impress with complexity; we are trying to submit the most reliable profitable lender.

