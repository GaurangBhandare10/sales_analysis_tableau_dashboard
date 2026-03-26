# Sales & Customer Dashboards (Dynamic)

A Tableau workbook providing interactive, year-over-year analysis of sales performance and customer behavior. The dashboards are built around a dynamic year parameter so stakeholders can instantly compare any year against the prior year without modifying the workbook.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Dashboards](#dashboards)
3. [Worksheets](#worksheets)
4. [Data Sources & Schema](#data-sources--schema)
5. [Data Model & Relationships](#data-model--relationships)
6. [Calculated Fields & KPIs](#calculated-fields--kpis)
7. [Parameters](#parameters)
8. [Dashboard Actions & Interactivity](#dashboard-actions--interactivity)
9. [File Structure](#file-structure)
10. [Prerequisites & Opening the Workbook](#prerequisites--opening-the-workbook)
11. [Reconnecting to Live Data](#reconnecting-to-live-data)
12. [Known Limitations](#known-limitations)

---

## Project Overview

| Attribute        | Value                                      |
|------------------|--------------------------------------------|
| File name        | `Sales & Customer Dashboards (Dynamic).twbx` |
| File type        | Tableau Packaged Workbook (`.twbx`)        |
| Tableau version  | 2023.2 (source), last saved in 2025.1      |
| Workbook version | 18.1                                       |
| Revision         | 3.0                                        |
| Data coverage    | Orders placed in 2020 – 2023               |
| Default year     | 2023                                       |
| Currency / Locale | USD display, `en_DE` locale (decimal `,`, thousands `.`) |

The workbook contains **two fully interactive dashboards** backed by **15 worksheets** and a single federated data source joining four CSV files.

---

## Dashboards

### 1. Sales Dashboard *(default view)*

Tracks high-level sales performance for the selected year vs. the prior year. Gives a quick pulse on whether the business is growing, which months are outliers, and which product sub-categories are driving or dragging performance.

**Key panels:**
- KPI tiles for **Total Sales**, **Total Profit**, **Total Quantity**, and **Total Orders** — each showing current-year value, prior-year comparison, and a colored trend indicator (▲/▼)
- **Weekly Trends** sparkline — highlights weeks that are above/below the average and marks the min/max weeks
- **Subcategory Comparison** bar chart — ranks product sub-categories by CY sales vs. PY sales side by side

### 2. Customer Dashboard

Focuses on customer-level metrics. Helps identify top customers, understand purchasing frequency, and spot shifts in the customer base.

**Key panels:**
- KPI tiles for **Total Customers**, **Total Sales per Customer**, **Total Orders per Customer**
- **Customer Distribution** — shows how customers are spread across order-frequency buckets
- **Top 10 Customers** table — ranked by sales, showing individual KPIs and trend indicators per customer

---

## Worksheets

Each worksheet feeds one or more panels in the dashboards above.

| Worksheet | Purpose |
|---|---|
| `KPI Sales` | Single-value tile: CY Sales vs. PY Sales with % diff |
| `KPI Profit` | Single-value tile: CY Profit vs. PY Profit with % diff |
| `KPI Quantity` | Single-value tile: CY Quantity vs. PY Quantity with % diff |
| `KPI Orders` | Single-value tile: CY Orders vs. PY Orders with % diff |
| `KPI Customers` | Single-value tile: CY Customers vs. PY Customers with % diff |
| `KPI Sales Per Customers` | Single-value tile: CY Sales per Customer vs. PY |
| `Legend KPI` | Color legend for KPI trend arrows |
| `Weekly Trends` | Line/bar chart of weekly CY sales; highlights above/below-average weeks and min/max |
| `Subcategory Comparison` | Horizontal bar chart comparing CY vs. PY sales by sub-category |
| `Legend Subcategory` | Color legend for the sub-category comparison bars |
| `Customer Distribution` | Distribution histogram of customers by order frequency |
| `Top Customers` | Top 10 customers ranked by CY sales |
| `Test KPI` | Development/prototype sheet for KPI layout testing |
| `Test KPI2` | Development/prototype sheet (secondary KPI variant) |
| `Test Max Min` | Development/prototype sheet for min/max calculation testing |

> **Note:** `Test KPI`, `Test KPI2`, and `Test Max Min` are development worksheets retained in the workbook. They are not embedded in either published dashboard but can be reviewed for reference.

---

## Data Sources & Schema

The workbook uses a **single federated data source** named `Sales DataSource`, which joins four semicolon-delimited CSV files via an inner-join on shared key columns.

Original file path on the author's machine:
```
C:/Tableau Data/TableauProject-SalesDashboard/
```

A `.hyper` extract is packaged inside the `.twbx`, so the workbook is self-contained and can be opened without the original CSVs.

### Orders.csv *(fact table)*

| Column | Type | Description |
|---|---|---|
| Row ID | Integer | Unique row identifier |
| Order ID | String | Unique order identifier |
| Order Date | Date | Date the order was placed |
| Ship Date | Date | Date the order was shipped |
| Ship Mode | String | Shipping method (e.g., Standard Class, First Class) |
| Customer ID | String | Foreign key → Customers.csv |
| Segment | String | Customer segment (Consumer, Corporate, Home Office) |
| Postal Code | Integer | Foreign key → Location.csv |
| Product ID | String | Foreign key → Products.csv |
| Sales | Real | Revenue for the line item |
| Quantity | Integer | Units ordered |
| Discount | Real | Fractional discount applied (0.0 – 1.0) |
| Profit | Real | Profit for the line item |

### Customers.csv *(dimension)*

| Column | Type | Description |
|---|---|---|
| Customer ID | String | Primary key |
| Customer Name | String | Full name of the customer |

### Location.csv *(dimension)*

| Column | Type | Description |
|---|---|---|
| Postal Code | Integer | Primary key |
| City | String | City name |
| State | String | State name |
| Region | String | Geographic region (East, West, Central, South) |
| Country/Region | String | Country |

### Products.csv *(dimension)*

| Column | Type | Description |
|---|---|---|
| Product ID | String | Primary key |
| Category | String | Top-level product category (Furniture, Office Supplies, Technology) |
| Sub-Category | String | Product sub-category (e.g., Chairs, Binders, Phones) |
| Product Name | String | Full product name |

---

## Data Model & Relationships

```
Orders.csv
  ├── Customer ID  ──────►  Customers.csv (Customer ID)
  ├── Postal Code  ──────►  Location.csv  (Postal Code)
  └── Product ID   ──────►  Products.csv  (Product ID)
```

All joins are performed inside Tableau's logical layer as a **federated connection** (not a data blend). The joined result is extracted into a `.hyper` file stored inside the `.twbx` package, making the workbook fully portable.

---

## Calculated Fields & KPIs

All KPI calculations follow a consistent **Current Year (CY) / Prior Year (PY)** pattern driven by the `Select Year` parameter.

### Core Pattern

```tableau
-- Current Year Sales
IF YEAR([Order Date]) = [Select Year] THEN [Sales] END

-- Prior Year Sales
IF YEAR([Order Date]) = [Select Year] - 1 THEN [Sales] END
```

The same pattern is applied to Profit, Quantity, Orders (COUNTD of Order ID), and Customers (COUNTD of Customer ID).

### Percentage Difference

```tableau
-- % Diff Sales
( SUM([CY Sales]) - SUM([PY Sales]) ) / SUM([PY Sales])
```

Format applied: `*▲ 0.0%; ▼ -0.0%;` — displays a filled triangle prefix so positive and negative values are visually distinct without custom icons.

### Sales per Customer

```tableau
-- CY Sales per Customer
SUM([CY Sales]) / COUNTD([CY Customers])
```

### KPI Status Indicators

Two table-calculation-based string fields produce above/below-average and min/max highlighting:

```tableau
-- KPI Sales Avg (Above / Below average line)
IF SUM([CY Sales]) > WINDOW_AVG(SUM([CY Sales]))
THEN 'Above'
ELSE 'Below'
END

-- Min/Max Sales (highlights only the min and max bar)
IF SUM([CY Sales]) = WINDOW_MAX(SUM([CY Sales]))
THEN SUM([CY Sales])
ELSEIF SUM([CY Sales]) = WINDOW_MIN(SUM([CY Sales]))
THEN SUM([CY Sales])
END

-- KPI dot indicator (CY < PY signals a down period)
IF SUM([CY Sales]) < SUM([PY Sales]) THEN '⬤' ELSE '' END
```

The same Min/Max and Above/Below patterns are replicated for Profit, Quantity, Customers, Orders, and Sales per Customer.

### LOD Expression

```tableau
-- Total CY Sales (independent of viz filters, used for reference lines)
{ SUM([CY Sales]) }

-- Orders per Customer (customer-level FIXED aggregation)
{ FIXED [CY Customers] : COUNTD([CY Orders]) }
```

### Helper Fields

| Field | Formula | Purpose |
|---|---|---|
| Current Year | `[Select Year]` | Displays the selected year as a label |
| Previous Year | `[Select Year] - 1` | Displays the prior year as a label |
| Order Date (Year) | `YEAR([Order Date])` | Used for axis filtering |

---

## Parameters

| Parameter | Caption | Type | Default | Domain |
|---|---|---|---|---|
| `[Parameter 1]` | Select Year | Integer | 2023 | 2020, 2021, 2022, 2023 |

The `Select Year` parameter drives every CY/PY calculated field in the workbook. Changing it instantly updates all KPI tiles, trends, and comparison charts across both dashboards without any additional filters.

---

## Dashboard Actions & Interactivity

Four filter actions are defined on the dashboards:

| Action | Behavior |
|---|---|
| Product Based | To filter based on product catgory and subcategory|
| Location Based | To filter based on region, state & city |

---


## Known Limitations

- **Year range is fixed at 2020–2023.** To add a new year to the `Select Year` parameter, edit the parameter in Tableau Desktop and add the new year value to the list.
- **No geographic map view.** Although `City`, `State`, `Region`, and geographic semantic roles are assigned to location fields, neither dashboard includes a map visualization.
- **Test worksheets are included.** `Test KPI`, `Test KPI2`, and `Test Max Min` are development artifacts. They do not affect dashboard output but can be hidden or deleted before publishing to Tableau Server/Cloud.
- **Locale sensitivity.** The CSV files were authored with a European decimal format (`en_DE`). If you replace the source files with CSVs exported in a different locale, the numeric columns in `Orders.csv` may parse incorrectly. Verify the delimiter and decimal character settings in the data source connection dialog.
