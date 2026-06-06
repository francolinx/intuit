# Fallback Diagnostics (PHASE 1)

_Economics: r_perf = 0.0875, LGD = 0.9065_

## Deliverable A

- Row count: **13306**
- Duplicate applicant_ids: **0**
- Missing values per column: none
- Approvals: **2075**  |  Approval rate: **15.5945%**
- Mean PD approved: **6.6634%**  |  Mean PD declined: **31.4229%**
- Max approved PD: **8.8296%**  |  Min declined PD: **8.9258%**
- Interval width: min=0.0000, mean=0.1279, median=0.1228, max=1.0000
- Implied approval cutoff (max approved PD): **0.0883**

### Validation (labeled rows) metrics

- Validation labeled rows matched to A: **2551**
- Validation approved (labeled): **599**
- Validation bad rate among approved: **7.0117%**
- Validation bad rate among all labeled: **20.6194%**
- Rough validation profit (sum per-loan units, approved labeled): **10.6645**
- Profit per approved (mean per-loan): **0.017804**
- Rough validation profit (dollar-weighted, approved labeled): **$218,001**
- Validation AUC (all labeled): **0.7484**
- Validation Brier (all labeled): **0.1354**
- Mean predicted PD (labeled): **0.2061** vs actual **0.2062**

- (Interval coverage is computed in the calibration section of the rubric.)

## Deliverable B

- Row count: **169** (expected 169)
- Non-monotone cohorts: **0**
- Final (age=13) CDR by cohort:
    - cohort  1: 0.0665
    - cohort  2: 0.0677
    - cohort  3: 0.0650
    - cohort  4: 0.0664
    - cohort  5: 0.0656
    - cohort  6: 0.0666
    - cohort  7: 0.0666
    - cohort  8: 0.0677
    - cohort  9: 0.0667
    - cohort 10: 0.0662
    - cohort 11: 0.0667
    - cohort 12: 0.0676
    - cohort 13: 0.0675
- Day-90 spike (age13-age12) mean across cohorts: **0.0150** (min 0.0146, max 0.0152)

## Deliverable C

- Row count: **900** (expected 900)
- Mean counterfactual PD: **0.2901**
- Mean interval width: **0.2615** (min 0.0000, max 0.9978)

### C by intervenable flag

- intervenable=False: n=174, mean cf PD=0.3066, mean width=0.3256
- intervenable=True: n=726, mean cf PD=0.2862, mean width=0.2461

### C movement vs A baseline (matched 900/900)

- Mean absolute movement overall: **0.0285**
- intervenable=False: mean |movement|=0.0070, max=0.0648
- intervenable=True: mean |movement|=0.0337, max=0.4925
- Non-intervenable queries with |movement|>0.05: **2** of 174
    - by feature: {'days_since_last_inquiry_elsewhere': 1, 'platform_active_months': 1}
