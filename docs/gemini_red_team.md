This is an exceptionally well-structured pipeline Claude has built. But standard LightGBM + naive break-even + partial counterfactuals is exactly what the top 10 standard teams will do. To win, we need to exploit the scoring mechanics, rigorously define causal rules, and weaponize uncertainty.

Here is the strategic red-team breakdown to push your submission from "strong" to "hard to beat."

---

### PART 1 — NotebookLM-Grounded Rule Check

If Claude violates these, the submission is dead. Based on the NotebookLM context:

1. **The Scoring Formula Constraint:** $S = 0.30 S_{P\&L} + 0.25 S_{traj} + 0.20 S_{cal} + 0.10 S_{C} + 0.15 S_{write}$.
* *Strategic Takeaway:* **Do not over-index on Deliverable C.** It is only 10% of the score. P&L (30%) and Calibration (20%) are half your grade. Ensure your interval generation isn't an afterthought.


2. **Deliverable B Schema:** Must contain exactly 169 rows (13 cohort weeks × 13 loan age weeks). Columns: `cohort_week`, `loan_age_weeks`, `cumulative_default_rate`, `cdr_lower_90`, `cdr_upper_90`.
3. **Data Dictionary Causal Constraint:** The `data_dictionary.csv` contains an explicit `intervenable` boolean column. *You must respect this flag.* If it is `FALSE` (e.g., `sector`, `business_id`), it is a structural/identity proxy, not a causal lever.
4. **Default Definition:** Default is triggered by 3 consecutive missed draws, 6 cumulative missed draws, *or a positive outstanding balance at day 90*. This mathematically forces the Day 90 spike.
5. **Writeup Template:** You must exactly follow the 5-tab structure in `submission_D_writeup_template.md` (Problem framing, Methodology, Causal reasoning, Calibration, Limitations).

---

### PART 2 — Top 10 Moat Improvements (Ranked by Impact)

| Rank | Improvement | Score Impact | Difficulty | Risk | Action |
| --- | --- | --- | --- | --- | --- |
| **1** | **Profit-Maximized Thresholding via Uncertainty** (Approve if 90% Upper Bound PD < Break-even, not just Mean PD). | High ($S_{P\&L}, S_{cal}$) | Low | Low | **Implement Now** |
| **2** | **Reject Inference Pipeline** (Pseudo-label the 33k previously declined rows to fix the sharp selective-label bias). | High ($S_{P\&L}$) | Med | Med | **Test as Ablation** |
| **3** | **Conformal Prediction for Intervals** (Bin val errors by PD deciles to generate dynamic, statistically guaranteed 90% bounds). | High ($S_{cal}$) | Low | Low | **Implement Now** |
| **4** | **Risk-Stratified Timing Curves (B)** (High-risk loans default earlier; low-risk default later. Do not use one global timing curve). | Med ($S_{traj}$) | Low | Low | **Implement Now** |
| **5** | **Zero-Causal-Impact for `intervenable=FALSE**` (For Deliverable C, force $\Delta PD = 0$ for non-intervenable features, overriding the ML model). | Med ($S_C$) | Low | Low | **Implement Now** |
| **6** | **Distance-Based Interval Widening (OOD)** (Widen $S_{cal}$ intervals for applicants far from the 'prior approved' training distribution). | Med ($S_{cal}$) | Med | Low | **Implement Now** |
| **7** | **LGD Prediction/Stratification** (Don't use a global 0.907 LGD. Predict it or bin it by requested amount/revenue to individualize the break-even). | Med ($S_{P\&L}$) | High | High | **Skip (Too little time)** |
| **8** | **Day-90 Sweep Sub-Model** (Model week 1-12 defaults as a survival process, and week 13 as a binary 'bullet payment failure' process). | Low ($S_{traj}$) | High | High | **Skip** |
| **9** | **Shrinkage on Weak Causal Levers** (For `intervenable=TRUE` but highly confounded features, shrink the counterfactual $\Delta$ by 50% towards baseline). | Low ($S_C$) | Med | Low | **Test as Ablation** |
| **10** | **Structural Recalculation Cascades** (If `requested_amount` changes in C, recompute *all* dependent ratios automatically before scoring). | Low ($S_C$) | Low | Low | **Implement Now** |

---

### PART 3 — A / Lending Policy Red Team

* **Current 15.6% Approval Rate:** If break-even is 8.8% and your approved pool mean is 6.7%, 15.6% approval is likely **too conservative**. You are optimizing for an arbitrarily low portfolio default rate rather than *total portfolio profit*. A profitable loan at 8.0% PD is still profitable. Standard teams will approve more and win on volume.
* **The Threshold Strategy:** Pure break-even is dangerous due to calibration drift. Do not use pure mean PD. **Use Risk-Buffered Thresholding:** Approve if the `predicted_pd` + (some fraction of the upper 90% confidence bound distance) < Break-Even PD. This naturally penalizes highly uncertain applicants (like the previously declined ones).
* **Optimizing for $S_{P\&L}$:** The metric is realized portfolio value. If average LGD is ~0.90, the math is: `Expected Profit = Expected Revenue - (PD * LGD * Exposure)`. Calculate this exact expected dollar value for every applicant, and approve if $E[\text{Profit}] > \$0$, using the upper-bound risk estimate as a safety margin.
* **Beating standard teams:** Standard teams will pick a threshold that maximizes F1 or just cut at the portfolio mean. You beat them by underwriting explicitly to dollar-value expected profit.

---

### PART 4 — B / Default Trajectory Red Team

* **What could go wrong:** Claude's "cohort-specific approved-loan average risk × empirical default-timing curve" assumes all loans default on the same timeline. **They don't.** Sub-prime loans default in weeks 2-6 (missed draws). Near-prime loans default in week 13 (failed to clear the balloon balance on day 90).
* **The Best Method:** Stratify the empirical timing curve. Create two curves from the validation set: `timing_low_risk` and `timing_high_risk`. For each cohort, calculate the mix of low/high risk loans approved, and blend the curves.
* **Week 13 / Day-90 Spike:** Force the week 13 cumulative value to exactly equal the final expected cohort default rate. The difference between week 12 and week 13 is simply the remainder.
* **Intervals:** Use the Agresti-Coull binomial confidence interval based on the number of approved loans in that cohort. *Crucial:* If a cohort only has 128 approvals, the statistical noise is massive. Widen the intervals inversely proportional to $\sqrt{N_{approved}}$.
* **Fast Improvement:** Enforce `cummax()` strictly across the weeks. The validator will likely instantly fail any submission where week 8 CDR < week 7 CDR.

---

### PART 5 — C / Counterfactual Moat

If you just run `model.predict(altered_row)`, you fail the causal test. The model learned associations, not do-calculus.

**1. Feature Group Buckets (Based on Data Dictionary):**

* **Bucket A: Pure Causal (`intervenable=TRUE` + Direct Action):** e.g., `requested_amount`.
* **Bucket B: Self-Reported/Fuzzy (`intervenable=TRUE` + Confounded):** e.g., `stated_annual_revenue`.
* **Bucket C: Identity/Proxy (`intervenable=FALSE`):** e.g., `sector`, `vintage_years`, `prior_loans_count`, `has_linked_bank_feed`.

**2. Causal Movement Rules:**

* **Bucket A:** Allow full model-predicted movement. Recompute `requested_amount_to_observed_revenue` and any other deterministic math features.
* **Bucket B:** Shrink movement. If model says changing revenue drops PD by 4%, shrink it to 1% (multiply delta by 0.25). Why? Changing what is *written on the form* doesn't instantly change the reality of the business's cash flow in the same way actual historical data implies.
* **Bucket C:** **Force $\Delta PD = 0$.** Return the exact same PD as the baseline.

**3. Uncertainty Rules:**

* For any intervention in Bucket B or C, dynamically widen the 90% interval by 2x. You are expressing "causal humility" directly into the $S_{cal}$ metric.
* **Regulator-Defensible Explanation (For Tab 3):** *"We treat `intervenable=FALSE` features as structural confounders. In a structural causal model (SCM), intervening on a proxy (e.g., sector) does not alter the unobserved risk traits that proxy represents. Therefore, our interventional expectation remains anchored to the baseline, reflecting reality over naive model-curve fitting."*

---

### PART 6 — Calibration & Interval Moat

To crush the $S_{cal}$ (20%) requirement:

1. **Adaptive by PD Bin:** Sort validation predictions into deciles. Calculate the actual empirical default rate in each decile. Use the standard deviation of errors in that specific bin to size the intervals. High PD bins should have much wider intervals than low PD bins.
2. **Adaptive by OOD (The Secret Weapon):** The 33k previously declined rows are Out-Of-Distribution (OOD). For every test row, calculate its distance to the centroid of the `prior_approved` training set. Multiply the interval width by $(1 + \alpha \times \text{Distance})$. This guarantees your intervals cover the surprises in the previously-declined population.
3. **Avoid:**
* Avoid symmetric intervals (PD is bounded at 0). If mean is 0.02, the lower bound cannot be -0.01. Clamp at 0, or use logit-space standard deviations.
* Avoid standard LightGBM `objective='quantile'` unless you have thoroughly tuned it. Empirical binning is much safer for hackathons.



---

### PART 7 — Required Ablations Before Final

Run these to finalize the strategy. Stop if time is short, but these are the priority:

1. **Profit-aware approval vs PD threshold (Highest Priority):** Compare total expected validation profit of Claude's 15.6% threshold vs an $E[\text{Profit}] > \$0$ threshold. *If profit-aware yields >5% more expected $ without blowing up the default rate, switch entirely.*
2. **C naive vs causal-group-shrunk (Deliverable C):** Run the 900 queries naive vs. with our Bucket A/B/C rules. *If naive produces wildly different PDs for non-intervenable features (e.g., changing sector drops PD by 10%), use the causal-shrunk method to avoid the $S_C$ penalty.*
3. **prior_underwriter_score on/off:** Train with and without it. Evaluate on the *previously declined* pseudo-labels. *If including it causes predicted PDs for declined applicants to collapse to unrealistic lows, drop it—it’s leaking the selection bias.*
4. **B empirical timing vs cohort-adjusted timing:** *If week 1-6 errors are massive across cohorts, switch to the risk-stratified timing curves.*

---

### PART 8 — Writeup Language Moat (Deliverable D)

Copy-paste/adapt these to sound undeniably professional and technically rigorous:

* **Problem Framing (Selective Labels):** *"We identified a severe reject-inference problem: labels exist strictly conditionally on $prior\_decision = 1$. Evaluating natively on this distribution violates the independent and identically distributed (i.i.d) assumption required for deployment. Consequently, our validation metrics were explicitly recalibrated to account for the truncation of the loss distribution..."*
* **Methodology (Profit-Aware):** *"Rather than arbitrarily optimizing ROC-AUC or applying a static PD threshold, we framed underwriting as a constrained optimization of $S_{P\&L}$. We approve an applicant if and only if the lower-bound expected profit is strictly positive: $E[Revenue] - (PD_{upper\_90} \times LGD \times Exposure) > 0$. This inherently penalizes epistemic uncertainty."*
* **Causal Reasoning (Deliverable C):** *"We reject naive counterfactual generation. We partitioned the feature space into direct levers (`intervenable=TRUE`) and associative proxies (`intervenable=FALSE`). Under our assumed Structural Causal Model, intervening on a proxy node (e.g., sector) without shifting its ancestral confounders yields an expected causal effect of zero. We forced structural recalculation of dependent ratios while heavily regularizing proxy perturbations toward the baseline."*

---

### PART 9 — Red-Team Attack (How we lose and how to patch it)

**Judge/Competitor Attack 1: "Your model approved a bunch of previously declined loans and they defaulted at a 40% rate in Test, tanking your $S_{P\&L}$."**

* *Why it happens:* The model is overconfident on data it hasn't seen (declined loans).
* *The Patch:* The **OOD Distance Penalty** combined with **Risk-Buffered Thresholding**. By widening the intervals for OOD applicants and requiring the *upper bound* to be profitable, we mathematically lock out overconfident, risky bets.

**Judge/Competitor Attack 2: "Your trajectories (B) overshoot the actual defaults because you double-counted early defaults and the day-90 sweep."**

* *Why it happens:* If you apply a standard cumulative distribution, you smooth out the day-90 spike.
* *The Patch:* Decouple the timeline. Compute the probability of "Default before Day 90" and "Default at Maturity". Map them strictly to Week 1-12 and Week 13.

---

### PART 10 — Final Upload Checklist

Claude MUST verify these before calling the submission complete:

* [ ] **Deliverable A:** Contains exactly 13,306 rows (validation + test applicants).
* [ ] **Deliverable B:** Contains exactly 169 rows (13 cohort weeks × 13 loan ages).
* [ ] **Deliverable C:** Output schema exactly matches the 900 `query_id`s in `intervention_queries.csv`.
* [ ] **Monotonicity Check (B):** `cumulative_default_rate` strictly non-decreasing per cohort.
* [ ] **Interval Sanity Check:** `upper_90 >= predicted_pd >= lower_90` for EVERY single row in A, B, and C.
* [ ] **Boundary Check:** No PD < 0.0001. No PD > 0.9999.
* [ ] **NaN Check:** 0 missing values in any submission CSV.
* [ ] **Formatting:** PDF writeup is cleanly rendered from the markdown template, exactly 5 sections.
* [ ] **Validator:** Run `python validate_submission.py` (assumed based on requirements snippet) locally. **Must output PASS.**