# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-center retrospective registry study (HARMONY) of terlipressin use for hepatorenal syndrome-acute kidney injury (HRS-AKI). Data collected from 15 hospitals via REDCap, merged and analyzed in R.

**Centers:** BID, BSW, CCF, CPM, CSF, INU, Mayo, MCM, MGH, MMC, NWM, OCH, PEN, UNM, Yale

## Repository Structure

- `code/Tables.Rmd` — Main analysis: data ingestion from 15 CSVs, QC checks, derived variable creation, Table 1 generation, survival analysis, competing risk models. This is the primary code file (~1600 lines).
- `code/revision 0311.Rmd` — Revision analysis: replicates data pipeline from Tables.Rmd, adds center grouping, dose-change categorization, multiple imputation (MICE m=10, PMM), Fine-Gray competing risk models with proper Rubin's rules pooling via `run_pooled_fg()`.
- `data/final_master_01282026.xlsx` — Cleaned final dataset.
- `Terlipressin Database _ REDCap Code Book.pdf` — REDCap data dictionary (132 pages, 1500+ fields across 18 instruments).

## Data Pipeline

1. **Raw data** read from `~/Partners HealthCare Dropbox/Tianqi Ouyang/De-identified Data/{CENTER}/` (15 separate CSVs)
2. Each center gets an `id` column (`center_rownum`) and columns are alphabetically sorted
3. Consult date and access group columns dropped for consistency
4. `reorder_and_bind()` merges all 15 dataframes
5. `qc_check()` function identifies missing data and determines `Discont_Day`
6. Excluded subjects: 7 BID-PTD, 24 MAYO (5-28), 3 MMC; re-treatment subjects separated
7. Derived variables computed (scores, outcomes, staging) — see variable dictionary
8. Final dataset exported to xlsx

## Key Derived Variables

- **Discont_Day**: Day (0-13) terlipressin was discontinued (first `discont_terli_dayN == 1`)
- **HRS_reversal_new**: Primary outcome — `lowest_creatinine <= 1.5 AND (scr_day0 - lowest_creatinine) >= 0.3`
- **hrs_responders_cat_2**: Binary responder (any AKI stage improvement)
- **time_to_death_90days_cmp / time_to_death_status_90days_cmp**: Competing risk outcome (death vs transplant, 90-day)
- **akin / discont_aki_stage**: AKIN staging at initiation and discontinuation
- **MELD scores**: `admit_meld`, `admit_meld_na`, `admit_meld_3`, `day0_meld`, `day0_meld_na`, `day0_meld_3`
- **CLIF-C score**: `clif_c_score` / `clif_c_score_new` (median-imputed)
- **Child-Pugh score**: `child_pugh_score`
- **baseline_scr_new**: Best baseline creatinine (outpatient > 120-365d value > min(CKD-EPI estimate, admit SCr))

## R Dependencies

tidyverse, tableone, gtsummary, survival, survminer, readxl, writexl, xlsx, ggplot2, ggpubr, splines, lattice, mice, DescTools, finalfit, tidycmprsk, ggsurvfit, sqldf, table1, cobalt, riskRegression, prodlim, pammtools, adjustedCurves

## Rendering

```bash
# Render variable dictionary
quarto render code/variable_dictionary.qmd

# Render main analysis (requires raw data access)
# Run Tables.Rmd interactively in RStudio — it reads from Partners HealthCare Dropbox
```

## Coding Conventions

- Variable names are lowercase with underscores
- REDCap fields use pattern `{lab}_terli_day{N}` for daily treatment labs (day 0-13)
- Post-discontinuation SCr: `scr_day{N}_post_terli`
- MAP measured 4x daily: `map_terli_day{N}_time{0-3}`
- Availability flags: `dc_available_{lab}___999` (checkbox, 999 = not available)
- Binary outcomes: 1 = event, 0 = no event
- `rowwise()` used extensively for dynamic column access via `get(paste0(...))`
