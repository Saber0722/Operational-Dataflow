# 📘 Phase 3 Documentation

## Exploratory Data Analysis (EDA) on `clinical_data_master_subject_index.parquet`

---

## 1. Phase 3 Context & Positioning

### Where Phase 3 sits in the pipeline

| Phase       | Purpose                                      | Status         |
| ----------- | -------------------------------------------- | -------------- |
| Phase 1     | Schema understanding & canonicalization      | ✅ Completed   |
| Phase 2     | Subject-level aggregation & DQI construction | ✅ Completed   |
| **Phase 3** | **EDA, explainability & weight validation**  | **🟡 Current** |
| Phase 4     | Modeling / monitoring / dashboards           | ⏭️ Next        |

Phase 3 **does not modify** the dataset or the weights.
It **evaluates and explains** them.

---

## 2. Phase 3 Objectives (Authoritative)

Phase 3 has **five non-negotiable objectives**:

### **Objective 3.1 — Structural & Statistical Validation**

Ensure the master subject index is:

- Internally consistent
- Free of pathological features
- Suitable for downstream analytics

---

### **Objective 3.2 — Operational Risk Behavior Understanding**

Understand how clinical operational failures manifest:

- Frequency
- Severity
- Concentration across subjects

---

### **Objective 3.3 — Empirical Justification of Weight Design**

Provide **data-backed evidence** that:

- Higher weights correspond to higher severity signals
- Lower weights correspond to weaker or noisier signals
- The final DQI is not arbitrary

---

### **Objective 3.4 — Explainability & Trust**

Ensure that:

- DQI behavior is interpretable
- High/low scores align with intuitive operational risk
- The index can be explained to non-technical stakeholders

---

### **Objective 3.5 — Readiness for Phase 4**

Clearly state:

- What the dataset is ready for
- What it is not yet suitable for
- What assumptions are now validated

---

## 3. Audience & Communication Strategy

### Audience: **Mixed**

- Clinical Operations
- Data Science
- Audit / Review stakeholders

### Strategy

- **Notebook-first**, with:

  - Clear plots
  - Minimal jargon in summaries

- **Executive markdown summary after each plot**, written jointly after visual inspection
- Strict separation between:

  - Observation
  - Interpretation
  - Implication

---

## 4. Phase 3 Deliverables

### 📓 Primary Deliverable

**Jupyter Notebook** containing:

- Sequential EDA sections
- All plots
- Inline explanations
- Embedded markdown summaries after each figure

### 📄 Secondary Deliverable

**Executive Markdown Summary**

- Compiled from per-figure summaries
- Serves as:

  - Review document
  - Audit artifact
  - Phase 4 handoff note

---

## 5. Phase 3 Analytical Structure (Final)

This structure will be followed **exactly** in the notebook.

---

### **Section 3.1 — Dataset Overview & Structural Sanity**

**Purpose:**
Confirm the dataset behaves exactly as designed in Phase 2.

**Analyses:**

- Dataset shape & feature roles
- Missingness patterns
- Feature range checks
- Zero / near-zero variance features

**Outcome:**
Confidence that no structural issues undermine later conclusions.

---

### **Section 3.2 — Univariate Distribution Analysis**

**Purpose:**
Understand natural distributions of operational risk indicators.

**Analyses:**

- Histograms / KDEs
- Log-scale views for skewed variables
- Zero-inflation checks

**Key Insight Sought:**
Operational risk should be **long-tailed**, not uniform.

---

### **Section 3.3 — Severity Gradient Validation (Weight Logic Support)**

**Purpose:**
Validate that features deemed “severe” in Phase 2 are empirically severe.

**Analyses:**

- Spread comparison across:

  - Count-based metrics
  - Duration-based metrics

- Percentile and tail behavior
- Variance comparisons

**Weight Justification Angle:**
Higher variance + heavier tails → higher weight justified.

---

### **Section 3.4 — Correlation & Redundancy Analysis**

**Purpose:**
Ensure the DQI is not dominated by redundant signals.

**Analyses:**

- Correlation heatmaps
- Pairwise relationships
- Qualitative redundancy assessment

**Weight Justification Angle:**
Weights mitigate correlation rather than amplify it.

---

### **Section 3.5 — DQI Decomposition & Contribution Analysis (Core Section)**

**Purpose:**
Explain _why_ subjects receive their DQI scores.

**Analyses:**

- Component-wise contribution plots
- DQI vs component scatter plots
- Top vs bottom decile comparison

**Critical Outcome:**
Low DQI subjects are penalized for **meaningful reasons**, not noise.

---

### **Section 3.6 — Counterfactual Stress Checks (EDA-only)**

**Purpose:**
Demonstrate why the chosen weighting scheme outperforms naive alternatives.

**Analyses:**

- Weighted vs unweighted score comparison
- Rank stability & displacement analysis

**Interpretation Rule:**
This is **comparative validation**, not optimization.

---

### **Section 3.7 — Subject Archetypes (Descriptive Patterns)**

**Purpose:**
Reveal recognizable operational failure profiles.

**Analyses:**

- Multi-feature visualizations
- Descriptive archetype labeling

**Value:**
Improves interpretability for clinical teams.

---

### **Section 3.8 — Phase 3 Summary & Decisions**

**Purpose:**
Formally conclude Phase 3.

Includes:

- Evidence-backed justification of weights
- Limitations
- Phase 4 readiness statement

---

## 6. Weight Justification: Formal Claim Boundaries

### What Phase 3 **WILL Claim**

- The weight design is:

  - Reasonable
  - Data-aligned
  - Non-arbitrary

- It improves interpretability and risk separation

### What Phase 3 **WILL NOT Claim**

- Optimality
- Uniqueness
- Statistical optimal weighting
- Performance superiority on outcomes

This preserves scientific and regulatory integrity.

---

## 7. Working Protocol During EDA (Agreed)

1. We generate **one figure at a time**
2. You review the figure
3. Together we:

   - Interpret behavior
   - Validate assumptions
   - Write the executive markdown summary

4. Only then do we move forward

This ensures:

- No rushed conclusions
- No post-hoc rationalization
- Strong narrative coherence

---

## 8. Phase 3 Exit Criteria

Phase 3 is complete when:

- Weight logic is empirically supported
- DQI behavior is explainable
- Stakeholders can trust the index
- Phase 4 can begin without revisiting assumptions

---
