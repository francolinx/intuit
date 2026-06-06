# Final Diagnostics Summary

## Decision: SHIP THE FALLBACK (unchanged)
`final_submission/` is byte-identical to `fallback_submission/`. No tested improvement clearly beat
the validator-passing fallback.

## Validator
```
python validate_submission.py final_submission/
Expecting 13,306 applicants, 900 queries, 13x13 trajectory grid.
No issues found.
RESULT: PASS  (0 errors, 0 warning(s))
```
`final_submission/` contains exactly: submission_A_decisions.csv, submission_B_trajectory.csv,
submission_C_counterfactuals.csv, submission_D_writeup.pdf.

## Key metrics (final = fallback)
| Metric | Value |
|---|---|
| A rows | 13,306 (0 dup IDs, 0 missing) |
| Approval rate | 15.59% (2,075 approved) |
| Approved-book mean PD | 6.66% |
| Approval cutoff | ≈0.0885 (at break-even 0.0880) |
| Validation AUC | 0.7484 |
| Validation Brier | 0.1354 |
| Mean predicted PD vs actual | 0.2061 vs 0.2062 |
| Validation approved (labeled) | 599, bad rate 7.0% |
| Validation rough profit | 10.66 per-loan units (~$218k dollar-weighted) |
| B | 169 rows, 0 non-monotone cohorts, day-90 spike preserved (~0.015) |
| C | 900 rows, intervals valid, non-intervenable \|Δ\| 0.007, weak-group intervals wider |

## Why no change
- **A:** validation-optimal cutoff (0.0900) gain is concentrated in 33 above-break-even loans with
  1 lucky default — noise, not signal. Fallback sits at the principled break-even PD.
- **B:** risk-bucket timing blend differs ≤0.0013/cell — no material improvement.
- **C:** structurally clean; non-intervenable movements small, weak-group intervals already wide.
- **D:** coherent and unchanged because A/B/C unchanged.

## Remaining risks
- Validation labeled set is small (2,551 rows; 599 approved), so profit/threshold estimates are
  noisy — this is the reason we did **not** chase the apparent threshold gain.
- Counterfactual (C) correctness is unverifiable by construction (no counterfactual ground truth);
  mitigated by wider intervals on weakly-identified features.
- Selective-label bias: outcomes exist only for prior-approved matured loans; PD generalization to
  the declined population is assumed, not observed (disclosed in writeup D).
- Test set (8,817 rows) is unlabeled, so all test-side economics rely on calibration transfer.

## Files to upload (exactly four, from `final_submission/`)
1. submission_A_decisions.csv
2. submission_B_trajectory.csv
3. submission_C_counterfactuals.csv
4. submission_D_writeup.pdf
