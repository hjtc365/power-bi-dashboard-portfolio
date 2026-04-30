# Manufacturing Downtime Analysis — Dashboard Build Guide

> **Theme**: Fizzy Line Theme (`assets/Fizzy Line Theme.json`)  
> **Primary Color**: `#34A85A` | Background: `#ebf6ef` | Light Background: `#d6eede`  
> **PBIP File**: `pbip/manufacturing-downtime-analysis.pbip`

---

## Key Questions Coverage

| Business Question | Measure(s) | Page |
|---|---|---|
| What's the current line efficiency? | `[Batch Efficiency %]`, `[Total Over Target Minutes]`, `[Downtime Rate %]` | Page 2 — KPI cards + trend line |
| Are any operators underperforming? | `[Batch Efficiency %]`, `[Avg Downtime Per Batch]`, `[Operator Downtime Rank]`, `[Operator Downtime Share %]` | Page 4 — bar, scatter, summary table |
| What are the leading factors for downtime? | `[Top N Factor Downtime]`, `[Factor Downtime Share %]`, `[Operator Error Downtime %]` | Page 2 (quick bar) + Page 3 (Top N + incidents) |
| Do any operators struggle with particular operator error types? | `[Total Downtime Minutes]`, `[Operator Error Factor Share %]` | Page 4 — Operator × OE Factor matrix |

---

## Data Model Overview

### Star Schema

```
Dim_Date ──────────────────────┐
Dim_Product ───────────────────┤──── Fact_Productivity  (38 rows, batch grain)
Dim_Operator ──────────────────┤──── Fact_Downtime      (sparse, batch+factor grain)
Dim_DowntimeFactor ────────────┘
_Measures                           (DAX measure container, no data)
```

### Relationships

| From | To | Cardinality | Active |
|---|---|---|---|
| `Fact_Productivity[DateKey]` | `Dim_Date[DateKey]` | Many:One | ✅ |
| `Fact_Productivity[ProductKey]` | `Dim_Product[ProductKey]` | Many:One | ✅ |
| `Fact_Productivity[OperatorKey]` | `Dim_Operator[OperatorKey]` | Many:One | ✅ |
| `Fact_Downtime[DateKey]` | `Dim_Date[DateKey]` | Many:One | ✅ |
| `Fact_Downtime[ProductKey]` | `Dim_Product[ProductKey]` | Many:One | ✅ |
| `Fact_Downtime[OperatorKey]` | `Dim_Operator[OperatorKey]` | Many:One | ✅ |
| `Fact_Downtime[FactorKey]` | `Dim_DowntimeFactor[FactorKey]` | Many:One | ✅ |

### Key Surrogate Keys
- **ProductKey**: `fn_GenerateSurrogateKey({Product}, "PROD")`
- **OperatorKey**: `fn_GenerateSurrogateKey({Operator}, "OPR")`
- **FactorKey**: `fn_GenerateSurrogateKey({Text.From(Factor)}, "DTF")`
- **DateKey**: `YYYYMMDD` integer

---

## Report Pages (5 pages)

### Page 1 — Cover

**Purpose**: Professional landing page with branding and navigation buttons.

**Canvas Layout** (1280×720):

```
┌──────────────────────────────────────────────────────────────────────┐
│  [hero-cover.svg — full-width banner, 1200×200]                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   Manufacturing Downtime Analysis                                      │
│   Production Intelligence · Aug 2024                                  │
│                                                                        │
│   Last Refresh: [Last Refresh card]                                    │
│                                                                        │
│  ┌──────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────┐ │
│  │ Overview │ │ Downtime Analysis│ │ Operator Perf.   │ │ Product  │ │
│  └──────────┘ └──────────────────┘ └──────────────────┘ └──────────┘ │
│                                                                        │
│  [icon-overview.svg] [icon-downtime-analysis.svg] [icon-operator.svg] [icon-product-analysis.svg] │
└──────────────────────────────────────────────────────────────────────┘
```

**Visuals**:
| # | Visual Type | Content |
|---|---|---|
| 1 | Image | `assets/icons/hero-cover.svg` |
| 2 | Text Box | Title: "Manufacturing Downtime Analysis" (32pt, bold, #34A85A) |
| 3 | Text Box | Subtitle: "Production Intelligence Dashboard" (14pt, #555) |
| 4 | Card | `[Last Refresh]` with label "Data Last Refreshed" |
| 5–8 | Buttons (4) | Page navigation: Overview, Downtime Analysis, Operator Performance, Product Analysis |

**Button Style**: Filled, background `#34A85A`, white text, 8px corner radius. Set Action → Page Navigation for each.

---

### Page 2 — Executive Overview

**Purpose**: High-level KPIs and production performance summary.

**Canvas Layout**:

```
┌─────────────────────────────────────────────────────────────────┐
│  [Page Header: icon-overview.svg + "Executive Overview"]         │
│  [Date Slicer]  [Product Slicer]  [Operator Slicer]             │
├──────────┬──────────┬──────────┬──────────┬────────────────────┤
│  Total   │  Total   │  Batch   │  Avg     │  Efficiency        │
│  Batches │  Downtime│  Eff. %  │  Downtime│  Status Icon       │
│  38      │  405 min │  82.3%   │  10.7 min│  🟢 / 🔴          │
├──────────┴──────────┴──────────┴──────────┴────────────────────┤
│  [Line Chart: Daily Batch Efficiency % trend by Date]           │
├────────────────────────┬────────────────────────────────────────┤
│  [Bar: Total Downtime  │  [Donut: Operator Error vs Machine     │
│   by Downtime Factor]  │   Downtime split]                      │
└────────────────────────┴────────────────────────────────────────┘
```

**Visuals**:
| # | Visual Type | Field / Measure | Format |
|---|---|---|---|
| 1 | Card | `[Total Batches]` | Title: "Total Batches" |
| 2 | Card | `[Total Downtime Minutes]` | Title: "Total Downtime (min)" |
| 3 | Card | `[Batch Efficiency %]` | Title: "Batch Efficiency" |
| 4 | Card | `[Avg Downtime Per Batch]` | Title: "Avg Downtime/Batch (min)" |
| 5 | Card (Image) | `[Efficiency Status Icon]` | dataCategory=ImageUrl already set |
| 6 | Line Chart | X: `Dim_Date[Date]`, Y: `[Batch Efficiency %]` | Tooltip: `[Total Batches]`, `[Downtime Rate %]` |
| 7 | Bar Chart | X: `[Total Downtime Minutes]`, Y: `Dim_DowntimeFactor[Description]` | Sort descending |
| 8 | Donut Chart | Values: `[Operator Error Downtime Minutes]`, `[Machine Downtime Minutes]` | Legend: custom labels |

**Slicers**:
- `Dim_Date[Date]` — Date range slicer
- `Dim_Product[Product]` — Dropdown or tile
- `Dim_Operator[Operator]` — Dropdown or tile

**Time Intelligence Slicer**: Add `Time Intelligence[Time Period]` slicer (from Calculation Group) to switch KPI context between Actual, YoY, YTD, etc.

---

### Page 3 — Downtime Analysis

**Purpose**: Deep dive into downtime factors, frequency, and duration.

**Canvas Layout**:

```
┌─────────────────────────────────────────────────────────────────┐
│  [Header: icon-downtime-analysis.svg + "Downtime Analysis"]      │
│  [Date Slicer]  [Operator Error filter: Yes/No]  [Top N Slicer] │
├─────────────────────────────────────────────────────────────────┤
│  Total Downtime  │  Total Incidents  │  Operator Error %        │
│  [405 min]       │  [42]             │  [62.3%]                  │
├────────────────────────┬────────────────────────────────────────┤
│  [Bar: Top N Downtime  │  [Bar: Downtime Incidents by Factor]   │
│   by Factor (Top N     │   (Count of rows in Fact_Downtime)     │
│   What-If)]            │                                        │
├────────────────────────┴────────────────────────────────────────┤
│  [Matrix: Batch × Factor — Downtime Minutes heatmap]            │
│  Rows: Batch, Cols: Dim_DowntimeFactor[Description]             │
│  Values: [Total Downtime Minutes], conditional format (green)   │
└─────────────────────────────────────────────────────────────────┘
```

**Visuals**:
| # | Visual Type | Fields | Notes |
|---|---|---|---|
| 1 | Card | `[Total Downtime Minutes]` | |
| 2 | Card | `[Total Downtime Incidents]` | |
| 3 | Card | `[Operator Error Downtime %]` | |
| 4 | Bar Chart | Y: `Dim_DowntimeFactor[Description]`, X: `[Top N Factor Downtime]` | Filtered by Top N slicer |
| 5 | Bar Chart | Y: `Dim_DowntimeFactor[Description]`, X: `[Total Downtime Incidents]` | |
| 6 | Matrix | Rows: `Fact_Productivity[Batch]`, Cols: `Dim_DowntimeFactor[Description]`, Values: `[Total Downtime Minutes]` | Conditional format: white→green scale |

**Slicers**:
- `Dim_Date[Date]` — range
- `Dim_DowntimeFactor[Operator Error]` — tile (Yes/No)
- `Top N[Top N]` — slider (1–12) — drives `[Top N Factor Downtime]`

---

### Page 4 — Operator Performance

**Purpose**: Operator-level accountability and efficiency benchmarking.

**Canvas Layout**:

```
┌─────────────────────────────────────────────────────────────────┐
│  [Header: icon-operator.svg + "Operator Performance"]            │
│  [Date Slicer]  [Product Slicer]  [Top N Slicer]                │
├──────────────┬──────────────┬──────────────────────────────────┤
│  Operator    │  Operator    │  Operator Downtime Share %        │
│  Batch Count │  Error %     │                                   │
├──────────────┴──────────────┴──────────────────────────────────┤
│  [Bar: Total Downtime Minutes by Operator]                      │
│  Colors by downtime rank (use data bars or diverging palette)   │
├────────────────────────┬────────────────────────────────────────┤
│  [Scatter:             │  [Table: Operator Summary]             │
│   X: Total Batches     │  Cols: Operator, Batches, Total        │
│   Y: Total Downtime    │  Downtime, Avg Downtime, Eff%          │
│   Size: Batch Eff %    │  Sort by Total Downtime DESC           │
│   Legend: Operator]    │                                        │
├────────────────────────┴────────────────────────────────────────┤
│  [Matrix: Operator × Operator Error Factor — answers Q4]        │
│  Rows: Dim_Operator[Operator]                                   │
│  Cols: Dim_DowntimeFactor[Description]                          │
│  Values: [Total Downtime Minutes] + [Operator Error Factor %]   │
│  Visual filter: Dim_DowntimeFactor[Operator Error] = "Yes"      │
└─────────────────────────────────────────────────────────────────┘
```

**Visuals**:
| # | Visual Type | Fields | Notes |
|---|---|---|---|
| 1 | Card | `[Total Batches]` filtered by selected operator | |
| 2 | Card | `[Operator Error Downtime %]` | |
| 3 | Card | `[Operator Downtime Share %]` | |
| 4 | Bar Chart | Y: `Dim_Operator[Operator]`, X: `[Top N Operator Downtime]` | Sorted DESC |
| 5 | Scatter | X: `[Total Batches]`, Y: `[Total Downtime Minutes]`, Size: `[Batch Efficiency %]`, Legend: `Dim_Operator[Operator]` | |
| 6 | Table | `Dim_Operator[Operator]`, `[Total Batches]`, `[Total Downtime Minutes]`, `[Avg Downtime Per Batch]`, `[Batch Efficiency %]` | Conditional bars on downtime |
| 7 | Matrix | Rows: `Dim_Operator[Operator]`, Cols: `Dim_DowntimeFactor[Description]`, Values: `[Total Downtime Minutes]`, `[Operator Error Factor Share %]` | Add visual-level filter: `Dim_DowntimeFactor[Operator Error] = "Yes"`. Conditional format the values — highlights which operator is disproportionately responsible for each error type. |

**Slicers**:
- `Dim_Date[Date]` — range
- `Dim_Product[Product]` — multi-select
- `Top N[Top N]` — drives `[Top N Operator Downtime]`

---

### Page 5 — Product Analysis

**Purpose**: Product-level performance: which products cause most downtime and slowest batches.

**Canvas Layout**:

```
┌─────────────────────────────────────────────────────────────────┐
│  [Header: icon-product-analysis.svg + "Product Analysis"]        │
│  [Dimension Selector slicer]  [Measure Selector slicer]         │
├──────────────┬──────────────┬──────────────────────────────────┤
│  Total       │  Avg Batch   │  Total Over Target               │
│  Products    │  Duration    │  Minutes                         │
├──────────────┴──────────────┴──────────────────────────────────┤
│  [Bar: Dynamic measure by Dimension]                            │
│  X: [Measure Selector], Y: [Dimension Selector]                 │
│  (Field Parameters drive both axes)                             │
├────────────────────────┬────────────────────────────────────────┤
│  [Bar: Batch Eff % by  │  [Bar: Over Target Minutes by         │
│   Product]             │   Product]                            │
└────────────────────────┴────────────────────────────────────────┘
```

**Visuals**:
| # | Visual Type | Fields | Notes |
|---|---|---|---|
| 1 | Card | `COUNTROWS(Dim_Product)` or `DISTINCTCOUNT(Fact_Productivity[ProductKey])` | "Active Products" |
| 2 | Card | `[Avg Batch Duration Minutes]` | |
| 3 | Card | `[Total Over Target Minutes]` | |
| 4 | Bar Chart | Y: `Dimension Selector[Dimension Selector]`, X: `Measure Selector[Measure Selector]` | **Field Parameter visuals** |
| 5 | Bar Chart | Y: `Dim_Product[Product]`, X: `[Batch Efficiency %]` | Sort DESC |
| 6 | Bar Chart | Y: `Dim_Product[Product]`, X: `[Total Over Target Minutes]` | Conditional color: red if > 0 |

**Slicers**:
- `Dimension Selector[Dimension Selector]` — tile, single-select (drives chart axis)
- `Measure Selector[Measure Selector]` — tile, single-select (drives chart measure)
- `Dim_Product[Flavor]` — filter by flavor

---

## Applying the Theme

1. In Power BI Desktop, go to **View → Themes → Browse for themes**
2. Select `dashboards/manufacturing-downtime/assets/Fizzy Line Theme.json`
3. Theme applies automatically to all pages

> **Theme Colors**:
> - Primary: `#34A85A` (green)  
> - Background: `#ebf6ef`  
> - Light Background (cards/panels): `#d6eede`  
> - Text: `#1A1A1A`  
> - Accent: `#1F7A3A` (dark green)

---

## SVG Assets Reference

Located in `assets/icons/`:

| File | Usage |
|---|---|
| `hero-cover.svg` | Page 1 banner image (1200×400) |
| `icon-cover.svg` | Navigation button icon (Cover page) |
| `icon-overview.svg` | Page 2 header icon |
| `icon-downtime-analysis.svg` | Page 3 header icon |
| `icon-operator-performance.svg` | Page 4 header icon |
| `icon-product-analysis.svg` | Page 5 header icon |
| `icon-scenario.svg` | Page 5 field parameter controls icon |
| `kpi-up.svg` | KPI direction: positive (green triangle) |
| `kpi-down.svg` | KPI direction: negative (red triangle) |
| `kpi-neutral.svg` | KPI direction: flat (gray dash) |

**Adding SVG images**: Insert → Image → browse to `.svg` file. Set scaling to "Fit" or "Fill".

---

## DAX Measures Reference

### Core Metrics
| Measure | Formula Summary |
|---|---|
| `Total Batches` | `COUNTROWS(Fact_Productivity)` |
| `Total Downtime Minutes` | `SUM(Fact_Downtime[Downtime Minutes])` |
| `Total Actual Duration Minutes` | `SUM(Fact_Productivity[Actual Duration Minutes])` |
| `Total Over Target Minutes` | `SUM(Fact_Productivity[Over Target Minutes])` |
| `Avg Batch Duration Minutes` | `DIVIDE([Total Actual Duration Minutes], [Total Batches], 0)` |
| `Avg Downtime Per Batch` | `DIVIDE([Total Downtime Minutes], [Total Batches], 0)` |
| `Operator Error Downtime Minutes` | `CALCULATE([Total Downtime Minutes], Operator Error = "Yes")` |
| `Machine Downtime Minutes` | `CALCULATE([Total Downtime Minutes], Operator Error = "No")` |
| `Total Downtime Incidents` | `COUNTROWS(Fact_Downtime)` |

### Rates & Ratios
| Measure | Formula Summary |
|---|---|
| `Batch Efficiency %` | `DIVIDE([Total Min Batch Time], [Total Actual Duration Minutes], 0)` |
| `Downtime Rate %` | `DIVIDE([Total Downtime Minutes], [Total Actual Duration Minutes], 0)` |
| `Operator Error Downtime %` | `DIVIDE([Operator Error Downtime Minutes], [Total Downtime Minutes], 0)` |
| `Operator Downtime Share %` | Share vs all operators (removes Dim_Operator filter) |
| `Factor Downtime Share %` | Share vs all factors (removes Dim_DowntimeFactor filter) |
| `Operator Error Factor Share %` | This OE factor's share of the operator's total OE downtime — use in Operator × OE Factor matrix to identify disproportionate error concentrations |

### Dynamic Measures (What-If / Ranking)
| Measure | Description |
|---|---|
| `Top N Factor Downtime` | Downtime for top N factors by `Top N` parameter |
| `Top N Operator Downtime` | Downtime for top N operators by `Top N` parameter |
| `Operator Downtime Rank` | Dense rank by downtime descending |
| `Factor Downtime Rank` | Dense rank by downtime descending |
| `Efficiency Status Icon` | Returns base64 SVG URL for KPI direction indicator |

---

## Calculation Groups

### Time Intelligence (precedence = 10)
Column: `Time Intelligence[Time Period]`  
Items: **Actual**, **Prior Year**, **YoY Change**, **YoY %**, **YTD**, **MTD**, **Prior Month**, **MoM %**

Usage: Add `Time Intelligence[Time Period]` as a slicer or small multiples axis to apply time context to any measure.

### Format Switcher (precedence = 5)
Column: `Format Switcher[Number Format]`  
Items: **Default**, **Integer**, **Decimal**, **Percentage**

Usage: Useful for dynamic table or matrix formatting.

---

## Field Parameters

### Dimension Selector
DAX Table (PBI_ResultType=FieldParameter). Selectable dimensions:
- Operator, Product, Flavor, Downtime Factor, Date, Month

Usage: Drag `Dimension Selector[Dimension Selector]` to chart axis. Add `Dimension Selector[Dimension Selector Fields]` to axis **as the field** (not the parameter column).

### Measure Selector
DAX Table (PBI_ResultType=FieldParameter). Selectable measures:
- Total Downtime Minutes, Total Batches, Batch Efficiency %, Downtime Rate %, Operator Error Downtime %, Total Over Target Minutes

### Top N (What-If Parameter)
DAX Table: `GENERATESERIES(1, 12, 1)`. Used with `[Top N Value]` measure.  
Add `Top N[Top N]` as a numeric range slicer (min=1, max=12, step=1).

---

## Tooltip Pages (Optional)

Create small tooltip pages (320×240 canvas) for rich hover tooltips:

**Tooltip: Batch Detail**  
- Visual: Card — `[Avg Batch Duration Minutes]`, `[Avg Downtime Per Batch]`, `[Batch Efficiency %]`
- Set as tooltip page: Format page → Tooltip toggle ON

**Tooltip: Factor Detail**  
- Visual: Card — `[Total Downtime Minutes]`, `[Total Downtime Incidents]`, `[Factor Downtime Share %]`
- Assign to Downtime Analysis charts via Visual → Tooltip → Report page

---

## Quality Checklist

- [ ] All 7 relationships visible in Model view with correct cardinality arrows  
- [ ] `Dim_Date` marked as Date Table (right-click → Mark as date table → `DateKey`)  
- [ ] `Year-Month` column sorted by `YearMonthNum`  
- [ ] All surrogate key columns hidden in Report view  
- [ ] Theme applied (Fizzy Line Theme.json)  
- [ ] All 5 pages created with correct names  
- [ ] Page navigation buttons on Cover page functional  
- [ ] `Top N` slicer shows values 1–12  
- [ ] Time Intelligence slicer switches measure context correctly  
- [ ] Dimension Selector and Measure Selector slicers drive dynamic bar chart  
- [ ] Conditional formatting applied to Matrix on Page 3  
- [ ] `Last Refresh` card shows correct timestamp  
- [ ] Report saved as `.pbip` format  
