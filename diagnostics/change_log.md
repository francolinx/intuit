# Change Log

## Summary
`final_submission/` is **byte-identical to `fallback_submission/`**. No deliverable was changed.
Every tested improvement failed to clearly beat the validator-passing fallback, so the fallback
ships unchanged (brief rule 8).

## Actions taken
| Phase | Action | Outcome |
|---|---|---|
| 0 | Installed requirements; created `final_submission/` + `diagnostics/`; copied fallback into final; validated. | PASS |
| 1 | Full fallback diagnostics (A/B/C). | All metrics match the brief; healthy. |
| 2 | A threshold scan 0.0850–0.0950 + bootstrap. | Apparent +1.89 profit at 0.0900 is small-sample noise (33 loans, 1 default, above break-even). **A unchanged.** |
| 3 | Built candidate B with risk-bucket blended timing curves. | Differs ≤0.0013/cell from fallback. **B unchanged.** |
| 4 | C structural + movement + interval-width sanity. | IDs exact, intervals valid, non-intervenable movements small, weak groups wider. **C unchanged.** |
| 5 | A & C interval sanity. | No suspiciously tight intervals in normal PD range. No widening. |
| 6 | Writeup coherence. | A/B/C unchanged → **D unchanged** (valid 3-page PDF). |
| 7 | Internal rubric scorecard. | 92/100; keep fallback on every criterion. |
| 8 | Final validation. | PASS, exactly 4 files. |

## Files changed in submission
None. (A, B, C, D all IDENTICAL to fallback per `cmp`.)

## Artifacts added (non-submission, in `diagnostics/`)
- `run_diagnostics.py` — reproducible diagnostics script (Phases 1, 2, 5)
- `fallback_diagnostics.md` (Phase 1)
- `phase2_threshold_sensitivity.md` (Phase 2 + decision)
- `phase3_B_check.md` (Phase 3 + decision)
- `phase4_C_check.md` (Phase 4 + decision)
- `phase5_interval_sanity.md` (Phase 5)
- `rubric_scorecard.md` (Phase 7)
- `final_diagnostics_summary.md` (Phase 8)
- `change_log.md` (this file)

`fallback_submission/` was never modified or overwritten.
