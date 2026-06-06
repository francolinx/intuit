# PHASE 4 — C Sanity Check

## Structural checks (all PASS)
| check | result |
|---|---|
| query count | 900 ✓ |
| query_ids == expected_ids/query_ids.txt | True ✓ |
| query_ids == intervention_queries.csv | True ✓ |
| duplicate query_ids | 0 ✓ |
| values in [0,1] | True ✓ |
| interval order (lower ≤ point ≤ upper) | True ✓ |
| query applicants all present in val+test | True ✓ |

## Counterfactual movement vs A baseline (matched 900/900)
- Mean absolute movement overall: **0.0285**
- intervenable=True: mean |movement| = **0.0337**, max 0.4925
- intervenable=False: mean |movement| = **0.0070**, max 0.0648
- Non-intervenable queries with |movement| > 0.05: **2 of 174**
  (`days_since_last_inquiry_elsewhere`, `platform_active_months`) — rare and modest.

Interventions on intervenable features move PD more than on non-intervenable ones — the correct
causal behavior. Non-intervenable movements are appropriately near-zero.

## Interval width by group (wider where causal identification is weaker)
| group | n | mean width |
|---|---|---|
| intervenable=False | 174 | **0.326** |
| intervenable=True | 726 | **0.246** |

By feature group: business_identity 0.317, application_context 0.308, platform_engagement 0.289,
self_reported 0.283, bank_feed 0.240, bureau_credit 0.220. Weak/proxy/non-intervenable groups
carry the **widest** intervals — the intended uncertainty framing.

- Narrow intervals (<0.02) in normal PD range [0.05, 0.95]: **0** (no suspiciously tight bands).
- The only width<0.005 rows (3) sit at PD = 1.0 (a saturated edge, legitimately tight).

## Decision: KEEP FALLBACK C
Non-intervenable/proxy movements are small and their intervals are wide; intervenable effects are
larger as expected; no narrow intervals in the normal range. No methodological change is safer than
the current structured do(feature=value) approach with dependency recomputation and
group-aware widening. Per brief: **C is unchanged.**
