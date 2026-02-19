# Antibiotic Prescribing & Salmonella AMR Resistance Analysis

An exploratory data analysis linking **outpatient antibiotic prescribing rates** (CDC, 2015–2022) to **antimicrobial resistance (AMR)** in clinical *Salmonella enterica* isolates (NCBI Pathogen Detection, 2015–2022) across US Census regions.

---

## Research Questions

1. Does higher regional antibiotic prescribing correlate with greater AMR prevalence in *Salmonella*?
2. Is population density independently associated with AMR burden?
3. Did COVID-19 (2020) disrupt regional prescribing patterns, and did recovery vary by region?

---

## Data Sources

| Dataset | Source | Years | Notes |
|---|---|---|---|
| Outpatient antibiotic Rx rates | [CDC Antibiotic Use](https://www.cdc.gov/antibiotic-use/data/report.html) | 2015–2022 | Prescriptions per 1,000 population, state level |
| 2019 Rx rates (local fallback) | `2019_report.csv` | 2019 | CDC archive page unavailable at scrape time |
| *Salmonella* isolate metadata | [NCBI Pathogen Detection FTP](https://ftp.ncbi.nlm.nih.gov/pathogen/Results/Salmonella/) | 2015–2022 | ~232k clinical/human USA isolates |
| State populations | [US Census Bureau API](https://api.census.gov) | 2020 | 2020 Decennial Census (P1_001N) |
| State land areas | US Census Bureau | Static | Square miles, hardcoded from official figures |

---

## Repository Structure

```
.
├── antibiotic_amr_analysis.ipynb   # Main analysis notebook (cleaned)
├── 2019_report.csv                 # Local fallback for 2019 CDC data
├── prescribing_vs_resistance.png   # Output: Rx rate vs AMR scatter
├── popdensity_vs_resistance.png    # Output: Pop density vs AMR scatter
├── resistance_correlations.png     # Output: 4-panel correlation overview
└── README.md
```

---

## Setup & Requirements

### Install Dependencies

```bash
pip install duckdb pandas numpy requests pyarrow lxml matplotlib scipy scikit-learn
```

Or run the first cell of the notebook, which handles installation automatically.

### Census API Key

The notebook uses a US Census Bureau API key to pull 2020 population data. If the hardcoded key expires, obtain a free key at [api.census.gov/data/key_signup.html](https://api.census.gov/data/key_signup.html) and update the `CENSUS_KEY` variable in **Section 5.1**.

---

## Notebook Structure

| Section | Description |
|---|---|
| **1. Setup** | Package installation |
| **2. CDC Data (2020–2022)** | Download & clean CSV-format prescribing data |
| **3. Extend to 2015–2019** | Scrape HTML reports + load local 2019 CSV |
| **4. Full Panel** | Concatenate all years; assign Census regions |
| **5. Population Density** | Census API fetch; compute persons/sq mi |
| **6. Feature Engineering** | YoY change, rolling mean, baseline, resilience flags |
| **7. QC** | Shape, dtypes, missing values, year coverage checks |
| **8. Salmonella AMR Data** | Stream NCBI metadata; map isolates to Census regions |
| **9. Merge** | Join prescribing panel with AMR summary on region × year |
| **10. Visualizations** | Scatter plots, regression models, correlation matrix |
| **11. Within-Region Analysis** | Region-stratified Pearson correlations |
| **12. National vs. Regional** | Validate regional subsample representativeness |
| **13. Summary & Limitations** | Key findings and caveats |

---

## Key Methods

**Prescribing data pipeline:**
- 2020–2022: direct CSV download from CDC
- 2015–2018: HTML scraping with `pd.read_html()` / regex fallback
- 2019: local CSV (archive page inaccessible)
- State names normalized via `norm_state()` before merging

**Salmonella AMR pipeline:**
- Streamed in 50,000-row chunks from the NCBI FTP server
- Filtered to `epi_type ∈ {clinical, human}`, USA, 2015–2022
- Region mapped via 5-priority lookup: HHS region > institution > abbreviation > state substring > geo_loc_name
- Core AMR defined as `number_core_amr_genes > 0` (excludes intrinsic *mdsA/mdsB* efflux genes)

**Statistics:**
- Pearson r with two-tailed p-values
- OLS and Ridge regression with leave-one-group-out CV (group = region)
- Within-region correlations to control for between-region confounders

---

## Key Results

- **Cross-region:** Moderate negative correlation (r = −0.55, p = 0.014) between average population density and core AMR prevalence — denser, lower-prescribing regions do *not* show higher Salmonella AMR.
- **Within-region (Midwest/Northeast):** AMR rose significantly over time (r > 0.93) while prescribing fell, suggesting temporal AMR trends are not primarily driven by local outpatient antibiotic use.
- **COVID disruption:** All regions showed sharp prescribing declines in 2020. Southern states were most resilient (smallest relative drop), with partial recovery across all regions by 2022.

---

## Limitations

- Only ~10% of NCBI Salmonella isolates could be mapped to a Census region (most report only "CDC" as collector), limiting statistical power for region-level inference.
- AMR genotype (gene presence) does not perfectly predict clinical phenotypic resistance.
- Ecological associations at the region level cannot establish individual-level causation.
- Population density from the 2020 Census is applied uniformly across all study years.
- The analysis covers outpatient antibiotic use only; agricultural and veterinary use are not captured.
