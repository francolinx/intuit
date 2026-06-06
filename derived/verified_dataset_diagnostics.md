# Verified Dataset Diagnostics (derived from full train.csv; full train omitted to stay under 30 MB)

Full train.csv is 36 MB, so it is intentionally not included in this Claude Code pack.
Use this file plus `derived/train_default_events_minimal.csv` for final low-risk tuning.

## Shapes / label structure
- train.csv: 85,340 rows × 44 columns.
- prior approved in train: 51,722.
- prior declined in train: 33,618.
- default_flag non-default: 42,698.
- default_flag default: 9,024.
- default_flag missing: 33,618.
- Labels exist iff prior_decision == 1.
- Train labeled default rate: 0.1745.

## LGD / economics
- Defaulters: 9,024.
- Mean LGD = 1 - recovered/requested_amount: 0.9086.
- Median LGD: 0.9285.
- Performing revenue per dollar r_perf = 0.03 + 0.35 * 60/365 ≈ 0.0875.
- Break-even PD using mean LGD ≈ 0.0878.

## Timing
- days_to_default count: 9,024.
- mean: 43.08.
- median: 37.00.
- min: 3.
- max: 90.
- Use `derived/train_timing_curves_by_group.csv` for overall and risk-bucket default timing curves.
- Use `derived/train_default_events_minimal.csv` if you need to recompute custom timing curves without full train.csv.
