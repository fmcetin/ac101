# Tesla Data for AC101

## Overview

This folder contains data for the AC101 Management Accounting course. We use Tesla as our flagship case throughout Part 1.

**Critical Distinction**: Some data is **real** (from public filings), some is **simulated** (based on industry benchmarks). This is clearly marked throughout.

---

## The student workbook — `Tesla_data.xlsx`

**The single file to upload to Moodle.** One sheet per CSV below, tab-coloured by provenance (REAL = green, SIMULATED = orange, DEMO = grey), fronted by a READ ME sheet carrying the sheet index and the data-quality notes from this README. Rebuild after **any** CSV edit:

```bash
.venv/bin/python data/teaching/make_tesla_data_xlsx.py
```

The CSVs remain the machine-readable source of truth — the Colab notebooks download them from gh-pages; students never need to handle raw CSVs.

---

## Data Files

### Real Data (`/raw/`)

| File | Description | Source | Coverage |
|------|-------------|--------|----------|
| `tesla_quarterly.csv` | Quarterly financials and production | Compustat (2023–24), Tesla IR (2025) | Q1 2023 - Q4 2025 |
| `tesla_annual_2024.csv` | FY2024 income statement | SEC 10-K | FY2024 |
| `tesla_production_by_model.csv` | Production split by model | Tesla delivery reports | Q1 2023 - Q4 2025 |
| `tesla_targets.csv` | Musk's public production targets | Earnings calls, statements | Various |

#### Data Quality Notes

1. **Q4 2025 financials**: Revenue and cost data not yet available (as of January 2026). Production/delivery figures are reported.

2. **Model split 2023-2024**: Tesla only began reporting Model 3/Y vs Other split in 2025. Earlier quarters are **estimated** based on delivery mix (~97% Model 3/Y historically).

3. **Quarterly COGS vs annual cost of revenues (Compustat convention)**: The quarterly `cogs_millions` column sums to **$74.87B for 2024** ($74.45B for 2023), not the $80.24B total cost of revenues in the FY2024 10-K. This is expected, not an error: the column is **Compustat COGS**, which **excludes depreciation, amortisation and impairment** ($5,368M in FY2024 per the 10-K cash-flow statement) that GAAP cost of revenues includes — Compustat reclassifies D&A to a separate item. Tesla's actual 10-Q quarterly cost of revenues ($17,605M / $20,922M / $20,185M / $21,528M) sums to exactly the annual $80,240M. **Implication for Session 2**: the regression estimates the fixed/variable split of *Compustat COGS (ex-D&A)* — a labelled proxy; since depreciation is a classic fixed cost, the intercept understates Tesla's total fixed production cost.

4. **SG&A convention**: `sga_millions` is Compustat XSGA, which **already includes R&D** (FY2024: $9,690M = $5,150M SG&A + $4,540M R&D). Do **not** add `rd_millions` to it. Note that `tesla_annual_2024.csv`'s `selling_general_admin` ($5,150M) is *pure* SG&A per the 10-K — the two files define "SG&A" differently.

### Simulated Data (`/simulated/`)

| File | Purpose | Session | Methodology |
|------|---------|---------|-------------|
| `overhead_cost_pools.csv` | ABC cost pools | Session 3 | Industry benchmarks for auto manufacturing |
| `product_activity_drivers.csv` | ABC activity drivers | Session 3 | Teardown reports, industry estimates |
| `battery_make_vs_buy.csv` | Make-or-buy analysis | Session 4 | BloombergNEF battery index, industry reports |
| `variance_analysis_q4.csv` | Variance analysis | Session 5 | Constructed with actual Q4 2024 as base |

#### Simulation Methodology

**Overhead Cost Pools** ($9.2B total):
- Assembly (26%): Based on auto industry labour content
- Battery Systems (35%): Tesla's largest cost component
- Quality Testing (9%): Industry standard for EV testing
- Logistics (13%): Based on Tesla's delivery model (no dealers)
- Facilities (17%): Property and equipment costs

**Activity Drivers**:
- Assembly hours: Estimated from manufacturing complexity (Model S/X take 50-100% longer than Model 3/Y)
- Battery kWh: From published vehicle specifications
- Test cycles: Premium vehicles (S/X) require more extensive testing

**Battery Costs** (`battery_make_vs_buy.csv` — **cell / variable level**):
- The $70/kWh (in-house) and $90/kWh (Panasonic) figures are **cell-level variable cost**, deliberately below the ~$115–140/kWh **pack-level** all-in price (BloombergNEF 2024): pack assembly, module housing, BMS and the $800m/yr line fixed cost sit *on top* of the cell.
- Tesla's vertical integration shows up as the ~20% in-house variable advantage ($70 vs $90/kWh).
- Panasonic pricing reflects typical supplier cell quotes.
- **Do not compare the $70/$90 cell figures directly to a $140 pack price** — they measure different scopes.

---

## Cross-Reference Validation

### Production Totals Check

| Year | Quarterly Sum | Annual Reported | Match |
|------|---------------|-----------------|-------|
| 2023 | 1,845,985 | 1,845,985 | ✓ |
| 2024 | 1,773,443 | 1,773,443 | ✓ |
| 2025 | 1,654,667 | (in progress) | — |

### Financial Totals Check (2024)

| Metric | Quarterly Sum | Annual Reported | Difference |
|--------|---------------|-----------------|------------|
| Revenue | $97.69B | $97.69B | $0.00B |
| Cost of Revenue | $74.87B | $80.24B | $5.37B (Compustat convention — see Data Quality Note 3) |

The COGS gap is Tesla's FY2024 depreciation, amortisation and impairment ($5,368M), which Compustat excludes from COGS but the 10-K includes in cost of revenues.

---

## Usage by Session

| Session | Primary Data | Notes |
|---------|--------------|-------|
| **S1: Cost Management** | `tesla_annual_2024.csv` | Real income statement data |
| **S2: Cost Estimation** | `tesla_quarterly.csv` | 8 quarters (2023–24) with complete financials for regression; 2025 estimated/partial |
| **S3: Cost Allocation** | `overhead_cost_pools.csv`, `product_activity_drivers.csv` | Simulated internal data |
| **S4: Decision Making** | `battery_make_vs_buy.csv` | Simulated supplier/internal costs |
| **S5: Variance Analysis** | `variance_analysis_q4.csv`, `tesla_targets.csv` | Mix of real actuals + simulated standards |
| **S6: Integration & Review** | None (pen-and-paper) | Pure consolidation — five-tool drills, past-exam practice, transfer test; no new data |

---

## Pedagogical Notes

### Why Simulated Data?

Tesla (like most companies) does not publicly disclose:
- Detailed cost breakdowns by component
- Internal transfer prices
- Activity-based cost pools
- Standard costs for variance analysis

This teaches students an important lesson: **internal management accounting data is proprietary**. Analysts must often estimate internal costs from external information.

### Transparency Principle

All materials clearly distinguish:
- **REAL**: Data from SEC filings with citations
- **SIMULATED**: Data based on industry benchmarks with methodology

Students learn to be critical consumers of cost data and understand its limitations.

---

## Data Dictionary

### tesla_quarterly.csv

| Column | Type | Description |
|--------|------|-------------|
| quarter | string | Q1, Q2, Q3, Q4 |
| year | integer | Calendar year |
| revenue_millions | float | Total revenue in $m |
| cogs_millions | float | Compustat COGS in $m — cost of revenues **excluding** D&A; does not tie to the 10-K's total cost of revenues ($80.24B) — see Data Quality Note 3 |
| sga_millions | float | SG&A **including R&D** (Compustat XSGA convention) in $m — do not add rd_millions to this column |
| rd_millions | float | Research & development expense in $m |
| production | integer | Vehicles produced |
| deliveries | integer | Vehicles delivered (units sold) |
| source | string | Data source (Compustat / Tesla IR) |
| notes | string | Data quality notes |

> **Complete financial quarters:** 8 (Q1 2023 – Q4 2024). 2025 quarters carry production/deliveries; 2025 financials are estimated and Q4 2025 financials are not yet reported. Use the 2023–24 quarters for the Session 2 regression.

### tesla_annual_2024.csv

| Column | Type | Description |
|--------|------|-------------|
| item | string | Income-statement line item |
| amount_millions | float | Amount in $m |
| source | string | Source (SEC 10-K FY2024 / derived) |
| notes | string | Notes |

### product_activity_drivers.csv

| Column | Type | Description |
|--------|------|-------------|
| model | string | Vehicle model name |
| annual_volume_fremont | integer | Estimated Fremont production |
| assembly_hours_per_unit | float | Labour hours per vehicle |
| kwh_capacity | float | Battery capacity in kWh |
| test_cycles_per_unit | integer | QC test cycles |
| avg_price_usd | integer | Average selling price |
| direct_materials_per_unit | integer | Estimated materials cost |
| notes | string | Additional context |

---

## Version History

| Date | Change |
|------|--------|
| Jan 2026 | Initial data collection |
| Jan 2026 | Extended production_by_model to 2023-2024 (estimated) |
| Jan 2026 | Added notes columns for data quality transparency |
| Jan 2026 | Updated simulated volumes to Fremont-specific focus |

---

*Data compiled for AC101 Summer School 2026. For educational use only.*
