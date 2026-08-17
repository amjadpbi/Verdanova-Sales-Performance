<img width="1037" height="600" alt="image" src="https://github.com/user-attachments/assets/789e032c-08dc-4580-9d91-fb303de03bef" />




# Verdanova Sales Performance

An end-to-end **Microsoft Fabric Lakehouse** project for transforming multi-source operational data into a validated analytical model and Power BI report.

The project was developed from a Scenario Lab business case and implemented as a hands-on Fabric solution using synthetic operational data, Medallion architecture, PySpark notebooks, a Direct Lake semantic model, and Power BI reporting.

---

## Project Overview

**Verdanova Consumer Products Ltd.** operates across multiple business systems that produce customer, product, sales, marketing, and planning data.

The objective of this project was to build a scalable analytical foundation that brings these sources together and supports sales-performance analysis.

The solution follows:

**Operational Sources → Bronze → Silver → Gold → Semantic Model → Power BI Report**

The project deliberately includes data-quality problems in the generated source data so that profiling, cleansing, and validation are part of the implementation rather than being assumed to be perfect from the beginning.

---

## Business Scenario

The project started from a Scenario Lab exercise focused on:

- sales performance analysis
- multi-source data ingestion
- Lakehouse architecture
- Medallion design
- analytical performance
- scalable data processing

The scenario was then adapted into the **Verdanova** implementation used in this repository.

The source systems were designed before implementation to represent a realistic operational environment.

### Operational Sources

| Source System | Business Data |
|---|---|
| CRM | Customers, Sales Representatives |
| ERP | Products, Orders, Order Lines |
| Excel | Sales Targets |
| Marketing | Campaign Performance |

Each source system owns its operational entities. The generated data is periodically exported and then ingested into Microsoft Fabric.

---

## Architecture

```text
CRM ───────────────┐
ERP ───────────────┤
Excel ─────────────┼──→ Lakehouse Files → Bronze
Marketing ─────────┘                       │
                                           ↓
                                         Silver
                                           │
                                           ↓
                                          Gold
                                           │
                                           ↓
                                   Semantic Model
                                           │
                                           ↓
                                      Power BI
```

### Medallion Layers

#### Bronze

Raw synthetic operational data is stored as Delta tables.

Examples:

- `bronze_customers`
- `bronze_products`
- `bronze_orders`
- `bronze_order_lines`
- `bronze_campaign_performance`
- `bronze_sales_targets`

The Bronze layer preserves the source structure before analytical transformations.

#### Silver

The Silver layer cleans and standardizes the Bronze data.

Typical operations include:

- duplicate removal
- data-type conversion
- categorical standardization
- missing-value handling
- normalization of source values
- resolution of identified data-quality issues

The Silver layer is driven by findings from the profiling stage rather than by arbitrary transformations.

#### Gold

The Gold layer provides analytical dimensions and fact/business tables:

**Dimensions**

- `Calendar`
- `Product`
- `Customer`
- `Region`

**Facts / business tables**

- `Sales`
- `MarketingCampaign`
- `SalesTarget`

The Gold layer is designed for analytical consumption and semantic-model development.

---

## Notebook Workflow

The five notebooks form a deliberate learning and engineering sequence.

### Notebook 01 — Generate Operational Source Data

Generates synthetic data for the operational source systems, writes raw exports to Lakehouse Files, and loads the data into Bronze Delta tables.

Intentional data-quality issues are included in the generated data.

### Notebook 02 — Profile Operational Data

Profiles the Bronze layer before transformation.

The notebook checks areas such as:

- record counts
- missing values
- duplicates
- distinct values
- product pricing relationships
- other data-quality observations

The findings provide the basis for Silver-layer cleansing.

### Notebook 03 — Transform Operational Data to Silver

Reads the Bronze tables and applies the cleansing and standardization rules identified during profiling.

Examples include:

- removing duplicate records
- standardizing casing
- handling missing values
- correcting known data-quality issues
- converting columns to appropriate data types

### Notebook 04 — Build Gold Analytical Model

Transforms Silver data into analytical dimensions and fact/business tables.

The notebook builds:

- Calendar
- Customer
- Product
- Region
- Sales
- MarketingCampaign
- SalesTarget

It also contains targeted refinements required for analytical use.

### Notebook 05 — Gold Validation

Validates the completed Gold layer before analytical consumption.

Validation covers:

- table and key integrity
- referential integrity
- duplicate business grain
- sales business rules
- campaign business rules
- sales-target relationships
- cross-table consistency

The project follows the cycle:

**Build → Question → Validate → Find Issue → Understand → Fix → Validate Again**

---

## Data Quality by Design

The project intentionally introduces data-quality problems into the source data.

Examples include:

- missing customer information
- inconsistent region casing
- duplicate records
- missing product category
- inconsistent product-category casing
- missing marketing platform
- duplicate campaign records
- invalid campaign conversion relationships
- sales-target grain issues

This creates a realistic engineering workflow:

```text
Intentional Source Issue
        ↓
Bronze
        ↓
Profile
        ↓
Identify Problem
        ↓
Silver Transformation / Source Regeneration
        ↓
Gold
        ↓
Validation
```

This was one of the main learning objectives of the project.

---

## Important Data Decisions

### Product Pricing

The initial product generator produced an unrealistic relationship between `StandardCost` and `UnitPrice`.

The data was profiled, the distribution was reconsidered, and the generation logic was refined before continuing through the pipeline.

This demonstrates why synthetic data should still be tested for business plausibility.

### Marketing Revenue

Marketing campaign revenue is intentionally treated as an **independent marketing metric** rather than being forced to reconcile with Sales revenue.

The project therefore does not fabricate equality between the two datasets simply to make the dashboard look cleaner.

### Sales Targets

The intended SalesTarget grain is:

**Year + Month + Region + Product Category**

The final validation confirmed **192 unique target combinations**:

**12 months × 4 regions × 4 product categories = 192**

### Marketing Conversion Validation

An initial validation found campaigns where conversions exceeded clicks.

The source-generation rule was corrected so that:

```python
conversions <= clicks
```

The affected data was regenerated and the validation was rerun.

---

## Semantic Model

The Gold layer feeds a **Direct Lake semantic model**.

The model brings together:

- Calendar
- Product
- Customer
- Region
- Sales
- MarketingCampaign
- SalesTarget

The Calendar table includes fields such as:

- Date
- Year
- Quarter
- Month Number
- Month Name
- Month Short
- YearMonthKey

The semantic model was refined to support correct month ordering and analytical filtering.

The report uses the semantic model rather than directly building visuals from the raw Bronze/Silver layers.

---

## Power BI Report

The final report presents the Verdanova sales-performance view for the primary **2025 reporting period**.

It includes:

- Total Sales
- Profit
- Total Orders
- Target Achievement
- Monthly Sales & Profit Trend
- Actual vs Target Sales
- Sales by Region
- Sales by Product Category
- Marketing Revenue by Platform

Interactive filters include:

- Date
- Region
- Product Category

The report was refined after validating the underlying data and semantic model, including removal of irrelevant 2026 reporting noise from the main analysis.

---

## Source Data Design

The source-system design was planned before the Fabric implementation.

```text
CRM
├── Customers
└── Sales Representatives

ERP
├── Products
├── Orders
└── Order Lines

Excel
└── Sales Targets

Marketing
└── Campaign Performance
```

This separation establishes clear ownership of operational entities before they enter the Lakehouse.

---

## Validation Approach

Validation was not limited to checking whether tables were created successfully.

The Gold layer was tested at several levels:

### Structural Validation

- expected tables exist
- expected keys are present
- duplicate business-grain combinations are checked

### Referential Validation

Relationships between fact and dimension data are checked using referential-integrity tests.

### Business Validation

Sales calculations are checked against expected business rules, including:

- quantity
- unit price
- cost
- discount
- line amount

Marketing campaign values are also checked against their business constraints.

### Cross-Fact Validation

Sales, SalesTarget, and MarketingCampaign are compared where meaningful without assuming that independent business metrics must reconcile.

---

## Project Assets

The repository contains the main artifacts used to design, build, validate, and present the solution.

```text
/
├── Notebooks/
│   ├── Notebook 01 — Generate Operational Source Data
│   ├── Notebook 02 – Profile Operational Data
│   ├── Notebook 03 – Transform Operational Data to Silver
│   ├── Notebook 04 – Build Gold Analytical Model
│   └── Notebook_5_Gold_Validation
│
├── PBIP/
│   └── Power BI project and semantic model definitions
│
├── Planning/
│   ├── Source system planning
│   └── Report planning
│
├── Documentation/
│   ├── Notebook audit report
│   └── Scenario / project planning material
│
└── README.md
```

*Folder names can be adjusted to match the final repository structure.*

---

## What This Project Demonstrates

This project demonstrates practical experience with:

- Microsoft Fabric Lakehouse
- Medallion architecture
- Bronze / Silver / Gold design
- PySpark data transformation
- Delta tables
- Lakehouse Files
- Data-quality profiling
- Data-quality remediation
- Analytical fact and dimension design
- Gold-layer validation
- Semantic model design
- Direct Lake
- Power BI reporting
- Business-rule validation
- Iterative debugging and refinement

More importantly, the project demonstrates the process of **building, testing, finding problems, understanding them, and refining the solution** rather than treating a generated dataset as automatically correct.

---

## Learning Approach

This was built as a hands-on learning project.

The implementation was not treated as a one-pass exercise. Data was inspected after generation, assumptions were challenged, validation exposed issues, and the affected layers were refined before continuing.

The notebooks were also reviewed and standardized after implementation to improve their readability and make the engineering decisions easier to follow.

The goal was not to produce a perfect-looking project from the first cell.

The goal was to understand how an end-to-end Fabric solution is actually built.

---

## Current Scope

The implemented solution covers:

**Operational source simulation → Lakehouse ingestion → Bronze → Silver → Gold → Gold validation → Semantic model → Power BI report**

The project uses synthetic data and is intended as a practical Fabric learning and portfolio project rather than a production deployment.

Production concerns such as enterprise workspace security, CI/CD automation, real-time ingestion, and production-scale performance testing are outside the implemented scope unless explicitly documented elsewhere in the repository.

---

## Project Outcome

The final result is an end-to-end Microsoft Fabric sales analytics solution that starts with multiple simulated operational systems and finishes with a validated analytical model and interactive Power BI report.

The project combines **data engineering, data modeling, validation, and reporting** in one workflow rather than treating the dashboard as an isolated deliverable.

## Development Workflow

The project was developed using a combination of Fabric-native development and AI-assisted tooling.

### Tools and Practices

- **Microsoft Fabric** — Lakehouse, notebooks, Delta tables, semantic model, and Direct Lake reporting.
- **Fabric skills** — used to support development and investigation of Fabric-specific implementation details.
- **Claude Code** — used to audit notebook structure and documentation and help identify inconsistencies without changing the project logic blindly.
- **Microsoft Power BI MCP** — used to work with and inspect Power BI-related project components.
- **TMDL** — used to inspect and edit semantic-model metadata directly.

A key part of the workflow was **verify rather than blindly accept**. Changes were tested in the actual Fabric environment to observe their effect.

For example, when adding `MonthShort` to the Calendar table, the TMDL definition was updated directly, the model was refreshed, and the resulting behavior was checked in the report. This provided practical experience with how semantic-model metadata changes affect the reporting layer.
