Financial Data Quality Audit — SEC EDGAR Multi-Company Review
The notebook pulls the same 5 companies from the SEC EDGAR project, but instead of analyzing the financials, it audits the data itself — checking completeness, mathematical consistency, cross-filing reconciliation, anomaly detection, and XBRL tag reliability. 

Then it exports an Excel audit report with an issue log, severity ratings, and a data quality scorecard.

Completeness — missing quarters, null fields, expected vs actual row counts
Mathematical Consistency — does Revenue − COGS = Gross Profit? Do margins recalculate correctly?
Cross-filing Reconciliation — do Q1+Q2+Q3+Q4 sum to the annual 10-K figure?
Temporal Continuity — gaps in filing dates, out-of-sequence periods
Anomaly Detection — values outside 3σ, quarter-over-quarter swings >50%
XBRL Tag Consistency — same concept reported under different tags across years
Duplicate Detection — same period filed multiple times

The notebook — 13 sections, fully documented. Sections 4–10 are the seven audit checks, each one writing to the central issue log. The last section has the narrative explaining why auditing first is the right workflow, which is the thing that makes this project land differently than just "I cleaned some data."
The Excel report — 4 sheets with 15 realistic sample issues seeded across all five companies:

3 Critical (a YTD reconciliation failure on Apple, conflicting duplicate on Tesla, the J&J litigation charge that made net income go negative)
4 High (Amazon's supply chain QoQ swing, Microsoft's XBRL tag switch in 2019, Tesla missing quarters pre-2020, an Apple mathematical consistency flag)
6 Medium and 2 Low
