# PHASE 2 — A Threshold Sensitivity

_Profit per dollar = (1-y)*0.0875 - y*0.9065. Break-even PD = 0.0880_

Scan applies a PD cutoff (approve if predicted_pd <= cutoff) and evaluates on validation labeled rows. 'total approvals' counts val+test rows with predicted_pd<=cutoff.

**Current fallback A**: val approved labeled=599, bad rate=7.0117%, total profit=10.6645, per-loan=0.017804, total val+test approvals=2075

| cutoff | val_appr | val_appr_rate | val_bad_rate | val_total_profit | profit_per_appr | total_vt_appr |
|---|---|---|---|---|---|---|
| 0.0850 | 566 | 0.2219 | 0.0689 | 10.7590 | 0.019009 | 1917 |
| 0.0855 | 583 | 0.2285 | 0.0669 | 12.2465 | 0.021006 | 1995 |
| 0.0860 | 583 | 0.2285 | 0.0669 | 12.2465 | 0.021006 | 1995 |
| 0.0865 | 583 | 0.2285 | 0.0669 | 12.2465 | 0.021006 | 1995 |
| 0.0870 | 583 | 0.2285 | 0.0669 | 12.2465 | 0.021006 | 1995 |
| 0.0875 | 594 | 0.2328 | 0.0690 | 11.2210 | 0.018891 | 2049 |
| 0.0880 | 598 | 0.2344 | 0.0702 | 10.5770 | 0.017687 | 2073 |
| 0.0885 | 599 | 0.2348 | 0.0701 | 10.6645 | 0.017804 | 2075 |
| 0.0890 | 599 | 0.2348 | 0.0701 | 10.6645 | 0.017804 | 2075 |
| 0.0895 | 610 | 0.2391 | 0.0705 | 10.6330 | 0.017431 | 2116 |
| 0.0900 | 632 | 0.2477 | 0.0680 | 12.5580 | 0.019870 | 2206 |
| 0.0905 | 632 | 0.2477 | 0.0680 | 12.5580 | 0.019870 | 2206 |
| 0.0910 | 632 | 0.2477 | 0.0680 | 12.5580 | 0.019870 | 2206 |
| 0.0915 | 637 | 0.2497 | 0.0706 | 11.0075 | 0.017280 | 2214 |
| 0.0920 | 661 | 0.2591 | 0.0696 | 12.1135 | 0.018326 | 2333 |
| 0.0925 | 661 | 0.2591 | 0.0696 | 12.1135 | 0.018326 | 2333 |
| 0.0930 | 661 | 0.2591 | 0.0696 | 12.1135 | 0.018326 | 2333 |
| 0.0935 | 661 | 0.2591 | 0.0696 | 12.1135 | 0.018326 | 2336 |
| 0.0940 | 779 | 0.3054 | 0.0783 | 7.5285 | 0.009664 | 2818 |
| 0.0945 | 779 | 0.3054 | 0.0783 | 7.5285 | 0.009664 | 2818 |
| 0.0950 | 779 | 0.3054 | 0.0783 | 7.5285 | 0.009664 | 2818 |

**Best cutoff by val total profit:** 0.0900 (profit=12.5580, approvals=632, bad=0.0680, val+test appr=2206)

**Fallback total profit:** 10.6645

**Delta vs fallback:** +1.8935 (MATERIAL)


## Decision: KEEP FALLBACK A

The apparent best cutoff (0.0900, +1.89 validation profit) is **small-sample noise**, not signal:

- The entire gain comes from **33 marginal loans** (predicted_pd in (0.0885, 0.0900]) that
  had only **1 realized default** (realized bad rate 3.0% vs model-implied 8.95%).
- Those marginal loans sit **above the break-even PD (0.0880)**, so the calibrated model
  prices them as **negative expected value** (model-implied EV = -0.049 over the 33 loans).
- A well-calibrated model (AUC 0.748, Brier 0.135, mean PD 0.2061 vs actual 0.2062) saying
  "8.95% PD" against a realized 1/33 is the *sample* being lucky, not the model being wrong.
- If 2-3 of those 33 had defaulted (expected at 8.95%), the +1.89 evaporates to ~0 or negative.
- The profit-vs-cutoff curve is jagged/non-monotone (0.0875->11.2, 0.0880->10.6, 0.0900->12.6,
  0.0940->7.5) — the signature of noise, not a stable optimum.
- The fallback threshold (~0.0885-0.0890) sits **exactly at the theoretical break-even PD
  (0.0880)** — the principled, defensible choice. Funding model-negative-EV loans on the large
  unlabeled test population to chase one lucky validation draw is exactly the overfit the brief
  warns against.

Per brief rule 8 and "do not switch for tiny/noisy gains": **A is unchanged.**
