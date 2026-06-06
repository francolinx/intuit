# Internal Rubric Scorecard — Final Candidate (= Fallback)

The final candidate is byte-identical to the fallback in all four files (no tested improvement
beat it). Scores below are the assessed quality of that candidate.

| # | Criterion | Max | Score | Replace fallback? |
|---|---|---:|---:|---|
| 1 | Validator safety | 10 | **10** | n/a (identical) |
| 2 | A — PD + profit policy | 30 | **27** | No |
| 3 | B — trajectory | 15 | **14** | No |
| 4 | C — counterfactuals | 20 | **18** | No |
| 5 | Calibration / uncertainty | 15 | **14** | No |
| 6 | D — writeup | 10 | **9** | No |
| | **Total** | **100** | **92** | |

## 1. Validator safety — 10/10
Reasoning: `validate_submission.py final_submission/` → **PASS (0 errors, 0 warnings)**. Correct
row counts (A=13,306; B=169; C=900), all IDs match the expected sets, all intervals ordered, all
values in [0,1], B monotone per cohort, exactly the 4 required files.
vs fallback: identical. No change warranted.

## 2. A — PD + profit policy — 27/30
Reasoning: Calibrated PD (AUC 0.7484, Brier 0.1354, mean predicted 0.2061 vs actual 0.2062 — near
perfect aggregate calibration). Approval rate 15.6%, approved-book mean PD 6.66%, cutoff at the
**theoretical break-even PD (0.0880)** — a principled, profit-aware policy, not a naive low-PD cut.
Validation: 599 approved labeled rows, 7.0% bad rate, positive rough profit (10.66 per-loan units;
~$218k dollar-weighted). Lost points: threshold scan shows the validation-optimal cutoff is noisy;
a slightly different sample could shift it — but chasing it overfits (see Phase 2).
vs fallback: a 0.0900 cutoff showed +1.89 validation profit but is driven by 33 above-break-even
loans with 1 lucky default; not robust. **Keep fallback.**

## 3. B — trajectory — 14/15
Reasoning: Monotone in all 13 cohorts, week-13 anchored to cohort mean approved PD (~0.066),
day-90 spike preserved (~0.015 jump at age 13, matching the train curve's day-90 balloon-payment
default mechanism). Built as cohort-mean-PD × empirical timing curve — defensible and simple.
Lost point: a single shared timing shape across cohorts (acceptable given near-identical group curves).
vs fallback: risk-bucket blend differs by ≤0.0013/cell — no material gain. **Keep fallback.**

## 4. C — counterfactuals — 18/20
Reasoning: All 900 queries, IDs exact, intervals valid. do(feature=value) framing with dependency
recomputation; intervenable interventions move PD (mean |Δ| 0.034) while non-intervenable barely
move (mean |Δ| 0.007); intervals widen for weak/proxy/non-intervenable groups (0.326 vs 0.246).
Lost points: counterfactual ground truth is unverifiable (inherent to the task); a couple of
non-intervenable features move modestly (≤0.065).
vs fallback: no safer methodology identified. **Keep fallback.**

## 5. Calibration / uncertainty — 14/15
Reasoning: Aggregate PD calibration is excellent (0.2061 vs 0.2062); 90% interval coverage ~93.3%
(mildly conservative, acceptable). Interval widths sensible: tight only at PD≈1 edges, none
suspiciously tight in the normal range. C intervals appropriately wider than A.
vs fallback: identical. No widening necessary.

## 6. D — writeup — 9/10
Reasoning: Valid PDF, 3 pages (≤4 limit), strong coherent technical narrative covering selective
labels, profit-aware decisions, separated timing model, interventional counterfactuals, and
uncertainty. Remains fully consistent because A/B/C are unchanged.
vs fallback: identical and still coherent. **Keep fallback.**

## Overall verdict
No tested improvement clearly beats the fallback. **Ship the fallback.** Final score 92/100.
