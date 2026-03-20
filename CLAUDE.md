# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an R-based epidemiological analysis project — a Living Systematic Review (LSR) of COVID-19 clinical trials registered in a trial registry. It analyzes pharmaceutical interventions, registration trends, sample sizes, geographic distribution, and drug redundancy across pandemic eras.

**Data source:** REDCap database export (`data/data_2402.csv`) from the IDDO Tropical Medicine group.

## Running the Project

Scripts must be run sequentially in RStudio (or via `rmarkdown::render()`):

1. `scripts/01_clean1.Rmd` — raw data ingestion and QC filtering
2. `scripts/02_clean2.Rmd` — standardization and intervention processing
3. `scripts/03–07_results_*.Rmd` — individual analysis sections (can run in any order after step 2)

**Important:** Scripts contain hard-coded absolute paths to a Windows OneDrive directory (`C:/Users/rchhajed/OneDrive - Nexus365/...`). These must be updated to match the local environment before running.

**Required R packages:** tidyverse, lubridate, readr, scales, countrycode, ggplot2, rlang

## Data Pipeline Architecture

The project follows a sequential ETL → analysis pattern:

### Phase 1: `01_clean1.Rmd` — Raw Data Cleaning
- **Input:** `data/data_2402.csv` (raw REDCap export)
- Extracts three table types from REDCap's mixed-instrument export:
  - Study-level rows (non-repeating instruments)
  - Study–country relationships (pivoting `st_ctry*_name` columns)
  - Arm-level data (repeating "Study Arm" records)
- Applies inclusion filters: eligible=Yes, not cancelled, registered ≥ March 2020, pharmaceutical RCTs only
- Standardizes country names via `countrycode`
- **Output tables:** `study_table`, `country_table`, `arm_table`

### Phase 2: `02_clean2.Rmd` — Standardization
- **Input:** Tables from Phase 1
- Pivots intervention slots (`sa_intv1–sa_intv10_*`) to a component-level table (1 row = 1 arm–drug pair)
- Maps free-text drug names to standardized categories
- **Output tables:** `arm_drug_table`, `study_drug_table`, `pharma_table`, `pharma_table1`
- **Saves:** `data/covid_lmr_clean7_YYYYMMDD.RData` (loaded by all analysis scripts)

### Phase 3: Analysis Scripts (03–07)
Each loads the `.RData` file and produces an HTML report:

| Script | Content |
|--------|---------|
| `03_results_s1s2.Rmd` | Registration trends (quarterly/cumulative), recruitment status |
| `04_results_s3.Rmd` | Sample size distribution and temporal trends |
| `05_results_s4.Rmd` | Trial phases; era stratification (Early/Mid/Post-PHEIC) |
| `06_results_s5.Rmd` | Geographic analysis, country income group stratification |
| `07_results_duplicatestudies.Rmd` | Drug redundancy after WHO stop dates (HCQ, Ivermectin, LPV/r, Convalescent Plasma) |

## Key Domain Concepts

- **Pandemic eras:** Early (to May 2021), Mid-pandemic (May 2021–May 2023), Post-PHEIC (May 2023+)
- **Phase mapping:** Phase 1/2 → Exploratory; Phase 3/4 → Confirmatory
- **WHO stop dates tracked:** Hydroxychloroquine (2020-07-04), Ivermectin (2021-03-31), Lopinavir/Ritonavir (2020-07-04), Convalescent Plasma (2021-12-07)
- **Income groups:** Classified via `countrycode` package (World Bank income group)
