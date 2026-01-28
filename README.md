# NEST — Operational Dataflow

Comprehensive repository for NEST operational data aggregation, validation, and analysis pipelines. The project collects, cleans, aggregates, and analyzes clinical operational datasets (EDC metrics, missing pages, lab issues, SAE dashboards, coding reports, etc.) and produces reusable aggregated CSVs and analytical notebooks.

This README documents the repository layout, how to set up the environment, and gives a detailed description of each notebook and the main documentation files.

## Quick facts / contract
- Inputs: raw study data files (anonymized) stored under `data/raw` and project ancillary CSVs in `data/intermediate`.
- Outputs: aggregated CSV reports in `data/intermediate`, figures and analysis notebooks in `notebooks/`.
- Success criteria: each aggregation notebook produces one or more aggregated CSVs named in `data/intermediate/` and a small set of exploratory figures; notebooks are runnable in order with the provided `requirements.txt`.

Edge cases considered: missing or partially-anonymized input files, very large raw files (use chunking or parquet), inconsistent column names across studies, and duplicate rows.

---

## Repository layout

- `notebooks/` — Jupyter notebooks that perform data aggregation, validation, and analysis. Detailed description below.
- `data/`
  - `raw/` — raw input data (anonymized study files). Large, not tracked in full in repo.
  - `intermediate/` — produced aggregated CSVs used downstream (examples: `compiled_edrr_agg.csv`, `sae_dashboard_agg.csv`, etc.).
  - `master/` — (intended) master tables and canonical datasets.
- `docs/` — project artifacts, phase reports, design docs and appendices.
- `src/` — helper code, loaders, and aggregation utilities (organize reusable functions called by notebooks).
- `notebooks/` — collection of analysis and aggregation notebooks (see next section).
- `requirements.txt` — Python packages used by notebooks.

---

## Environment and setup

Recommended: create an isolated Python environment and install the dependencies in `requirements.txt`.

Example (bash):

```bash
# create and activate a venv
python3 -m venv .venv
source .venv/bin/activate

# install requirements
pip install --upgrade pip
pip install -r requirements.txt
```

Notes:
- If you work with very large raw files, prefer `pyarrow`, `fastparquet` and store intermediate datasets as parquet for faster I/O.
- Use `jupyter lab` or `nbclassic` to run notebooks interactively.

---

## Notebooks — detailed descriptions

The notebooks are named numerically to indicate a common workflow order. Below each notebook has a short summary, inputs, outputs, and key steps.

- `01_CPID_EDC_Metrics_aggregation.ipynb`
  - Purpose: Aggregate EDC (electronic data capture) operational metrics per CPID (center or study identifier) into a consolidated CSV.
  - Inputs: raw CPID input files from `data/raw/...` and any mapping files in `data/intermediate`.
  - Outputs: `cpid_edc_metrics_agg.csv` in `data/intermediate`.
  - Steps: read multiple study-specific sheets/files, standardize column names, compute per-CPID metrics (completeness, missing pages), and save aggregated results.

- `02_Global_Missing_Pages_Report_aggregation.ipynb`
  - Purpose: Detect and aggregate global missing pages across studies and produce a consolidated missing pages report.
  - Inputs: study page logs and input files in `data/raw` or preprocessed CSVs.
  - Outputs: `global_missing_pages_agg.csv` in `data/intermediate`.
  - Steps: harmonize page identifiers, flag missing pages, aggregate by study/CPID/form, and create summary visuals.

- `03_Visit_Projection_Tracker_aggregation.ipynb`
  - Purpose: Produce visit projection tracking tables for subject visits and timelines to support operational planning.
  - Inputs: visit schedules and subject-level enrollment dates from `data/raw`.
  - Outputs: `visit_projection_tracker_agg.csv` in `data/intermediate`.
  - Steps: compute projected visit dates, compare planned versus actual, and aggregate status metrics by site and study.

- `04_Missing_Lab_Name_and_Missing_Ranges_aggregation.ipynb`
  - Purpose: Identify missing lab test name mappings and missing numeric ranges in lab result fields.
  - Inputs: lab result files, lab coding/mapping tables.
  - Outputs: `missing_lab_issues_agg.csv` (or similar) in `data/intermediate`.
  - Steps: normalize lab test names, flag missing or out-of-range numeric values, and aggregate issues for review.

- `05_SAE_Dashboard_aggregation.ipynb`
  - Purpose: Aggregate Serious Adverse Event (SAE) data to produce a dashboard-ready dataset.
  - Inputs: SAE reports from `data/raw`.
  - Outputs: `sae_dashboard_agg.csv` in `data/intermediate` and supporting charts.
  - Steps: standardize SAE severity/coding, compute time-to-report metrics, and build summary tables by site and event category.

- `06_Inactivated_Forms_and_Loglines_aggregation.ipynb`
  - Purpose: Identify inactivated forms and extract logline issues (form inactivation, reason, timestamps).
  - Inputs: CRF/form inventories and EDC logs.
  - Outputs: `inactivated_forms_loglines_agg.csv` in `data/intermediate`.
  - Steps: parse EDC loglines, join with form inventory, and summarize inactivated forms per study.

- `07_Compiled_EDRR_aggregation.ipynb`
  - Purpose: Compile EDRR (Electronic Data Review Reports / data review outputs) across studies into a single aggregated dataset.
  - Inputs: per-study EDRR output files.
  - Outputs: `compiled_edrr_agg.csv` in `data/intermediate`.
  - Steps: harmonize EDRR schemas across studies, deduplicate, and compute aggregate metrics.

- `08_Coding_Reports_MedDRA_WHO_aggregation.ipynb`
  - Purpose: Aggregate coding reports using MedDRA and WHO dictionaries (e.g., AE coding, drug coding summaries).
  - Inputs: coding reports from vendor tools or in-house coders.
  - Outputs: `coding_reports_agg.csv` and `coding_reports_whodra_agg.csv` in `data/intermediate`.
  - Steps: normalize dictionary versions, map raw codes to standardized hierarchies, and produce counts and frequency tables.

- `09_Coding_Reports_WHODRUG_aggregation.ipynb`
  - Purpose: Specialized aggregation for WHO-Drug coded datasets (drug exposure and concomitant medication coding summaries).
  - Inputs: WHO-Drug coding outputs, medication reports.
  - Outputs: `coding_reports_whodra_agg.csv` or similarly named CSV in `data/intermediate`.
  - Steps: harmonize drug name mapping, aggregate by ATC or WHO-Drug hierarchy, and export summary tables.

- `EDA.ipynb`
  - Purpose: Exploratory Data Analysis across core aggregated tables. Useful for quick diagnostics, trends, and visual checks.
  - Inputs: aggregated CSVs in `data/intermediate`.
  - Outputs: visualizations and ad-hoc tables; no single canonical CSV output.
  - Steps: load aggregated data, compute summary statistics, plot distributions, and highlight anomalies.

- `Master_Subject_Table.ipynb`
  - Purpose: Build and validate the master subject-level table used by multiple downstream analyses and modeling.
  - Inputs: subject enrollment files, demographics, visit and events data.
  - Outputs: subject master table (saved to `data/master/` or `data/intermediate` depending on pipeline stage).
  - Steps: merge subject-level sources, deduplicate records, compute derived fields (age, study-phase flags), and validate consistency.

- `model_traiing.ipynb` (note: spelled `traiing`)
  - Purpose: Training environment for predictive models (risk stratification, prediction tasks described in Phase 4). Contains preprocessing, baseline models, and evaluation metrics.
  - Inputs: master subject table and feature-engineered datasets from `data/intermediate` or `data/master`.
  - Outputs: model artifacts, evaluation results, and example prediction outputs.
  - Steps: feature selection, model training (sklearn), cross-validation, and result visualization.

---

## Documentation files (brief descriptions)

- `docs/draft.txt` — project notes and drafts.
- `docs/6932c39b908b6_detailed_problem_statements_and_datasets/` — dataset bundle and problem statements used for the project (also provided as a zip).

Phases and appendices (under `docs/phases/`):
- `phase_1.md` / `Phase 1.pdf` — initial planning and design artifacts.
- `phase_2.md` / `Phase 2 - Master Table Design & Data Quality Framework.pdf` — master table design and the data quality framework.
- `phase_3.md`, `phase_3_validation.md` / `Phase 3 Part 1 - EDA.pdf`, `Phase 3 Part 2 - DQI Validation.pdf` — exploratory data analysis and data quality index validation.
- `phase_4.md` / `Phase 4 Part 1 - Risk Stratification & Predictive Modeling.pdf`, `Phase 4 Part 2 -Model Training.pdf` — model design and training artifacts.
- `appendix_data_validation.md` and `Appendix A (Phase 3) -Weight Justification for the Data Quality Index (DQI).pdf` — supporting appendices with validation procedures and DQI weight justification.
- `model_result.md` — narrative and results for modeling runs.

These documents explain the methodology, evaluation, and decisions made across project phases. The PDFs are the formal phase reports; the markdown files are lighter-weight notes and instructions.

---

## How to run the notebooks

1. Create and activate the Python environment as shown above.
2. Start Jupyter Lab or Notebook:

```bash
jupyter lab
```

3. Open notebooks in the recommended order for a fresh run (1 → 9):
   - Run the aggregation notebooks first (01 → 09) to populate `data/intermediate`.
   - Then run `Master_Subject_Table.ipynb` and `EDA.ipynb` for combined analyses.
   - Lastly, run `model_traiing.ipynb` with the master table as input.

Tips:
- If some raw files are missing, run notebooks on a per-file basis and use the helper functions in `src/loaders` to isolate file-specific transformations.
- For large datasets, change read_csv calls to read_parquet or use `chunksize` to stream process.

---

## Quality gates

- Build: N/A (not a compiled project). PASS if `pip install -r requirements.txt` succeeds locally.
- Lint / Typecheck: Not enforced for notebooks; consider adding pre-commit hooks and flake8/ruff for `src/` code.
- Tests: No automated tests included. Recommended next step: add a tiny pytest suite for key loader functions under `src/`.

---

## Next steps and suggestions

- Add a small `src/README.md` describing helper modules and public functions.
- Add a `CONTRIBUTING.md` with data access rules and notebook execution order.
- Add simple unit tests for core `src/loaders/*` functions and run them in CI.

---

If you'd like, I can:
- generate a `docs/index.md` that links each notebook and displays short descriptions, or
- create minimal unit tests for `src/loaders` and a test runner.

Contact / maintenance
- Maintainer: repository owner and primary contributors (see Git history).

---

End of README.
