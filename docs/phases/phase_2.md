# Phase 2 — Master Table Design & Data Quality Framework (LOCKED)

---

## Phase 2 Objective

Design a **subject-centric master integration layer** that:

- Integrates all standardized datasets from Phase 1
- Produces reliable, explainable subject-level metrics
- Enables data quality scoring, readiness assessment, and downstream AI use
- Preserves auditability and regulatory safety

---

## Phase 2 · Step 1 — Master Grain & Dataset Classification (LOCKED)

### Master Table Grain

```
1 row = Study + Site + Subject
```

This grain:

- Aligns with clinical readiness and submission logic
- Supports rollups to site and study
- Avoids duplication and double counting

---

### Dataset Classification

#### Bucket A — Direct Master Inputs

(Already subject-level or natively resolved)

- **CPID_EDC_Metrics**
- **Compiled_EDRR**

These form the **spine** of the master table.

---

#### Bucket B — Aggregated Contributors

(Many-to-one → summarized to subject)

- Visit Projection Tracker
- Global Missing Pages Report
- Inactivated Forms & Loglines
- Missing Lab Name & Missing Ranges
- SAE Dashboard

These contribute **counts, flags, and derived indicators**.

---

#### Bucket C — Drill-down / Semantic Fact Tables

(Too granular / semantic for master)

- GlobalCodingReport_MedDRA
- GlobalCodingReport_WHODRA

Only **aggregated signals** enter the master; raw rows remain external.

---

## Phase 2 · Step 2 — Master Subject Table Schema (LOCKED)

### Table Role

The **Master Subject Table** is:

- The single source of truth for subject readiness
- Used by dashboards, analytics, and AI
- Stable, deterministic, and explainable

It is **not**:

- A raw data table
- An audit log
- An event-level fact table

---

### Column Groups

#### 1. Canonical Identifiers (Primary Key)

```
study_id
region
country
site_id
subject_id
```

Composite PK:

```
(study_id, site_id, subject_id)
```

---

#### 2. Subject Status & Lifecycle

```
subject_status
overall_subject_status
latest_completed_visit
visit_status_summary
```

---

#### 3. Visit & Page Completeness (Derived)

```
expected_visits
completed_visits
missing_visits_count
max_days_visit_overdue

missing_pages_count
max_days_page_missing
```

---

#### 4. Query & Data Quality Metrics

```
total_queries
open_queries

queries_dm
queries_clinical
queries_medical
queries_site
queries_monitor
queries_coding
queries_safety

non_conformant_pages_count
total_crfs_with_issues
total_clean_crfs
percent_clean_crfs
```

---

#### 5. Verification, Lock & Signature Status

```
crfs_requiring_sdv
forms_verified

crfs_frozen
crfs_locked
crfs_unlocked

crfs_signed
crfs_never_signed
broken_signatures

overdue_signatures_45_days
overdue_signatures_45_90_days
overdue_signatures_90_plus_days
```

---

#### 6. Coding & Safety Summary (Aggregated)

```
coded_terms_count
uncoded_terms_count
coding_required_flag

open_sae_count
pending_sae_dm_review_flag
pending_sae_safety_review_flag
```

(No individual AE or drug terms stored)

---

#### 7. Lab & Reconciliation Health

```
missing_lab_issues_count
lab_ranges_missing_flag

edrr_open_issues_count
```

---

#### 8. Inactivation & Protocol Deviations

```
inactivated_forms_count
inactivated_records_flag

protocol_deviations_confirmed
protocol_deviations_proposed
```

---

#### 9. Composite & Readiness Fields (Derived)

```
is_clean_subject_flag
is_submission_ready_flag
subject_data_quality_score
```

---

### Explicit Exclusions from Master Table

- Individual visits
- Individual pages
- Individual lab tests
- Individual MedDRA / WHODrug terms
- Audit loglines
- Record-level discrepancies

These remain in **fact tables for drill-down only**.

---

## Phase 2 · Step 3 — Integration & Aggregation Logic (LOCKED)

### Integration Rules

1. All datasets integrate **into the master**, never into each other
2. One dataset → one responsibility
3. Aggregations are deterministic (`count`, `sum`, `max`, `exists`)
4. Null ≠ zero
5. CPID metrics take precedence when overlaps exist

---

### Dataset-Specific Integration

#### CPID_EDC_Metrics

- Join: `study + site + subject`
- Role: Master spine
- No aggregation applied

---

#### Compiled_EDRR

```
edrr_open_issues_count = SUM(total_open_issue_count)
```

---

#### Visit Projection Tracker

```
missing_visits_count   = COUNT(days_outstanding > 0)
max_days_visit_overdue = MAX(days_outstanding)
```

---

#### Global Missing Pages Report

```
missing_pages_count   = COUNT(*)
max_days_page_missing = MAX(days_page_missing)
```

---

#### Inactivated Forms & Loglines

```
inactivated_forms_count   = COUNT(DISTINCT form)
inactivated_records_flag = EXISTS(any record)
```

---

#### Missing Lab Name & Ranges

```
missing_lab_issues_count = COUNT(*)
lab_ranges_missing_flag  = EXISTS(ranges/units missing)
```

---

#### SAE Dashboard

```
open_sae_count                 = COUNT(open discrepancies)
pending_sae_dm_review_flag     = EXISTS(dm review pending)
pending_sae_safety_review_flag = EXISTS(safety review pending)
```

---

#### MedDRA Coding Report

```
uncoded_ae_terms_count =
  COUNT(coding_status = 'Uncoded' AND require_coding = 'Yes')
```

---

#### WHODrug Coding Report

```
uncoded_drug_terms_count =
  COUNT(coding_status = 'Uncoded' AND require_coding = 'Yes')
```

---

### Priority Order (for scoring & overrides)

```
Safety > Coding > Labs > Queries > Visits > Pages
```

---

## Phase 2 · Step 4 — Data Quality Metrics & Data Quality Index (LOCKED)

### Data Quality Dimensions

1. **Completeness**

   - Visits, pages, labs

2. **Conformity**

   - Non-conformant data
   - Protocol deviations

3. **Consistency**

   - Cross-field contradictions

4. **Timeliness**

   - Visit overdue days
   - Page missing days
   - Signature aging

5. **Critical Issue Presence**

   - Open SAE
   - Uncoded required terms
   - Open EDRR issues

---

### Subject-Level DQI (Base Model)

```
DQI =
  0.30 * Completeness +
  0.20 * Conformity +
  0.20 * Timeliness +
  0.10 * Consistency +
  0.20 * Query_Health
```

---

### Hard Overrides (Caps)

```
IF open SAE               → DQI ≤ 0.30
IF uncoded required terms → DQI ≤ 0.40
IF open EDRR issues       → DQI ≤ 0.50
```

---

### Clean Subject Flag (Strict)

```
is_clean_subject = TRUE if ALL:
- missing_visits == 0
- missing_pages == 0
- open_queries == 0
- uncoded_required_terms == 0
- open_sae_count == 0
- edrr_open_issues == 0
- overdue_signatures == 0
```

---

### Rollups

#### Site Level

```
site_avg_dqi
percent_clean_subjects
subjects_with_critical_issues
```

#### Study Level

```
study_readiness_index
sites_at_risk
```

---

### Explainability Requirement

Every score must decompose into:

- Which dimension failed
- Which underlying metrics caused the penalty

Mandatory for QA, CRAs, and AI outputs.

---
