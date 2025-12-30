# 📘 Phase 3 — Final Consolidated Documentation

## Exploratory Data Analysis & DQI Validation

---

## 1. Objective of Phase 3

Phase 3 was conducted to:

- validate the integrity of the subject-level master dataset,
- explain the behavior of the Data Quality Index (DQI),
- and empirically justify the design and weighting choices made in Phase 2.

Phase 3 is **explanatory and diagnostic**, not optimizational.

---

## 2. Dataset Integrity & Readiness

### 2.1 Structural Validation (Phase 3.1)

- Dataset shape: **5,461 subjects × 29 features**
- `master_subject_id` confirmed unique → true subject-level index
- 28/29 features show **0% missingness**
- One aggregate feature (`total_open_issue_count_per_subject`) shows structural NaNs due to absence of contributing records (treated as zero by design)

No structural anomalies or leakage were detected.

---

## 3. Operational Risk Landscape (Phase 3.2)

### 3.1 Risk Sparsity

- Between **95% and 99.8%** of subjects have zero operational issues across major signals.
- Risk is concentrated in a **small minority** of subjects.

### 3.2 Severity & Tail Behavior

- Raw signals exhibit heavy right tails.
- Log-scale analysis reveals **order-of-magnitude separation** between clean and severe subjects.
- Duration-based metrics show the largest variance, validating their role as severity amplifiers.

---

## 4. Correlation & Signal Independence (Phase 3.3)

- Raw risk signals are largely independent.
- Moderate correlations reflect logical relationships (e.g., missing visits ↔ overdue days).
- No evidence of harmful redundancy or signal hijacking.
- Composite quality scores behave as intended hierarchical aggregates.

---

## 5. DQI Explainability (Phase 3.4)

### 5.1 Selective Penalization

- **91.9% of subjects receive zero penalty**
- Only **8.1%** are penalized
- Maximum penalty observed: **0.50**

This confirms that DQI does **not degrade clean data**.

---

### 5.2 Why Subjects Are Penalized

- Penalized subjects show extreme values in:

  - overdue duration,
  - SAE burden,
  - inactivated records,

- Hygiene signals remain secondary contributors.

Penalty attribution aligns with clinical intuition and operational severity.

---

## 6. Counterfactual Validation (Phase 3.5)

- Naive and binary scoring approaches fail to preserve severity resolution.
- They either:

  - over-amplify minor issues, or
  - collapse catastrophic failures.

- Final DQI uniquely balances:

  - compression of clean subjects,
  - expansion of the high-risk tail.

---

## 7. Final Assessment

Phase 3 demonstrates that:

- The master dataset is structurally sound and analysis-ready.
- Operational risk is sparse, heavy-tailed, and severity-driven.
- The DQI:

  - penalizes the right subjects,
  - for the right reasons,
  - to the right degree.

- The weighting scheme is empirically defensible and operationally meaningful.

---
