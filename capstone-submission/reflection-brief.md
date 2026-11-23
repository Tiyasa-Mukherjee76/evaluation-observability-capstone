# Reflection Brief — Evaluation and Observability Capstone

**Name:** Tiyasa Mukherjee
**Date:** 24 Nov 2026

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Linux (see environment.txt) |
| Python version | Python 3.13.0 |
| Date run | 24 Nov 2026 |
| Ran any system live? (which) | No. All runs used offline or replay modes. |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 45 passed, 3 skipped |
| Routing output file | 01-policy-pipeline/pipeline-run.txt |
| auto_approve / human_review / spot_check counts | Demonstrated through routing tests |

**1a. Retry boundary.**

The retry and escalation behavior is verified in the policy pipeline test suite. Missing-source and invalid-input conditions are escalated rather than repeatedly retried because additional model calls cannot recover missing evidence. Retrying a futile case would waste API calls without improving the result.

**1b. Reading the router.**

In `01-policy-pipeline/pipeline-run.txt`, the test `test_ac_04_02_low_confidence_routes_to_human_review` shows that low confidence caused the record to be routed for human review. If confidence alone had been trusted, the system could have automatically approved an uncertain extraction.

**1c. Where the aggregate lies.**

In `01-policy-pipeline/calibration-report.txt`, the `umbrella / exclusions` cell reported `conf=0.93` and `acc=0.00`, while the report overall showed `OVERALL brier=0.291`. The aggregate metric appears reasonable, but the slice reveals a severe weakness for a specific policy type and field combination.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 25 passed |
| Document run | appraisal_informal_sqft.txt |
| Classified type | Property appraisal |

**2a. Two guarantees.**

In `02-mortgage-extraction/discrepancy-run.txt`, the validator reported `consistent=false` with `calculated=9642.17`, `stated=10892.17`, and `delta=-1250.0`. Tool use guarantees valid structured output, but it does not guarantee arithmetic correctness. Schema validation catches formatting errors, while the consistency validator catches incorrect calculations.

**2b. Refusing to fabricate.**

In `02-mortgage-extraction/extract-run.txt`, the field `bonus_monthly` was returned as `null`. Returning null is safer than inventing a value because the source document does not contain the missing information. The schema explicitly allows nullable fields.

**2c. Normalization.**

The source expressed the area as “about 2,400 sq ft,” while the extraction returned `"gross_living_area_sqft": 2400`. Normalizing during extraction creates a consistent format that downstream validation and analysis can use directly.

---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 34 passed |
| Briefing file | 03-supply-chain/investigation-run.txt |
| Section the conflict landed in | Contested |

**3a. Annotate, don't arbitrate.**

In `03-supply-chain/investigation-run.txt`, `on_time_delivery_rate` appeared as `95.0 percent — supplier_audit (2026-04-10)` and `78.0 percent — logistics (2026-04-05)`. Preserving both measurements lets readers evaluate source disagreement themselves instead of hiding uncertainty behind one number.

**3b. Source goes dark.**

In `03-supply-chain/timeout-run.txt`, the briefing reported `Sources unavailable: logistics unavailable (timeout)` and `missing source: timeout reading logistics`. Unreachable sources are marked as unavailable rather than treated as if they never existed. The run still finishes because the system continues using the remaining sources.

**3c. Dates as a guardrail.**

The delivery-rate claims were reported on different dates: `95.0 percent (2026-04-10)` and `78.0 percent (2026-04-05)`. Requiring dates prevents changes over time from being interpreted as contradictions.

---

## 4. Synthesis

**4a. One principle.**

The clearest example occurred in System 2. The extraction succeeded, but `02-mortgage-extraction/discrepancy-run.txt` still detected `delta=-1250.0`. A design that trusted model output without validation would have accepted the incorrect income total.

**4b. Confidence ≠ correctness.**

System 1 demonstrated this most clearly. The calibration report showed `conf=0.93` but `acc=0.00` for `umbrella / exclusions`, demonstrating that confidence alone is not a reliable measure of correctness.

**4c. Apply it.**

For a workflow such as insurance claims processing or mortgage underwriting, I would use validated retry with escalation. I would monitor validation-failure rates, routing-to-human-review rates, calibration slices, and discrepancy counts so that reliability problems become visible before they affect business decisions.
