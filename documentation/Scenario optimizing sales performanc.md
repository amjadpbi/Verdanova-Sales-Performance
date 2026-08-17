# Optimizing Sales Performance with Microsoft Fabric Lakehouse

**Difficulty:** Intermediate  
**Estimated Time:** 4 Hours  
**Actual Time:** —

## Business Context
Verdanova Consumer Products Ltd., a leading manufacturer of eco-friendly products, wants to improve its sales performance analysis. Currently, the company uses a combination of Excel files and an on-premises data warehouse to analyze sales data, which is time-consuming and prone to errors. EcoCycle has decided to migrate its data to a Microsoft Fabric Lakehouse using the Medallion architecture to unlock faster insights and better decision-making. The sales data is expected to grow significantly in the next quarter, and the company needs a scalable solution to handle the increased data volume. The IT team is tasked with designing and implementing a Lakehouse solution to meet these requirements.

## Target Skills
- Implement performance improvements in queries and report visuals
- Ingest or access data as needed
- Implement workspace-level access controls

## Requirements
- Ingest sales data from various sources, including CRM systems, ERP systems, and social media platforms
- Implement performance improvements in queries and report visuals to reduce latency and improve user experience
- Implement workspace-level access controls to ensure that only authorized personnel can access sensitive sales data

## Technical Constraints
- Use the Microsoft Fabric Lakehouse with Medallion architecture (Bronze, Silver, Gold layers) using Delta tables
- Ensure data ingestion is done in real-time to support timely sales performance analysis
- Use existing Azure Active Directory (AAD) for identity and access management

## Deliverables
- A Lakehouse with a Bronze layer for data ingestion, a Silver layer for data transformation, and a Gold layer for data analysis
- A set of optimized queries and report visuals to support sales performance analysis
- A workspace with implemented access controls to restrict access to sensitive sales data

## Success Criteria
- Sales performance analysis queries return results within 5 seconds
- 99.9% of sales data is ingested in real-time
- Only authorized personnel can access sensitive sales data

## Architecture Decisions

### Decision 1
**Question:** What analytical grain should the sales fact table use?
**Decision:** Order-line grain (one row per product purchased).
**Reasoning:** Enables product-, customer-, category-, and region-level analysis while supporting future analytical requirements.

### Decision 2
**Question:** How much historical data should be migrated?
**Decision:** 24 months of historical data.
**Reasoning:** Supports year-over-year comparisons, seasonal analysis, and a realistic migration scenario.

### Decision 3
**Question:** How should business growth be represented in the dataset?
**Decision:** Gradual growth with accelerated expansion after onboarding a national retail partner.
**Reasoning:** Creates realistic historical trends and provides a meaningful basis for scalability and performance testing.

### Decision 4
**Question:** What business process should the social media source represent?
**Decision:** Marketing campaign performance.
**Reasoning:** Allows campaign effectiveness to be correlated with sales outcomes, directly supporting the business objective of sales performance analysis.

### Decision 5
**Question:** How should source systems be modeled for the project?
**Decision:** Simulate realistic operational systems instead of using public datasets.
**Reasoning:** Produces an end-to-end implementation that reflects a real client environment and demonstrates solution design skills rather than dataset reuse.