# Nexus Operations | Operational Performance Dashboard

**Power BI Portfolio Project | Business Intelligence · Operational Analytics · Data Quality**

> **Portfolio disclaimer:** Nexus Operations Services is a fictional business case. All operational records, team names, targets, and analytical findings in this project are simulated for demonstration purposes. No confidential company data was used. The recommendations are proposals, not interventions that were implemented or proven effective.

## Project Overview

Nexus Operations Services is a simulated business-process operations organization managing incoming requests across eight teams. Management needs a consistent view of throughput, productive hours, processing quality, rework, and progress toward operational goals.

This project develops an interactive Power BI dashboard that combines daily operational performance with monthly targets. It is designed to help managers compare teams, identify trade-offs between speed and accuracy, and determine where deeper investigation may be worthwhile.

### Business Questions

- Which teams meet their productivity goal without compromising quality?
- How do productivity, quality, error rate, and rework rate vary by team and month?
- Which teams warrant investigation, and what additional evidence is needed before allocating resources?
- Can management reliably filter and compare team performance using validated operational data?

## Tools and Skills

| Tool | Application |
| --- | --- |
| **Power BI Desktop** | Interactive report, data model, DAX measures, conditional formatting, team and month filters |
| **Power Query** | Data types, profiling, validation, duplicate checks, preparation for reporting |
| **Microsoft Excel** | Simulated source tables and initial data-quality checks |
| **DAX** | Operational KPI calculations and target achievement logic |

## Dataset and Data Model

The model covers **2025** and **eight fictional teams (Team A–H)**.

| Table | Grain | Purpose |
| --- | --- | --- |
| `DimTeams` | One row per team | Team identifiers and descriptive attributes |
| `DimDate` | One row per calendar date | Calendar fields used to filter and order months |
| `FactOperations` | One row per operating date and team | Received and processed requests, errors, rework, and productive hours |
| `FactTargets` | One row per month and team | Productivity, quality, and rework targets |

**Reported source validation:** `FactOperations` contains **2,088** daily team records and `FactTargets` contains **96** team-month records. The year has 365 calendar dates; the operational fact table reflects 261 weekdays per team.

### Relationships

All four relationships are active, **one-to-many**, with **single-direction filtering from dimension to fact**:

- `DimTeams[TeamID]` → `FactOperations[TeamID]`
- `DimTeams[TeamID]` → `FactTargets[TeamID]`
- `DimDate[Date]` → `FactOperations[Date]`
- `DimDate[Date]` → `FactTargets[TargetMonth]`

**Modeling note:** `TargetMonth` stores the first day of each month. As a result, target rows are most naturally interpreted under full-month or full-year date selections; arbitrary daily selections can exclude monthly target records. Monthly targets should be reviewed carefully if the report is extended to daily target comparisons.

## Business Rules and Data Validation

- `Errors` is a subset of `Processed`, not a separate addition to processed volume.
- `Reworked` counts previously erroneous requests requiring correction and does **not** represent new processed requests.
- `ProductiveHours` includes productive time associated with operations, including rework.
- Expected logical constraint: `0 ≤ Reworked ≤ Errors ≤ Processed`.
- Each `Date` + `TeamID` pair should be unique in the operational fact table.
- Each `TargetMonth` + `TeamID` pair should be unique in the targets table.

Validation work included checks for column types, missing or invalid values, invalid dates and targets, duplicate operational keys, duplicate target keys, team coverage, and monthly target coverage. The Power Query validation reported **2,088 valid operational rows and zero duplicated date/team keys**. The targets validation reported **96 records, 12 months per team, no missing targets, and no invalid dates or target values**.

These are source-data validation results, not evidence that an operational change improved business outcomes.

## Key Performance Indicators

| KPI | Definition | Target |
| --- | --- | --- |
| **Total Processed** | Total first-pass processed requests | Volume indicator; no fixed minimum |
| **Productivity** | Processed requests ÷ productive hours | ≥ **6.00** cases/hour |
| **Quality Rate** | (Processed − Errors) ÷ Processed | ≥ **97%** |
| **Error Rate** | Errors ÷ Processed | ≤ **3%** |
| **Rework Rate** | Reworked ÷ Processed | ≤ **2.5%** |
| **Target Achievement** | Productivity ÷ productivity target | ≥ **100%** |

Supporting measures include `Total Productive Hours`, `Total Errors`, `Total Reworked`, `Total Correct Cases`, `Productivity Target`, and a target-status indicator.

### Representative DAX

```dax
Total Processed = SUM(FactOperations[Processed])

Total Errors = SUM(FactOperations[Errors])

Total Reworked = SUM(FactOperations[Reworked])

Total Productive Hours = SUM(FactOperations[ProductiveHours])

Productivity = DIVIDE([Total Processed], [Total Productive Hours])

Quality Rate =
    DIVIDE([Total Processed] - [Total Errors], [Total Processed])

Error Rate = DIVIDE([Total Errors], [Total Processed])

Rework Rate = DIVIDE([Total Reworked], [Total Processed])

Total Correct Cases = [Total Processed] - [Total Errors]

Productivity Target = SELECTEDVALUE(FactTargets[ProductivityTarget])

Target Achievement = DIVIDE([Productivity], [Productivity Target])
```

> **Implementation note:** These snippets use commas as DAX argument separators. Depending on Power BI locale settings, semicolons may be required. `SELECTEDVALUE` returns a blank when multiple distinct targets exist in the current filter context; a more robust target-selection policy would be needed if team or monthly targets were varied in future datasets.

## Dashboard Features

- Executive KPI cards for processed volume, productivity, quality, errors, and rework.
- Interactive team and month slicers.
- Monthly performance trend analysis.
- Team-level comparison table with target status and conditional colors.
- A **correct cases vs. errors** pie chart whose segments partition total processed volume.
- A productivity-versus-quality scatter chart for comparing teams and evaluating performance trade-offs.
- Goal-aware formatting to distinguish compliance from underperformance.

**Suggested dashboard screenshot:** Add a screenshot to `images/dashboard-overview.png` and embed it here once the file is committed:

```markdown
![Nexus Operations dashboard](images/dashboard-overview.png)
```

## Business Insights

The following findings are from the **simulated dataset**. Figures reflect the reviewed dashboard state and should be rechecked against the saved Power BI file before publication, especially if source data or filters have changed.

### 1. Team A: High productivity does not guarantee high quality

**Finding:** Team A produces approximately **6.08 cases per productive hour**, exceeding the **6.00** goal, but its quality rate is only **90.06%**, below the **97%** quality goal. Its error and rework rates are **9.94%** and **7.23%**, respectively.

**Potential business impact:** Elevated errors and rework may consume capacity, increase processing costs, and reduce service reliability. The dataset does not establish their financial cost or root cause.

**Recommended next investigation:** Examine error categories, request complexity, workflow adherence, and training opportunities; track quality and rework while monitoring whether productivity is maintained.

### 2. Team B: Improve efficiency without sacrificing quality

**Finding:** Team B produces approximately **5.22 cases per productive hour**, below its **6.00** goal. Its quality rate is **97.52%**, error rate **2.48%**, and rework rate **1.77%**—all within their respective quality-related targets.

**Potential business impact:** A productivity gap may represent missed processing capacity, but raising output requirements without identifying constraints could jeopardize existing quality performance.

**Recommended next investigation:** Measure processing time by request type, idle or non-value-added time, queue delays, and system bottlenecks. Improving from **5.22** to **6.00** cases/hour requires approximately **14.94%** higher throughput per productive hour under comparable conditions.

### 3. Prioritization: Compare economic impact before allocating resources

**Finding:** In the reviewed annual team results, Team A recorded approximately **3,141 errors** and **2,285 reworked cases**, versus Team B's **675 errors** and **482 reworked cases**.

**Potential business impact:** Team A's quality issues may carry a substantial cost, while Team B's lower productivity may represent an opportunity cost. The model does **not** include unit economics, penalties, staff costs, or revenue by request.

**Recommendation:** Estimate the cost of errors and rework for Team A and compare it with the value of recoverable capacity in Team B. Prioritize an intervention using the estimated business impact, feasibility, and risk rather than a single KPI alone.

## Limitations and Future Improvements

- **Synthetic data:** Findings demonstrate analytical reasoning, not verified performance of a real organization.
- **No financial fields:** Savings, revenue uplift, return on investment, and realized efficiency improvements cannot be measured from the available dataset.
- **No root-cause attributes:** Request categories, handling times, error reasons, and queue delays are not provided; proposed causes require additional data collection.
- **Monthly target structure:** Target rows are keyed to month-start dates, requiring careful date-filter behavior.
- **No intervention results:** Recommendations have not been implemented or experimentally validated.
- Potential enhancements: request-level data, cost-to-serve estimates, target modeling independent of month-start dates, and before/after intervention tracking.

## How to Explore the Project

1. Open `Nexus_Operational_Performance.pbix` in Power BI Desktop.
2. If prompted, update the source path to `Nexus_Operations_Data.xlsx` using **Transform data → Data source settings**.
3. Refresh the dataset and check model relationships.
4. Use the team and month filters to compare KPIs.
5. Review the scatter chart and team table together to distinguish throughput from quality performance.

> The PBIX and Excel files are companion artifacts created in Power BI and Excel. They are **not embedded in this README** and must be added separately to the repository.

## Suggested Repository Structure

```text
nexus-operational-performance/
├── README.md
├── Nexus_Operational_Performance.pbix
├── Nexus_Operations_Data.xlsx
└── images/
    ├── dashboard-overview.png
    ├── team-a-analysis.png
    └── team-b-analysis.png
```

## Portfolio Focus

This project demonstrates data preparation and quality checks, dimensional modeling, DAX-based KPI definitions, interactive business intelligence reporting, and the ability to distinguish analytical evidence from proposed operational actions.

---

**Project type:** Individual portfolio project · **Data:** Synthetic · **Period:** 2025 · **Status:** Dashboard developed; portfolio packaging and final source-file verification pending.
