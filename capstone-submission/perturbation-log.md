# Perturbation Log

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):**
  Used the routing test suite scenario that simulates low-confidence and reviewer-disagreement routing conditions.
- **Command I ran:**
  `pytest tests/test_us04_routing.py -v`
- **What I predicted:**
  Records with low confidence or reviewer disagreement would be routed to human review rather than auto-approved.
- **What actually happened (paste the key output line):**
  `test_ac_04_02_low_confidence_routes_to_human_review PASSED`

  `test_ac_04_06_high_confidence_plus_reviewer_disagreement_still_routes_to_human_review PASSED`
- **How this differs from the unperturbed run:**
  In the standard routing path, records can be auto-approved. Under the perturbed conditions, routing escalated to human review because independent review signals overrode automatic approval.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):**
  Compared `income_sum_mismatch.txt` against valid extraction inputs.
- **Command I ran:**
  `mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay`
- **What I predicted:**
  The extraction would succeed but the consistency validator would detect an arithmetic mismatch.
- **What actually happened (paste the key output line):**
  `"consistent": false`

  `"delta": -1250.0`
- **How this differs from the unperturbed run:**
  The normal appraisal extraction returned `"consistent": true`, while the perturbed paystub triggered the discrepancy validator because the stated total did not match the calculated total.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):**
  Simulated a source failure using the timeout option.
- **Command I ran:**
  `supply-chain-investigate meridian --offline --simulate-timeout`
- **What I predicted:**
  The failed source would be marked incomplete, but the investigation would still finish.
- **What actually happened (paste the key output line):**
  `Sources unavailable: logistics unavailable (timeout)`

  `missing source: timeout reading logistics`
- **How this differs from the unperturbed run:**
  The normal run included logistics data and showed a contested delivery-rate metric. The timeout run completed successfully but moved logistics-dependent information into the Incomplete section.
