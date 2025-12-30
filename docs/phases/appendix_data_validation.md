# 📎 Appendix A — Weight Justification for the Data Quality Index (DQI)

## A.1 Purpose of This Appendix

This appendix documents the rationale behind the weighting strategy used in the **Subject-Level Data Quality Index (DQI)** and provides **empirical evidence** demonstrating that the weights are **non-arbitrary, severity-aligned, and operationally justified**.

The intent is **not** to claim optimality, but to show that the chosen weights:

- reflect real data behavior,
- prioritize clinically meaningful risk,
- and outperform simpler alternatives.

---

## A.2 Conceptual Basis of Weighting

The DQI was designed around three core principles:

1. **Risk sparsity**
   Most subjects are operationally clean; risk is concentrated in a small minority.

2. **Severity and persistence dominate risk**
   Prolonged delays, repeated SAE events, and systemic data instability are more damaging than isolated hygiene issues.

3. **Presence ≠ severity**
   Binary indicators signal entry into a risk regime but should not dominate score magnitude.

Accordingly, higher weights were assigned to:

- duration-based metrics (e.g., overdue days),
- severity-heavy counts (e.g., open SAE burden),
  while lower weights were assigned to:
- binary or near-binary hygiene indicators.

---

## A.3 Empirical Evidence Supporting Weight Allocation

### A.3.1 Distributional Evidence (Phase 3.2)

Across all core operational signals:

- **95–99.8% of subjects exhibit zero issues**
- Non-zero subjects show **heavy right-tailed distributions**
- Severity escalates rapidly once risk appears

Examples:

- `max_days_visit_overdue`: max = **373 days**
- `open_sae_count`: max = **49**
- `inactivated_forms_count`: max = **170**

This confirms that **risk magnitude differs by orders of magnitude**, justifying non-uniform weighting.

---

### A.3.2 Zero vs Non-Zero Stratification (Phase 3.2.3)

| Feature            | % Non-Zero | Median (Non-Zero) | 90th %ile | Max |
| ------------------ | ---------- | ----------------- | --------- | --- |
| EDRR open issues   | 5.37%      | 2                 | 10        | 17  |
| Open SAE count     | 3.52%      | 2                 | 14.9      | 49  |
| Inactivated forms  | 0.16%      | 37                | 163.6     | 170 |
| Visit overdue days | 4.21%      | 13.5              | 45        | 373 |

This demonstrates:

- a clear **two-regime structure** (clean vs risky),
- rapid severity escalation within the risky regime.

---

### A.3.3 Correlation & Redundancy Check (Phase 3.3)

- No raw operational signal shows correlation > **0.7** with another independent signal.
- Duration metrics and count metrics are **complementary**, not redundant.
- No single feature dominates the final DQI.

This confirms that weights do **not double-count** the same information.

---

### A.3.4 Penalty Attribution (Phase 3.4.2)

Among penalized subjects (**8.11% of total population**):

| Component Group        | Mean Correlation with Penalty |
| ---------------------- | ----------------------------- |
| Severity & persistence | **−0.53**                     |
| Hygiene indicators     | −0.35                         |
| Quality scores         | **+0.79**                     |

Interpretation:

- Penalties increase primarily with **severity and persistence**.
- Hygiene signals act as secondary contributors.
- Base quality moderates penalties rather than being overridden.

---

### A.3.5 Counterfactual Stress Tests (Phase 3.5)

Two naive alternatives were evaluated:

1. Unweighted raw-sum risk score
2. Binary presence score

Findings:

- Both showed high rank correlation (~0.99) with DQI (expected monotonicity).
- Both **failed** to:

  - differentiate extreme severity,
  - preserve stability within the worst-risk tail,
  - avoid numeric explosion or collapse.

The final DQI uniquely preserved **severity resolution where it matters**, proving that the weighting encodes information lost by simpler schemes.

---

## A.4 Conclusion

The weighting scheme used in the DQI is:

- grounded in observed data behavior,
- aligned with clinical operational risk semantics,
- empirically validated against multiple failure modes,
- and demonstrably superior to naive alternatives.

Accordingly, the weights are **justified, non-arbitrary, and fit for purpose**.

---
