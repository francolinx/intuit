# PHASE 3 — B Risk-Bucket Timing Check

## Fallback B (reconstructed method)
The fallback B = `cohort_mean_approved_PD × overall_timing_fraction(age)`, with cumulative-max
monotonicity and week-13 anchored to the cohort mean approved PD. Verified:
- cohort 1, week 1 = 0.0665 × 0.0721 (overall fraction) = 0.004797 → matches fallback exactly.
- All 13 cohorts monotone; week-13 CDR ≈ cohort mean approved PD (0.065–0.068).
- Day-90 spike preserved: age13−age12 ≈ 0.015 across cohorts.

## Candidate B (risk-bucket blended timing)
Built per the brief: approved applicants from final A, assigned to cohort_week by
`application_timestamp`; cohort final risk = mean predicted_pd; timing = blend of the three
`prior_underwriter_score` bucket curves (low<0.5 / mid / high≥0.85) weighted by each cohort's
approved-loan score mix; cumulative-max monotonicity; week-13 anchored to cohort mean PD.

- Cohort coverage of approved book: 100% (all 2,075 approved mapped to a cohort week).
- Approved score mix: ~22.6% low, ~18.6% mid, ~58.8% high.

## Candidate vs Fallback — difference is negligible
| metric | value |
|---|---|
| Max \|candidate − fallback\| over all 169 cells | **0.0013** |
| Mean cell abs difference | **0.0006** |
| Candidate day-90 spike (mean) | 0.0157 (vs fallback 0.0150) |

The three score-bucket timing curves are **near-identical in shape** (all reach ~0.77–0.81 by
week 9, plateau, then jump to 1.0 at week 13), so blending by risk mix reproduces the overall
curve to within ~0.1 pp per cell.

## Decision: KEEP FALLBACK B
The risk-bucket blend changes no cell by more than 0.0013 — not a material or more-defensible
improvement. It would add bucket-assignment complexity and a small chance of cohort artifacts for
no real gain. The fallback is already monotone, anchored to the approved-book mean PD, and
preserves the day-90 spike. Per brief ("If uncertain, keep fallback B"): **B is unchanged.**
