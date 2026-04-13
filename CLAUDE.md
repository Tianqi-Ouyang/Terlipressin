# CLAUDE.md — Main Terlipressin Project

This is the main HARMONY Terlipressin project. Shared context (dataset, variables, conventions) is in the parent `CLAUDE.md`.

## Repository Structure

- `code/Tables.Rmd` — Main analysis: data ingestion from 15 CSVs, QC checks, derived variable creation, Table 1 generation, survival analysis, competing risk models (~1600 lines).
- `code/revision 0311.Rmd` — Revision analysis: center grouping, dose-change categorization, multiple imputation (MICE m=10, PMM), Fine-Gray competing risk models with Rubin's rules pooling via `run_pooled_fg()`.
- `code/shared_pipeline.qmd` — Shared pipeline for sub-projects: loads master xlsx, defines helper functions, computes all standard derived variables.
- `data/final_master_01282026.xlsx` — Cleaned final dataset (used by all sub-projects).
- `docs/` — Quarto website with variable dictionaries and analysis examples.
- `Terlipressin Database _ REDCap Code Book.pdf` — REDCap data dictionary (132 pages, 1500+ fields across 18 instruments).

## Raw Data Pipeline

This project (unlike sub-projects) also handles raw data ingestion:

1. **Raw data** read from `~/Partners HealthCare Dropbox/Tianqi Ouyang/De-identified Data/{CENTER}/` (15 separate CSVs)
2. Each center gets an `id` column (`center_rownum`) and columns are alphabetically sorted
3. `reorder_and_bind()` merges all 15 dataframes
4. `qc_check()` validates missing data and determines `Discont_Day`
5. Excluded subjects: 7 BID-PTD, 24 MAYO (5-28), 3 MMC; re-treatment subjects separated
6. Derived variables computed, final dataset exported to xlsx

## Rendering

```bash
# Render documentation site
quarto render docs/

# Main analysis — run interactively in RStudio (requires Dropbox access)
# Tables.Rmd reads raw CSVs from Partners HealthCare Dropbox
```
