# Power Query Documentation

## Overview

A single flat CSV file (`marketing_data.csv`) is transformed into a **star schema** using Power Query (M). The raw data is split into **3 fact tables**, **4 dimension tables**, and **1 data quality table** — all from one source.

## Data Source

| Query | Purpose |
|---|---|
| `_DataSources` | Connects to the local `datasets/` folder using `Folder.Files()` |
| `Marketing_Data_Source` | Imports `marketing_data.csv`, promotes headers, and sets column types |
| `Marketing_Data_Dictionary` | Imports `marketing_data_dictionary.csv` for field reference |

## Fact Tables

### Fact_ProductSpend

Captures how much each customer spent per product category.

**Key Steps:**
1. Select spend columns (`MntWines`, `MntFruits`, `MntMeatProducts`, etc.) along with `ID` and `Dt_Customer`
2. **Unpivot** spend columns → creates one row per customer per product
3. Clean product names via `ReplaceValue` (e.g., `MntWines` → `Wines`)
4. **Merge** with `Dim_Product` to get `Product_ID` (surrogate key)
5. Drop the text product column — only the key remains

**Output columns:** `Customer_ID`, `Product_ID`, `Amount`, `Enrollment_Date`

### Fact_ChannelPurchases

Captures how many purchases each customer made per channel.

**Key Steps:**
1. Select purchase columns (`NumDealsPurchases`, `NumWebPurchases`, etc.)
2. **Unpivot** → one row per customer per channel
3. Clean channel names (e.g., `NumWebPurchases` → `Web`)
4. **Merge** with `Dim_Channel` to get `Channel_ID`
5. Drop the text channel column

**Output columns:** `Customer_ID`, `Channel_ID`, `Purchases`, `Enrollment_Date`

### Fact_CampaignResponse

Captures whether each customer accepted each campaign.

**Key Steps:**
1. Select campaign columns (`AcceptedCmp1`–`AcceptedCmp5`, `Response`)
2. **Unpivot** → one row per customer per campaign
3. Rename values (e.g., `AcceptedCmp1` → `Campaign 1`, `Response` → `Campaign 6`)
4. **Merge** with `Dim_Campaign` to get `Campaign_ID`
5. Drop the text campaign column

**Output columns:** `Customer_ID`, `Campaign_ID`, `Accepted`, `Enrollment_Date`

## Dimension Tables

### Dim_Product

Lookup table of product categories (Wines, Fruits, Meat, Fish, Sweet, Gold).

**Key Steps:**
1. Filter to a single row (`ID = 1`) to extract distinct product column names
2. Unpivot to get one row per product
3. Clean names, sort alphabetically, and add an index as `Product_ID`

### Dim_Channel

Lookup table of purchase channels (Catalog, Deals, Store, Web).

**Key Steps:**
1. Same pattern as `Dim_Product` — filter, unpivot, clean, sort, index
2. Generates `Channel_ID` as surrogate key

### Dim_Campaign

Lookup table of campaigns (Campaign 1–6).

**Key Steps:**
1. Same pattern — filter, unpivot, rename, sort, index
2. Generates `Campaign_ID` as surrogate key

### Dim_Customer

The richest dimension — contains demographics, derived fields, and data quality flags.

**Key Steps:**
1. Calculate `TotalCampaignsAccepted` and `TotalChildren` from source columns
2. Select and rename relevant columns
3. **Null income handling:** flag nulls, compute median income, impute nulls with the median
4. **Derive Age** from `Year_Birth` (base year: 2024)
5. **Create Age Group** buckets: Under 30, 30–44, 45–59, 60+
6. **Flag outliers:** birth year < 1924, income > $162K
7. **Create Income Band** buckets: Low, Mid, High, Very High
8. **Group Marital Status** into `Relationship Status` (Partnered, Single, Former)
9. **Classify Campaign Engagement** (None, 1 Campaign, 2 Campaigns, 3+)
10. **Identify Children Status** (Has Children vs. No Children)

### Dim_Date

A calculated DAX table (not Power Query). Uses `CALENDAR()` spanning the full year range from the earliest to latest `Enrollment_Date`. Includes `Year`, `Month`, and `Month Number` columns.

## Data Quality Table

### _Data_Quality_Issues

A filtered view of `Dim_Customer` showing only rows with at least one data quality flag (null income, birth year outlier, or income outlier). Used for transparency in reporting.

## Common Patterns Used

| Pattern | Where Used |
|---|---|
| **Unpivot** | All 3 fact tables + 3 dimension tables — normalizes wide columns into rows |
| **Merge + Expand** | All 3 fact tables — joins dimension tables to get surrogate keys |
| **ReplaceValue** | All tables — cleans source column names into user-friendly labels |
| **Filter to single row** | Dimension tables — extracts distinct values from column headers |
| **AddColumn with conditional logic** | `Dim_Customer` — creates age groups, income bands, flags |
| **Median imputation** | `Dim_Customer` — fills null income with median |
