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

**2. XBRL tag changes** — Apple switched revenue tags in 2019. An analyst pulling only `Revenues` gets data through 2018; the rest is blank. The completeness check catches this before it causes silent data gaps.

**3. Refilings** — Companies refile quarters when correcting errors. EDGAR keeps all versions. The duplicate detection check identifies conflicting entries and flags them for manual resolution.

---

## How to Run

```bash
pip install requests pandas openpyxl
jupyter notebook financial_data_quality_audit.ipynb
```

Runtime: ~3–4 minutes (live SEC EDGAR API calls). Update the `User-Agent` header with your name/email per [SEC EDGAR API guidelines](https://www.sec.gov/os/accessing-edgar-data).

---

## Project Structure

```
financial_data_quality_audit/
├── financial_data_quality_audit.ipynb   # Main audit notebook (13 sections)
├── Financial_Data_Quality_Audit.xlsx    # Sample output — 4-sheet audit report
└── README.md
```

---

## Relationship to Other Projects

This audit is **Step 1** in a two-project sequence:

1. **Financial Data Quality Audit** ← this project — validate the data
2. **[SEC EDGAR Financial Dashboard](https://github.com/chooseyourtacoday/SEC-EDGAR-Financial-Analysis)** — analyze the clean data

Running the audit first means every number in the dashboard has been verified. That's the difference between an analyst who delivers results and one who delivers *reliable* results.

---

## Author

**Alejandro (Alex)**  
GitHub: [@chooseyourtacoday](https://github.com/chooseyourtacoday)  
Location: Dallas–Fort Worth, TX  
Certifications: Codecademy BI Data Analyst (SQL · Python · Tableau · Power BI)

*Part of a portfolio targeting Data Analyst roles in financial data, BI, and analytics.*
