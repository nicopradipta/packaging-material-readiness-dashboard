# Data Dictionary — Packaging Readiness Portfolio Project

Synthetic dataset simulating 100 SKUs over 24 months (Jan 2024 – Dec 2025), weekly cadence,
designed to support the KPI set defined for the Packaging Material Readiness project.

## Tables

### 1. dim_product.csv (Dimension)
Grain: 1 row = 1 SKU

| Field | Type | Description |
|---|---|---|
| Product_ID | Text | SKU identifier (PK) |
| Product_Name | Text | Product name |
| Packaging_Category | Text | Small / Medium / Large |
| Units_Per_Box | Number | Packaging units required per finished box |

### 2. fact_packaging_order.csv (Fact — event grain)
Grain: 1 row = 1 packaging purchase order event

| Field | Type | Description |
|---|---|---|
| PO_ID | Text | Purchase order identifier |
| Product_ID | Text | FK to dim_product |
| Order_Date | Date | Date PO was placed |
| Promised_ETA_Date | Date | Vendor-promised arrival date |
| Actual_Arrival_Date | Date | Actual arrival date (blank = still in transit / beyond data range) |
| Qty_Ordered | Number | Quantity ordered |
| Qty_Received | Number | Quantity received (blank if not yet arrived) |

### 3. fact_packaging_stock.csv (Fact — weekly snapshot grain)
Grain: 1 row = packaging stock condition for 1 SKU on 1 Wednesday snapshot date

| Field | Type | Description |
|---|---|---|
| Snapshot_Date | Date | Weekly snapshot date (Wednesdays) |
| Product_ID | Text | FK to dim_product |
| Stock_On_Hand | Number | Packaging stock on hand at snapshot |
| Qty_Used | Number | Packaging quantity used that week |

### 4. fact_fg_stock.csv (Fact — weekly snapshot grain)
Grain: 1 row = finished goods stock condition for 1 SKU on 1 Wednesday snapshot date

| Field | Type | Description |
|---|---|---|
| Snapshot_Date | Date | Weekly snapshot date |
| Product_ID | Text | FK to dim_product |
| FG_Stock_On_Hand | Number | Finished goods stock on hand |
| FG_Weekly_Shipment | Number | Finished goods shipped that week |

### 5. fact_production_incident.csv (Fact — event grain)
Grain: 1 row = 1 production stoppage incident caused by packaging shortage

| Field | Type | Description |
|---|---|---|
| Incident_ID | Text | Incident identifier |
| Product_ID | Text | FK to dim_product |
| Start_Date | Date | Date production stopped |
| End_Date | Date | Date production resumed |
| Duration_Days | Number | Length of stoppage in days |
| Root_Cause | Text | Cause of the incident |

---

## Known Data Quality Issues (intentional — for Data Cleaning practice)

These were injected deliberately. Do not treat this as an exhaustive list — part of the
exercise is discovering issues through profiling, not just reading this table.

- **dim_product**: `Packaging_Category` has inconsistent casing/whitespace on some rows
  (e.g. `"small "`, `"MEDIUM"`).
- **fact_packaging_order**: contains duplicate rows, a few negative `Qty_Ordered` values
  (data entry errors), `Order_Date` stored in mixed date formats, some missing
  `Promised_ETA_Date`, and stray whitespace around some `Product_ID` values.
- **fact_packaging_stock**: contains duplicate rows, a few negative `Stock_On_Hand` values
  (physically impossible — data entry errors), and a small number of missing weekly
  snapshots per SKU.
- **fact_fg_stock**: contains duplicate rows and a small number of missing weekly snapshots.

## Design Notes (for interview prep)

- Snapshot grain is **weekly (Wednesday)**, matching the real business planning cadence —
  not daily, since daily granularity would exceed what the actual decision-making process uses.
- `Fact_PackagingOrder` and `Fact_PackagingStock` are separate fact tables because they have
  different grains: event-based vs. time-snapshot-based.
- `Dim_Packaging` was merged into `Dim_Product` because the relationship is 1:1 (unnecessary
  snowflaking was avoided).
- Seasonal delay pattern is intentional: Medium packaging shows higher delay in Q1, Large
  packaging in Q2, repeating in both 2024 and 2025 — this is the pattern your dashboard should
  be able to surface.
