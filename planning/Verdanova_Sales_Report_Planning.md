# Verdanova Sales Dashboard Planning

## Purpose
This document defines the planned one-page executive dashboard for the connected Verdanova Sales semantic model.

## Report Type
Single-page executive summary dashboard.

## Audience
Leadership / management stakeholders who need to answer:
- Are sales on track?
- Are we hitting target?
- Which categories and regions are driving performance?
- How does marketing contribute to revenue?
- What needs attention?

## Design Direction
- Executive, clean, minimal, evidence-first
- High readability
- Low clutter
- Simple KPI cards with directional comparison
- One visual story across time, category, region, and marketing contribution

## Wireframe Alignment
The dashboard should follow the local visual reference in the project folder and use a three-part executive layout:
- left filter rail
- central KPI and chart content
- lower insight summary strip

## Page Layout

### Header
- Title: Sales & Marketing Overview
- Subtitle: Performance at a Glance
- Optional right-aligned text: Data as of <date>

### Filter Pane (left rail)
Suggested filters:
- Date range
- Year
- Month
- Region
- Category
- Customer
- Platform

### KPI Row (top row)
1. Total Sales
2. Sales Target
3. Target Achievement %
4. Total Profit
5. Total Orders
6. Marketing Revenue

Each KPI card should contain:
- value
- label
- variance or % change vs prior period
- small trend indicator
- clear formatting and contrast

### Main Chart Area
1. Sales vs Target over time
   - Line/column combo chart
   - X-axis: month/date
   - Y-axis: sales value
2. Sales by Category
   - Donut or ring chart
3. Sales by Region
   - Filled map or geographic chart
4. Top Products by Sales
   - Table
5. Marketing Revenue by Platform
   - Horizontal bar chart
6. Target Achievement by Region
   - Matrix or table with conditional formatting

### Bottom Insight Section
A compact insight strip or summary text area showing:
- top category
- best region
- lowest-performing segment
- highest platform contribution
- current target status

## Data Model Focus
Use the connected semantic model tables and fields:

### Sales
- OrderDate
- Quantity
- UnitPrice
- Discount
- LineAmount
- Region
- ProductID
- CustomerID
- Channel

### Product
- ProductName
- Category
- UnitPrice
- StandardCost

### Regions
- Region

### Customers
- CustomerName
- CustomerType

### Calendar
- Date
- Year
- MonthNumber
- MonthName

### Sales Target
- Region
- ProductCategory
- SalesTarget
- Year
- Month
- YearMonthKey

### Marketing Campaign
- CampaignDate
- Platform
- ProductCategory
- RevenueGenerated

## Measures to Define
These measures should be created or validated before report build:

- Total Sales
- Sales Target
- Target Achievement %
- Profit
- Total Orders
- Marketing Revenue
- Previous Period Sales
- Sales Delta
- Sales Growth %
- Product Sales Rank
- Contribution %

## Design Rules
- Use a limited, professional palette
- Keep a strong contrast between cards and page background
- Avoid clutter and redundant labels
- Use consistent spacing and chart sizing
- Distinguish actual vs target clearly
- Keep filter pane compact and readable
- Use conditional formatting to highlight underperforming areas

## Technical Notes
- Prefer the net revenue field based on the business definition of sales
- Do not mix gross and net sales in the same executive KPI unless intentional
- Ensure all time-based visuals use a consistent date dimension
- Keep target measures aligned to the same granularity as actual sales

## Build Sequence
1. Confirm KPI definitions and target logic
2. Validate the model fields and relationships
3. Create or confirm required measures
4. Build filter pane
5. Build KPI cards
6. Build monthly sales vs target trend visual
7. Build category and region views
8. Build product and marketing visuals
9. Add target comparison matrix
10. Validate page readability and spacing
11. Final review against the wireframe

## Success Criteria
The dashboard is successful if a manager can answer in under 10 seconds:
- Are sales above or below target?
- Which category is driving revenue?
- Which region is strongest or weakest?
- How much revenue did marketing contribute?
- Which products or segments deserve attention?

## Notes
This is a planning document only. It is not a report build and does not modify the semantic model.
