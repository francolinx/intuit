# Final Team Plan — Short Version

## Recommendation
Submit Claude’s current validator-passing version unless a final threshold/B timing ablation clearly improves results.

## Current grade
**89/100 — A-**

## Best practical final grade after low-risk tuning
**90–91/100**

## Keep
- Claude LightGBM calibrated PD model
- Deployment-era recalibration
- Profit-aware threshold near 8.8% break-even PD
- Current C causal framing and wider intervals for weak/proxy features
- Current D writeup unless outputs change

## Test, but only if fast
1. A threshold scan from 0.085–0.095.
2. B risk-bucket timing curves.
3. C diagnostics by intervenable flag and feature group.
4. Interval sanity for ultra-narrow rows.

## Avoid
- Full generative pseudo-labeling
- Full causal forest
- Full Cox replacement
- Upper-bound-only approval rule
- Zeroing every non-intervenable C effect
- Heavy D rewrite

## Final upload files
- `submission_A_decisions.csv`
- `submission_B_trajectory.csv`
- `submission_C_counterfactuals.csv`
- `submission_D_writeup.pdf`

## Golden rule
**Valid + calibrated + profit-aware beats speculative complexity.**
