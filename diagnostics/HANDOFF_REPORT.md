# HANDOFF REPORT — Intuit SMB Underwriting Hackathon Submission

## TL;DR
- **Status:** Complete. Final submission validated and pushed.
- **Decision:** Ship the **fallback unchanged** — no tested improvement beat it.
- **`final_submission/` is byte-identical to `fallback_submission/`** (verified with `cmp`).
- **Validator:** `PASS (0 errors, 0 warnings)`.
- **Branch pushed:** `claude/festive-maxwell-6Vsz4` (commit `3e7842f`).
- **Internal rubric:** 92/100.

---

## 1. Environment / setup
- Repo `/home/user/intuit` started empty; extracted the 30 MB pack into it.
- Installed `requirements.txt` (numpy 2.4.6, pandas 3.0.3).
- Created `final_submission/` and `diagnostics/`; copied fallback -> final; validated immediately (PASS).
- `fallback_submission/` was never modified or overwritten.

## 2. What the submission is (final = fallback)
| Deliverable | Content | Key facts |
|---|---|---|
| A — decisions | 13,306 applicants (val+test) | Approval rate 15.59% (2,075 approved), approved-book mean PD 6.66%, cutoff at break-even PD 0.0880. 0 dup IDs, 0 missing. |
| B — trajectory | 169 rows (13x13 grid) | 0 non-monotone cohorts; week-13 anchored to cohort mean approved PD (~0.066); day-90 spike preserved (~0.015). |
| C — counterfactuals | 900 queries | do(feature=value) with dependency recomputation; intervals valid; non-intervenable movements small, weak-group intervals wider. |
| D — writeup | PDF, 3 pages (<=4) | Coherent; unchanged because A/B/C unchanged. |

Calibration (validation, 2,551 labeled rows): AUC 0.7484, Brier 0.1354, mean predicted PD 0.2061 vs actual 0.2062, 90% interval coverage ~93.3%.

## 3. Improvement checks run (and why each was rejected)

### Phase 2 — A threshold sensitivity (the critical call)
Scanned cutoffs 0.0850-0.0950. Apparent "best" cutoff 0.0900 showed +1.89 validation profit (12.56 vs fallback 10.66).
- Rejected as noise. The entire gain comes from 33 marginal loans (predicted_pd ~= 8.95%, above the 0.0880 break-even) that happened to have only 1 realized default (3.0% vs model-implied 8.95%).
- Model-implied EV of those loans is negative. A bootstrap confirms fragility — if 2-3 of the 33 had defaulted (expected at 8.95%), the gain vanishes.
- The profit-vs-cutoff curve is jagged/non-monotone — signature of small-sample noise, not a stable optimum.
- Fallback already sits at the principled break-even PD. A unchanged.

### Phase 3 — B risk-bucket timing blend
Built a candidate B blending the three prior_underwriter_score bucket timing curves by each cohort's approved-loan risk mix.
- The bucket curves are near-identical in shape -> candidate differs from fallback by <=0.0013 per cell (mean 0.0006). No material gain; adds complexity. B unchanged.

### Phase 4 — C sanity
- Query count 900 OK, IDs match expected_ids/query_ids.txt exactly OK, no dups OK, values in [0,1] OK, intervals ordered OK, all query applicants in val+test OK.
- Intervenable interventions move PD (mean |delta| 0.034); non-intervenable barely move (mean |delta| 0.007, only 2/174 > 0.05).
- Interval widths wider for weak/proxy/non-intervenable groups (0.326 vs 0.246) — correct uncertainty framing. C unchanged.

### Phase 5 — interval sanity
- A: only 61/13,306 rows have width <0.005, all at PD = 1.0 (legitimate edge). 0 suspiciously tight in normal range.
- C: only 3/900 tight, all at PD = 1.0. No widening needed.

### Phase 6 — writeup
- A/B/C unchanged -> D stays. PDF is valid, 3 pages. D unchanged.

## 4. Internal rubric — 92/100
| Criterion | Max | Score |
|---|---:|---:|
| Validator safety | 10 | 10 |
| A — PD + profit policy | 30 | 27 |
| B — trajectory | 15 | 14 |
| C — counterfactuals | 20 | 18 |
| Calibration / uncertainty | 15 | 14 |
| D — writeup | 10 | 9 |

## 5. Remaining risks
- Small validation labeled set (2,551 rows; 599 approved) -> noisy profit/threshold estimates (the reason we did not chase the apparent threshold gain).
- Counterfactual (C) correctness is unverifiable by construction; mitigated by wider weak-feature intervals.
- Selective-label bias: outcomes only for prior-approved matured loans; PD generalization to declined population assumed (disclosed in D).
- Test set (8,817 rows) unlabeled -> test-side economics rely on calibration transfer.

## 6. Validator output
```
python validate_submission.py final_submission/
Expecting 13,306 applicants, 900 queries, 13x13 trajectory grid.
No issues found.
RESULT: PASS  (0 errors, 0 warning(s))
```

## 7. Files to upload (exactly four, from final_submission/)
1. submission_A_decisions.csv
2. submission_B_trajectory.csv
3. submission_C_counterfactuals.csv
4. submission_D_writeup.pdf

## 8. Artifacts added (non-submission, in diagnostics/)
run_diagnostics.py (reproducible), fallback_diagnostics.md, phase2_threshold_sensitivity.md,
phase3_B_check.md, phase4_C_check.md, phase5_interval_sanity.md, rubric_scorecard.md,
final_diagnostics_summary.md, change_log.md.

## 9. Git
- Branch claude/festive-maxwell-6Vsz4, commit 3e7842f, pushed to origin. No PR created.

Bottom line: A valid, strong, validator-passing fallback was protected and shipped. Every tested
"upgrade" was either statistical noise (A), negligible (B), or unnecessary (C/D). Nothing risky was
introduced.
