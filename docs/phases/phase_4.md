# **Phase 4 — Risk Stratification & Predictive Modeling**

## **Purpose of Phase 4**

The purpose of Phase 4 is to **translate validated data quality measurements into actionable intelligence**. While earlier phases focused on _measurement, construction, and validation_ of a Data Quality Index (DQI), Phase 4 operationalizes these results by:

1. **Stratifying subjects into risk tiers** based on data quality
2. **Predicting future data quality degradation** using machine learning
3. **Supporting proactive operational decision-making** in clinical data monitoring

This phase represents the transition from **descriptive analytics to decision support and prediction**.

---

## **Positioning Within the Overall Framework**

| Phase       | Role                                                |
| ----------- | --------------------------------------------------- |
| Phase 1     | Schema understanding and semantic alignment         |
| Phase 2     | Metric engineering and DQI construction             |
| Phase 3     | Statistical validation and exploratory analysis     |
| **Phase 4** | **Risk stratification and predictive intelligence** |

Phase 4 **does not introduce new raw signals**. All modeling and stratification rely strictly on **validated outputs from Phases 2 and 3**, ensuring traceability and methodological continuity.

---

## **Phase 4 Structure Overview**

Phase 4 is divided into four tightly coupled sub-phases:

1. **Risk Stratification Design & Validation**
2. **Feature Engineering for Predictive Modeling**
3. **Predictive Model Development**
4. **Explainability & Operational Translation**

Each sub-phase includes **explicit validation checkpoints** to prevent propagation of uncertainty into downstream modeling.

---

# **4.1 Risk Stratification Design & Validation**

## **Objective**

To convert continuous DQI values into **stable, interpretable, and operationally meaningful risk tiers**, which will later serve as **labels** for predictive modeling.

This step is critical:

> Incorrect or weak stratification directly leads to misleading machine learning models, regardless of algorithmic sophistication.

---

## **4.1.1 Inputs**

The stratification process uses only **previously validated subject-level metrics**:

- Final **DQI score**
- `naive_risk_score`
- `binary_risk_score`
- Component metrics:

  - Open issue counts
  - Missing visits/pages/labs
  - Overdue durations

- Subject identifiers (study, site, subject)

No additional transformations or derived signals are introduced at this stage.

---

## **4.1.2 Risk Tier Semantics**

Risk tiers are defined **by operational meaning**, not arbitrary numeric splits.

| Tier            | Definition                       | Operational Interpretation    |
| --------------- | -------------------------------- | ----------------------------- |
| **Low Risk**    | High-quality, stable data        | Routine monitoring sufficient |
| **Medium Risk** | Localized or emerging issues     | Targeted review recommended   |
| **High Risk**   | Structural or compounding issues | Immediate escalation required |

Numeric thresholds must align with these meanings.

---

## **4.1.3 Candidate Threshold Strategies**

To avoid bias, **multiple thresholding strategies are evaluated in parallel**.

### **Strategy A — Distribution-Based**

- Uses DQI percentiles (e.g., bottom 10–15% as High Risk)
- Advantages: simple, reproducible
- Limitations: may not align with issue severity

### **Strategy B — Outcome-Aligned**

- Thresholds chosen where:

  - Issue density increases sharply
  - Overdue durations escalate

- Advantages: operational relevance
- Limitations: requires careful validation to avoid overfitting

### **Strategy C — Consistency-Aligned**

- Thresholds selected where:

  - DQI rank
  - Naive score rank
  - Binary score rank
    converge in ordering

- Advantages: reinforces Phase 3 validation results
- Limitations: conservative, may reduce sensitivity

No single strategy is assumed correct; thresholds are accepted only after validation.

---

## **4.1.4 Validation Framework for Risk Thresholds**

Each proposed threshold set must pass **all** of the following checks.

### **Check 1 — Internal Separation**

For each tier, compute:

- Mean and median issue counts
- Mean overdue durations
- Missingness rates

**Requirement:**
All metrics must show monotonic worsening from Low → Medium → High.

---

### **Check 2 — Effect Size Validation**

Effect sizes (e.g., Cohen’s d, Cliff’s delta) are computed between tiers.

**Requirement:**
Medium vs High must demonstrate **non-trivial practical separation**, not merely statistical significance.

---

### **Check 3 — Cross-Study and Cross-Site Stability**

Risk tier distributions are examined across:

- Studies
- Sites (where applicable)

**Requirement:**
No tier dominance caused by structural artifacts (e.g., a single site populating nearly all High Risk cases).

---

### **Check 4 — Rank Consistency**

Correlation is measured between:

- DQI rank
- Ordinal risk tier encoding
- Naive and binary risk scores

**Requirement:**
Rank correlations remain high and consistent with Phase 3 results, with no systematic inversions near thresholds.

---

### **Check 5 — Boundary Stress Testing**

Subjects near threshold boundaries are manually inspected.

**Requirement:**
Subjects just above and below thresholds must exhibit similar raw profiles, indicating thresholds are not noise-driven.

---

## **4.1.5 Threshold Acceptance Criteria**

A threshold is finalized only if it:

- Passes all validation checks
- Has clear operational interpretation
- Produces actionable tier sizes
- Is stable under small perturbations (±2–5%)

Rejected thresholds are explicitly documented to ensure transparency.

---

## **4.1.6 Outputs of Risk Stratification**

Before proceeding to predictive modeling, the following artifacts are finalized:

1. Numeric thresholds for each risk tier
2. Justification for threshold selection
3. Validation tables and plots
4. Explicit rejection rationale for alternatives
5. A standalone **Risk Stratification Justification Appendix**

---

# **4.2 Feature Engineering for Predictive Modeling**

## **Objective**

To construct a **model-ready feature set** that captures both current data quality state and early warning signals of degradation.

### **Feature Sources**

- Phase 2 engineered metrics
- Phase 3 validated composite scores
- Optional temporal derivatives (if timestamps exist)

### **Label Definition**

Prediction targets are defined based on Phase 4.1 outcomes, such as:

- Transition into High Risk within a defined horizon
- DQI dropping below an accepted threshold

Strict leakage checks are applied.

---

# **4.3 Predictive Model Development**

## **Objective**

To predict **future data quality deterioration** while maintaining interpretability.

### **Modeling Approach**

- Baseline: Logistic Regression, Decision Trees
- Advanced: Random Forest, Gradient Boosting (if justified)

### **Evaluation Metrics**

- ROC-AUC
- Precision / Recall for High Risk
- Calibration curves
- Confusion matrices by tier

Model complexity is increased **only if interpretability is preserved**.

---

# **4.4 Explainability & Operational Translation**

## **Objective**

To ensure model outputs are **understandable and actionable**.

### **Explainability Techniques**

- Global feature importance
- Local explanations (per subject)
- Driver decomposition for risk escalation

### **Operational Mapping**

- Monitoring prioritization
- Escalation triggers
- Comparison of rule-based vs ML-predicted risk

Discordant cases are analyzed explicitly.

---

## **Final Phase 4 Deliverables**

By completion of Phase 4, the project produces:

1. A validated risk stratification framework
2. An explainable predictive model
3. Quantified performance metrics
4. Deployment-ready decision logic
5. A full end-to-end narrative:

   > Clinical signals → DQI → Risk tiers → Predictions → Actions

---
