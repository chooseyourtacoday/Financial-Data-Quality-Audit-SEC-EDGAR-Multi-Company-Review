# Financial Data Quality Audit — SEC EDGAR Multi-Company Review

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas)
![Excel](https://img.shields.io/badge/Excel-Audit%20Report-217346?logo=microsoft-excel)
![Data Source](https://img.shields.io/badge/Data%20Source-SEC%20EDGAR%20API-red)

---

## Overview

Most financial analysis projects start with the data. This one starts *before* the data — auditing it first.

This project runs **7 systematic data quality checks** against SEC EDGAR XBRL filings for Apple, Microsoft, Tesla, Amazon, and Johnson & Johnson before any analysis begins. Every issue is logged with severity rating, evidence, and remediation guidance, then exported to a 4-sheet Excel audit report.

This is the workflow used in professional financial data roles: **audit → document → remediate → analyze.**

---

## The 7 Audit Checks

| # | Category | What It Catches |
|---|---|---|
| 1 | **Completeness** | Missing quarters, null fields, expected vs. actual row counts |
| 2 | **Mathematical Consistency** | Gross Profit > Revenue? Margins outside valid range? |
| 3 | **Cross-Filing Reconciliation** | Do quarterly sums match the annual 10-K figure? |
| 4 | **Temporal Continuity** | Gaps between filings, out-of-sequence periods |
| 5 | **Anomaly Detection** | Values beyond 3σ, QoQ swings >50%, negative revenue |
| 6 | **XBRL Tag Consistency** | Same metric reported under different tags across years |
| 7 | **Duplicate Detection** | Same period filed more than once — identical or conflicting |

---

## Excel Audit Report — 4 Sheets

| Sheet | Contents |
|---|---|
| **Executive Summary** | Per-company Data Quality Score (0–100), issue counts by severity, overall audit snapshot |
| **Issue Log** | Every flagged issue — color-coded by severity, sortable, with expected vs. actual values |
| **Check Summary** | Pivot by check category — which checks passed, which flagged issues |
| **Remediation Tracker** | Critical & High issues only — Status, Owner, Due Date fields for workflow management |

**Severity levels:**
- 🔴 **Critical** — data is unusable as-is; corrupts downstream analysis
- 🟠 **High** — significant anomaly; investigate before use
- 🔵 **Medium** — notable irregularity; use with documented caveats
- 🟢 **Low** — minor inconsistency; monitor

---

## Why These Checks Matter

Three issues occur frequently enough in EDGAR data to warrant systematic checking every time:

**1. Cumulative YTD vs. Quarterly values** — EDGAR returns both in the same endpoint. A Q3 YTD revenue of $90B looks like a strong quarter; the actual standalone Q3 figure might be $28B. This is the most common source of inflated revenue figures in EDGAR-pulled data.

**2. XBRL tag changes** — Apple switched revenue tags in 2019. An analyst pulling only `Revenues` gets data through 2018; later years are on `RevenueFromContractWithCustomerExcludingAssessedTax`. The completeness check flags a metric only when every candidate tag comes back empty, so a retired tag that still has old rows is not, by itself, a completeness failure. The saved run (2026-11-22) keeps the candidate with the latest 10-Q/10-K period end, so Apple, Microsoft, Amazon, and Johnson & Johnson are audited on that current tag. Tesla stays on `Revenues`, which is already current. See Limitations.

**3. Refilings** — Companies refile quarters when correcting errors. EDGAR keeps all versions. The duplicate detection check identifies conflicting entries and flags them for manual resolution.

---

## How to Run

```bash
pip install requests pandas openpyxl
jupyter notebook financial_data_quality_audit.ipynb
```

Runtime: ~3–4 minutes (live SEC EDGAR API calls). Update the `User-Agent` header with your name/email per the [SEC fair-access rules for EDGAR](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data).

---

## Project Structure

```
.
├── financial_data_quality_audit.ipynb   # Main audit notebook (13 sections)
├── Financial_Data_Quality_Audit.xlsx    # Sample output — 4-sheet audit report (run date 2026-11-22)
├── .gitignore                           # Ignores Jupyter checkpoint files
└── README.md
```

---

## Relationship to Other Projects

This audit is **Step 1** in a two-project sequence:

1. **Financial Data Quality Audit** ← this project — validate the data
2. **[SEC EDGAR Financial Dashboard](https://github.com/chooseyourtacoday/SEC-EDGAR-Financial-Analysis)** — analyze the clean data

Running the audit first means every number in the dashboard has been verified. That's the difference between an analyst who delivers results and one who delivers *reliable* results.

---

## Limitations

The Excel workbook and the outputs saved in the notebook are the run from **2026-11-22**. Ingestion on that run kept the revenue tag whose latest 10-Q/10-K period end is most recent. An earlier run on **2026-05-29** stopped at the first tag with any rows, logged **209** issues, and also scored every company **0/100**. The counts below are the 2026-11-22 re-run.

**Every company still scores 0/100, and that number is not a ranking of the filers.** The score starts at 100 and subtracts 10 points for each Critical issue, 5 for High, 2 for Medium, and 1 for Low, then floors at 0. This run logged **238** issues: 28 Critical, 193 High, and 17 Medium. **184 of them are Temporal Continuity, and all 184 are High.** Those temporal findings alone subtract more than 100 points for every company: 235 for Apple, 210 for Microsoft, 225 for Tesla, 135 for Amazon, and 115 for Johnson & Johnson. After the score hits 0, more issues do not change it. A 0 does not mean every check failed.

Issue counts this run: Apple 60, Microsoft 49, Tesla 57, Amazon 38, Johnson & Johnson 34. On 2026-05-29 those counts were 54, 37, 56, 31, and 31.

**Most of those temporal findings are one missing quarter, on a repeating schedule.** The check keeps 10-Q and 10-K facts whose `frame` value contains `Q`, drops duplicate period ends, and flags any gap longer than 130 days as High (the code comment says to expect about 90 days). Of the 184 gaps, 166 are between 131 and 200 days. 107 are exactly 182 days and 48 are exactly 183. That is the distance from one quarter-end to the quarter-end after next.

For Apple, fiscal Q4 ends in late September. From FY2021 the 10-K gross-profit fact on that date is the full year (about 363 days). There is no separate ~90-day fourth-quarter fact. Many year-end rows also have no calendar `frame`, so the filter drops them even in earlier years when a quarterly fact does exist. The kept series jumps from late June to late December, and the check logs a High issue for every metric. The same shape shows up for the other companies whose fiscal year does not end in December. Eighteen gaps are longer than 200 days, and 14 of those are longer than 400. Those 18 long gaps are the same ones as in the 2026-05-29 log. The 21 new temporal rows are all in the 131–200 day bucket, from revenue facts the first-tag pull did not include. They sit at the same severity as the long gaps, so the issue log does not separate them.

The 130-day cutoff and the point weights are unchanged, so the workbook still matches the check that produced it. A wider cutoff would also hide the long gaps, and different weights would be a different scorecard than the one saved in the workbook.

**Revenue tags in this run.** Candidates are still requested as `Revenues`, then `SalesRevenueNet`, then `RevenueFromContractWithCustomerExcludingAssessedTax`. The saved workbook uses the 2026-11-22 tag. The 2026-05-29 run used the other tag. Row counts are raw company-concept rows from each pull.

| Company | Tag audited on 2026-11-22 | Tag the 2026-05-29 run kept |
|---|---|---|
| Apple | `RevenueFromContractWithCustomerExcludingAssessedTax`, 120 rows, through 2026-09-26 | `Revenues`, 11 rows, through 2018-09-29 |
| Microsoft | `RevenueFromContractWithCustomerExcludingAssessedTax`, 136 rows, through 2026-09-30 | `Revenues`, 31 rows, through 2010-12-31 |
| Amazon | `RevenueFromContractWithCustomerExcludingAssessedTax`, 132 rows, through 2026-09-30 | `SalesRevenueNet`, 185 rows, through 2018-06-30 |
| Johnson & Johnson | `RevenueFromContractWithCustomerExcludingAssessedTax`, 132 rows, through 2026-09-27 | `Revenues`, 18 rows, through 2014-09-28 |
| Tesla | `Revenues`, 286 rows, through 2026-09-30 | `Revenues`, 278 rows (already the current tag) |

Category totals this run: Temporal Continuity 184, Cross-Filing Reconciliation 30, Anomaly Detection 18, XBRL Tag Consistency 5, Completeness 1. On 2026-05-29 those were 163, 22, 18, 5, and 1. Mathematical consistency and duplicate detection still logged nothing. The completeness check still only flags a metric when every candidate tag is empty, so it would not have caught the old first-tag behavior.

Row counts move as companies file. A later pull will not match the table exactly.

---

## Author

**Alejandro (Alex)**  
GitHub: [@chooseyourtacoday](https://github.com/chooseyourtacoday)  
Location: Dallas–Fort Worth, TX  
Certifications: Codecademy BI Data Analyst (SQL · Python · Tableau · Power BI)

*Part of a portfolio targeting Data Analyst roles in financial data, BI, and analytics.*
