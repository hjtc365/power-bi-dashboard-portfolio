# Manufacturing Downtime Analysis — Dashboard Build Guide

> **Theme**: Fizzy Line Theme (`assets/Fizzy Line Theme.json`)  
> **Primary Color**: `#34A85A` | Page Canvas: `#ebf6ef` | Visual Fill: `#FFFFFF`  
> **PBIP File**: `pbip/manufacturing-downtime-analysis.pbip`

---

## Design System

The report follows a strict 3-color functional palette plus a minimal neutral ramp. Every visual must conform to this system — it is encoded in `Fizzy Line Theme.json` and should not be overridden per-visual unless explicitly noted.

### Color Tokens

| Token | Hex | Use |
|---|---|---|
| `brand.primary` | `#34A85A` | Active states, primary buttons, positive sentiment, brand accents |
| `brand.dark` | `#1F7A3A` | Title text on light, secondary button text, hover state |
| `sentiment.negative` | `#A4262C` | Operator-error series, downtime over target, low efficiency |
| `sentiment.warning` | `#C19C00` | Mid-range conditional formatting (60–75% efficiency) |
| `sentiment.neutral` | `#605E5C` | Flat KPI direction, secondary metadata |
| `canvas.page` | `#ebf6ef` | Page background ONLY |
| `canvas.visual` | `#FFFFFF` | Card / chart / table fill |
| `canvas.subtle` | `#F5FBF7` | Striped table rows, hover hint |
| `border.soft` | `#D6EEDE` | 1px visual borders, table grid lines |
| `text.primary` | `#1A1A1A` | KPI values, table data |
| `text.secondary` | `#605E5C` | Labels, axis text, captions |

> **Rule**: Orange/yellow gradients are removed from the data palette. Operator-error vs Machine-downtime is now `#A4262C` (red, error) vs `#34A85A` (green, no error) — both colorblind-safe and semantically aligned.

### Typography

| Role | Family | Size | Weight | Color |
|---|---|---|---|---|
| Page title | Segoe UI | 20pt | Semibold | `#1A1A1A` |
| Visual title | Segoe UI | 12pt | Semibold | `#1A1A1A` |
| KPI label | Segoe UI | 11pt | Regular | `#605E5C` |
| KPI value | Segoe UI | 24pt | Bold | `#1A1A1A` |
| Axis / data label | Segoe UI | 9pt | Regular | `#605E5C` |
| Tab button (active) | Segoe UI | 11pt | Semibold | `#FFFFFF` |
| Tab button (inactive) | Segoe UI | 11pt | Regular | `#1F7A3A` |

### KPI Card Standard

All KPI cards on every page must follow this spec:

```
┌─────────────────────────────┐  • Fill: #FFFFFF
│ ▎ Label (11pt, #605E5C)    │  • Border: 1px #D6EEDE, radius 8px
│ ▎                           │  • Left accent bar: 4px wide, full-height
│ ▎ Value (24pt, bold #1A1A1A)│    - Green #34A85A → positive sentiment
│ ▎                           │    - Red #A4262C → negative / cost / downtime
└─────────────────────────────┘    - Gray #605E5C → neutral / count
   ↑ 4px sentiment bar
   • Padding: 16px all sides
   • Drop shadow: none
```

| KPI | Sentiment Bar |
|---|---|
| `Total Batches`, `Total Downtime Incidents`, `Operator Batch Count` | Neutral gray |
| `Batch Efficiency %`, `Total Min Batch Time`, `Downtime Hours Saved`, `Downtime Cost Avoided` | Green |
| `Total Downtime Minutes`, `Downtime Rate %`, `Operator Error Downtime %`, `Total Over Target Minutes`, `Residual Downtime Cost` | Red |

Implementation: Use the `kpi.cardStyle` measures (`[Sentiment Color — Negative]`, etc.) bound to **Format → Effects → Background → Conditional formatting**, OR set the left-border color manually per card.

### Tab Navigation Buttons

Page navigation tabs (top-right of each page) follow a **two-state** rule:

| State | Fill | Text | Border |
|---|---|---|---|
| Active (current page) | `#34A85A` | `#FFFFFF` | none |
| Inactive | `#FFFFFF` | `#1F7A3A` | 1px `#D6EEDE` |
| Hover | `#D6EEDE` | `#1F7A3A` | 1px `#34A85A` |

> **Fix**: Inactive tabs were previously pure-white with no border, making them blend into KPI cards. The 1px green border now signals "clickable".

### Page Header Bar

Each page has a 60px-tall header strip at the top:

| Element | Spec |
|---|---|
| Fill | `#FFFFFF` |
| Bottom border | 1px `#34A85A` |
| Page icon | Left-aligned, 32×32 SVG from `assets/icons/` |
| Page title | 20pt Semibold `#1A1A1A`, 12px right of icon |
| Tab buttons | Right-aligned cluster, 8px gap |
| Slicers | Far right, beyond tabs (Product, Operator) |

### Layout Grid

- **Canvas**: 1280 × 720 (16:9)
- **Outer margin**: 20px on all sides
- **Visual gutter**: 16px between adjacent visuals (horizontal and vertical)
- **KPI row height**: 100px, 5 cards = `(1280 − 40 − 4×16) / 5 = ~235px each`
- **Section divider**: 1px `#D6EEDE` horizontal rule between header strip and content

### Color Encoding Rules (data viz)

| Series / Encoding | Color |
|---|---|
| Operator Error = Yes | `#A4262C` (red) |
| Operator Error = No (machine) | `#34A85A` (green) |
| Current vs Simulated (Page 6) | Current: `#605E5C` neutral, Simulated: `#34A85A` green |
| Heatmap gradient (matrix conditional formatting) | `#FFFFFF` → `#34A85A` |
| Efficiency conditional bands | `< 60%`: `#A4262C`, `60–75%`: `#C19C00`, `≥ 75%`: `#34A85A` |
| Trend line (single series) | `#34A85A`, 2px stroke, with markers |

### Accessibility Checklist

- [ ] All foreground/background pairs ≥ 4.5:1 contrast (WCAG AA)
- [ ] No information conveyed by color alone — pair with icon, label, or shape
- [ ] Verified in **View → Themes → Color blind safe** preview (deuteranopia + protanopia)
- [ ] Data labels enabled on all bar charts ≤ 12 categories

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

## Report Pages (7 pages)

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
| 1 | Image | `assets/icons/hero-cover.svg` (opacity reduced to 10–15% so it sits behind the title without competing) |
| 2 | Text Box | Title: "Manufacturing Downtime Analysis" — 32pt, bold, `#1A1A1A` (NOT brand green; reserve `#34A85A` for interactive elements) |
| 3 | Text Box | Subtitle: "Production Intelligence Dashboard" — 14pt, `#605E5C` |
| 4 | Card (top-right pill) | `[Last Refresh]` — small pill, `#FFFFFF` fill, 1px `#D6EEDE` border, 9pt `#605E5C` text |
| 5 | Button (Primary) | **Overview** — fill `#34A85A`, white text, 8px radius (this is the recommended starting page) |
| 6–8 | Buttons (Secondary) | Downtime Analysis, Operator Performance, Product Analysis — fill `#FFFFFF`, text `#1F7A3A`, 1px `#34A85A` border, 8px radius |

**Button Action**: Set Action → Page Navigation for each. Hover state: fill `#D6EEDE`, text `#1F7A3A`.

**Hierarchy rule**: The primary (filled green) button signals "start here". Avoid making all four buttons identical — it removes user guidance.

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
| # | Visual Type | Field / Measure | Sentiment Bar | Format |
|---|---|---|---|---|
| 1 | Card | `[Total Batches]` | Neutral gray | Title: "Total Batches" |
| 2 | Card | `[Total Downtime Minutes]` | **Red** | Title: "Total Downtime (min)" |
| 3 | Card | `[Batch Efficiency %]` | **Green** | Title: "Batch Efficiency" — pair with `[Efficiency Status Icon]` |
| 4 | Card | `[Avg Downtime Per Batch]` | Red | Title: "Avg Downtime/Batch (min)" |
| 5 | Card | `[Downtime Rate %]` | Red | Title: "Downtime Rate" |
| 6 | Line Chart | X: `Dim_Date[Date]`, Y: `[Batch Efficiency %]`, line color `#34A85A` 2px with markers | | Tooltip: `[Total Batches]`, `[Downtime Rate %]` |
| 7 | Bar Chart | X: `[Total Downtime Minutes]`, Y: `Dim_DowntimeFactor[Description]`, Legend: `Dim_DowntimeFactor[Operator Error]` | | **Yes → `#A4262C`, No → `#34A85A`** (replace orange). Sort descending. |
| 8 | Donut Chart | Values: `[Operator Error Downtime Minutes]` (red `#A4262C`), `[Machine Downtime Minutes]` (green `#34A85A`) | | Legend labels: "Operator Error" / "Machine Downtime" (matches DAX measure names) |

> **Removed**: "Ideal Batch Minutes" KPI — it's a static target rarely actioned by users. Replaced with `[Downtime Rate %]` for stronger signal in this slot.

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
| 1 | Card | `[Total Downtime Minutes]` | Red sentiment bar |
| 2 | Card | `[Total Downtime Incidents]` | Neutral gray |
| 3 | Card | `[Operator Error Downtime %]` | Red |
| 4 | Card | `[Avg Downtime Per Batch]` | Red — **replaces** "Total Batch Minutes" which doesn't belong on this page |
| 5 | Bar Chart | Y: `Dim_DowntimeFactor[Description]`, X: `[Top N Factor Downtime]` | **Replaces small left table.** Sort desc. Single color `#34A85A`. |
| 6 | Bar Chart | Y: `Dim_DowntimeFactor[Description]`, X: `[Total Downtime Incidents]` | |
| 7 | Matrix | Rows: `Fact_Productivity[Batch]`, Cols: `Dim_DowntimeFactor[Description]`, Values: `[Total Downtime Minutes]` | **Heatmap conditional format**: Background gradient `#FFFFFF` → `#34A85A` (or `#FFFFFF` → `#A4262C` if you prefer red-for-bad). Rotate column headers 30° to prevent mid-word wrap. Set min column width 70px. |

**Slicers** (move to top filter bar with Product/Operator):
- `Dim_Date[Date]` — range
- `Dim_DowntimeFactor[Operator Error]` — tile (Yes/No/All)
- `Top N[Top N]` — numeric range slider (1–12) — drives `[Top N Factor Downtime]`

> **Layout fix**: Top N slicer was previously isolated in left column wasting vertical space. Move it inline with other filters; reclaim left column for the Top N bar chart.

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
| 1 | Card | `[Total Batches]` | Neutral gray bar. Title: "Operator Batch Count" |
| 2 | Card | `[Operator Error Downtime %]` | Red |
| 3 | Card | `[Avg Downtime Per Batch]` | Red. **Replaces `[Operator Downtime Share %]`** which is meaningless without an operator filter (always reads 100%). |
| 4 | Bar Chart | Y: `Dim_Operator[Operator]`, X: `[Top N Operator Downtime]` | **Single solid color** `#34A85A` (no gradient — length already encodes magnitude). Add data labels. Sort DESC. |
| 5 | Dot Plot / Bar Chart | X: `Dim_Operator[Operator]`, Y: `[Avg Downtime Per Batch]`, with reference line at fleet average | **Replaces scatter** — 4 points crammed in a small range was unreadable. Dot plot with reference line answers "who is above average?" instantly. |
| 6 | Table | `Dim_Operator[Operator]`, `[Total Batches]`, `[Total Downtime Minutes]`, `[Avg Downtime Per Batch]`, `[Batch Efficiency %]` | Conditional data bars on `[Total Downtime Minutes]` (`#34A85A`). Conditional background on `[Batch Efficiency %]`: `<60%` red, `60–75%` warning, `≥75%` green. |
| 7 | Matrix | Rows: `Dim_Operator[Operator]`, Cols: `Dim_DowntimeFactor[Description]`, Values: `[Total Downtime Minutes]`, `[Operator Error Factor Share %]` | Visual filter: `Dim_DowntimeFactor[Operator Error] = "Yes"`. **Conditional background gradient on `[Operator Error Factor Share %]` column** (`#FFFFFF` → `#A4262C`). The Op Error % column is the actionable insight — it must be visually dominant. |

**Slicers**:
- `Dim_Date[Date]` — range
- `Dim_Product[Product]` — multi-select
- `Top N[Top N]` — drives `[Top N Operator Downtime]`

> **Removed KPI**: `[Operator Downtime Share %]` returns 100% when no operator is filtered — it carries zero information at the page-default state. If you want to keep the concept, surface it inside the per-operator table only.

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
| 1 | Card | `DISTINCTCOUNT(Fact_Productivity[ProductKey])` | "Active Products", neutral gray |
| 2 | Card | `[Avg Batch Duration Minutes]` | Neutral gray |
| 3 | Card | `[Total Over Target Minutes]` | Red |
| 4 | Slicer (single-select tile) | `Dimension Selector[Dimension Selector]` | **Replaces button group**. Header label: "Group by:". Compact horizontal layout. |
| 5 | Slicer (single-select tile) | `Measure Selector[Measure Selector]` | Header label: "Show:". |
| 6 | Bar Chart | Y: `Dimension Selector[Dimension Selector]`, X: `Measure Selector[Measure Selector]`, sorted DESC | **NEW** — fills the chart gap. The page was previously just a small table. |
| 7 | Table | `Dim_Product[Product]`, `[Total Min Batch Time]`, `[Total Actual Duration Minutes]`, `[Batch Efficiency %]` | Tighten row padding to "Small". Data bars on `[Total Actual Duration Minutes]`. Conditional background on `[Batch Efficiency %]`: red `<60%`, warning `60–75%`, green `≥75%`. Total row at **bottom** with 2px top border. |

> **Layout fix**: Previously the field-parameter buttons consumed ~25% of the canvas. Replacing with two slim tile slicers reclaims space for the new bar chart, which is the page's primary insight visual.

**Slicers**:
- `Dimension Selector[Dimension Selector]` — tile, single-select (drives chart axis)
- `Measure Selector[Measure Selector]` — tile, single-select (drives chart measure)
- `Dim_Product[Flavor]` — filter by flavor

---

### Page 6 — What-If Analysis (Downtime Reduction Simulator)

**Purpose**: Let operations leaders model the financial and efficiency impact of reducing operator-error downtime, hitting an efficiency target, and pricing the cost of lost production minutes. All three parameters are independent What-If sliders the user can move to see live recalculation.

**Why these scenarios fit the dataset**:
- Operator Error already accounts for the majority of downtime (`[Operator Error Downtime %]`) — quantifying a reduction goal is the highest-value lever.
- `Min Batch Time` gives a deterministic theoretical floor, so a "Target Efficiency %" simulation is mathematically defensible.
- Downtime is measured in minutes, which converts cleanly to a $/hr cost model for executive ROI framing.

#### What-If Parameters (Modeling tab → New parameter → Numeric range)

Add three new What-If parameter tables to the SemanticModel. Each generates a hidden DAX calculated table plus a `[* Value]` measure and a slicer-ready column.

| Parameter | Min | Max | Increment | Default | Slicer Type |
|---|---|---|---|---|---|
| `Downtime Reduction %` | 0 | 100 | 5 | 25 | Single value (slider) |
| `Target Efficiency %` | 50 | 100 | 5 | 85 | Single value (slider) |
| `Downtime Cost Per Hour` | 50 | 500 | 25 | 150 | Single value (slider) |

**TMDL skeleton** (mirrors the existing `Top N.tmdl` pattern):

```tmdl
table 'Downtime Reduction %'
    column 'Downtime Reduction %'
        formatString: 0"%"
        sourceColumn: [Downtime Reduction %]
        annotation ParameterMetadata = {"version":3,"kind":1}
    partition 'Downtime Reduction %' = calculated
        source = SELECTCOLUMNS(GENERATESERIES(0, 100, 5), "Downtime Reduction %", [Value])
    annotation PBI_NavigationStepName = Navigation
```

Repeat for `Target Efficiency %` (50–100, step 5) and `Downtime Cost Per Hour` (50–500, step 25).

#### New DAX Measures (add to `_Measures.tmdl`, displayFolder `⚙️ What-If`)

```dax
Downtime Reduction Value =
    SELECTEDVALUE('Downtime Reduction %'[Downtime Reduction %], 25) / 100

Target Efficiency Value =
    SELECTEDVALUE('Target Efficiency %'[Target Efficiency %], 85) / 100

Downtime Cost Per Hour Value =
    SELECTEDVALUE('Downtime Cost Per Hour'[Downtime Cost Per Hour], 150)

-- Simulated downtime if we cut OPERATOR ERROR downtime by the reduction %
Simulated Downtime Minutes =
    [Total Downtime Minutes] - ([Operator Error Downtime Minutes] * [Downtime Reduction Value])

-- Simulated actual duration after eliminating that fraction of downtime
Simulated Actual Duration Minutes =
    [Total Actual Duration Minutes]
        - ([Operator Error Downtime Minutes] * [Downtime Reduction Value])

-- Simulated efficiency under the reduction scenario
Simulated Batch Efficiency % =
    DIVIDE([Total Min Batch Time], [Simulated Actual Duration Minutes], 0)

-- Gap between simulated efficiency and the user's target
Efficiency Gap To Target % =
    [Simulated Batch Efficiency %] - [Target Efficiency Value]

-- Hours of production saved
Downtime Hours Saved =
    DIVIDE([Operator Error Downtime Minutes] * [Downtime Reduction Value], 60, 0)

-- $ cost avoided per the reduction scenario
Downtime Cost Avoided =
    [Downtime Hours Saved] * [Downtime Cost Per Hour Value]

-- $ cost of remaining downtime (residual)
Residual Downtime Cost =
    DIVIDE([Simulated Downtime Minutes], 60, 0) * [Downtime Cost Per Hour Value]

-- Color code KPI: green if simulated meets target, red otherwise
Target Status Icon =
    IF([Simulated Batch Efficiency %] >= [Target Efficiency Value],
       "data:image/svg+xml;base64,...kpi-up...",
       "data:image/svg+xml;base64,...kpi-down...")
```

> Reuse the existing base64 SVG strings from `[Efficiency Status Icon]` for `[Target Status Icon]`. Set `dataCategory: ImageUrl`.

#### Canvas Layout (1280×720)

```
┌────────────────────────────────────────────────────────────────────┐
│  [Header: icon-scenario.svg + "What-If: Downtime Reduction"]       │
├────────────────────────────────────────────────────────────────────┤
│  [Slider: Downtime Reduction %]  [Slider: Target Efficiency %]     │
│  [Slider: Downtime Cost / Hour]  [Date Slicer]                     │
├──────────┬──────────┬──────────┬──────────┬───────────────────────┤
│ Current  │ Simulated│ Hours    │ Cost     │  Status               │
│ Eff %    │ Eff %    │ Saved    │ Avoided  │  [Target Status Icon] │
│ 82.3%    │ 91.2%    │ 28.4 hrs │ $4,260   │  🟢 Meets target      │
├──────────┴──────────┴──────────┴──────────┴───────────────────────┤
│  [Gauge: Simulated Batch Efficiency % vs Target Efficiency Value]  │
│  Min=50%, Max=100%, Target=[Target Efficiency Value]               │
├──────────────────────────────┬─────────────────────────────────────┤
│ [Clustered Bar: by Operator] │ [Waterfall: Cost Breakdown]         │
│ Series:                      │ Categories: Current Cost,           │
│  - Current Downtime Min      │   Avoided (-), Residual,            │
│  - Simulated Downtime Min    │   Net Cost After Reduction          │
└──────────────────────────────┴─────────────────────────────────────┘
```

**Visuals**:
| # | Visual | Fields | Notes |
|---|---|---|---|
| 1 | Slicer (single value) | `'Downtime Reduction %'[Downtime Reduction %]` | Slider style, default 25% |
| 2 | Slicer (single value) | `'Target Efficiency %'[Target Efficiency %]` | Slider style, default 85% |
| 3 | Slicer (single value) | `'Downtime Cost Per Hour'[Downtime Cost Per Hour]` | Slider, $ format |
| 4 | Slicer | `Dim_Date[Date]` — range | |
| 5 | Card | `[Batch Efficiency %]` | Title: "Current Efficiency" |
| 6 | Card | `[Simulated Batch Efficiency %]` | Title: "Simulated Efficiency" — conditional font color: green if ≥ target |
| 7 | Card | `[Downtime Hours Saved]` | Format: `0.0 "hrs"` |
| 8 | Card | `[Downtime Cost Avoided]` | Format: `$#,##0` |
| 9 | Card (Image) | `[Target Status Icon]` | dataCategory=ImageUrl |
| 10 | Gauge | Value: `[Simulated Batch Efficiency %]`, Target: `[Target Efficiency Value]`, Min: 0.5, Max: 1 | |
| 11 | Clustered Bar | Y: `Dim_Operator[Operator]`, Values: `[Total Downtime Minutes]`, `[Simulated Downtime Minutes]` | Compare current vs simulated per operator |
| 12 | Waterfall | Category: hardcoded breakdown via DAX UNION calc table OR use 4 measures placed on a "Scenario Step" Field Parameter | See note below |

**Waterfall via Field Parameter** (recommended): Create a small Field Parameter `Scenario Step` with rows: `Current Total Cost`, `Cost Avoided`, `Residual Cost`, `Net Reduction`. Map each label to a measure (`[Total Downtime Minutes]/60 × [Downtime Cost Per Hour Value]`, `-[Downtime Cost Avoided]`, `[Residual Downtime Cost]`, etc.). Use that parameter on the waterfall's category axis.

**Slicers**:
- The three What-If sliders above
- `Dim_Date[Date]` — date range
- Optional: `Dim_Product[Flavor]` and `Dim_Operator[Operator]` for scoping the simulation

**Insights this page surfaces**:
- "If we cut operator-error downtime by 30%, efficiency rises from 82% → 89%, saving ~$3.5K"
- "Operator X contributes the most absolute simulated savings"
- "At a 50% reduction we still miss the 95% target — the remaining gap is machine downtime, not operator error"

---

### Page 7 — Paginated Report Export (Batch Detail Report)

**Purpose**: Provide a pixel-perfect, printable, exportable Batch & Downtime Detail report consumable by plant managers and auditors. Built with **Power BI Report Builder** (`.rdl`) and embedded into the Power BI report via the **Paginated Report visual** so end users can filter and export to PDF/Excel/CSV without leaving the interactive report.

**Output file**: `pbip/manufacturing-downtime-paginated.rdl` (Report Builder)  
**Embedded on**: Page 7 of the `.pbix` via the Paginated Report visual.

#### Report Parameters (deliberately use **different parameter types**)

| # | Parameter Name | Data Type | Selection | Default | Source |
|---|---|---|---|---|---|
| 1 | `pStartDate` | Date/Time | Date picker (single) | First of current month | Manual |
| 2 | `pEndDate` | Date/Time | Date picker (single) | Today | `=Today()` expression |
| 3 | `pOperator` | Text | **Multi-select dropdown** with "(Select All)" | All values | Query from `Dim_Operator[Operator]` |
| 4 | `pFlavor` | Text | **Single-select dropdown** | "All" | Query from `Dim_Product[Flavor]` + manual "All" entry |
| 5 | `pOperatorErrorOnly` | Boolean | **Radio buttons** (Yes / No / Both) | "Both" | Hardcoded list: `Yes`, `No`, `Both` |
| 6 | `pMinDowntimeMinutes` | Integer | **Numeric text input** (validation: ≥ 0) | 0 | Manual entry |
| 7 | `pTopNFactors` | Integer | **Available values from query** (dropdown 1–20) | 5 | `=GENERATESERIES(1,20,1)` |
| 8 | `pIncludeCharts` | Boolean | **Checkbox** (True/False) | True | Hardcoded |

> Each parameter uses a **different control style** — date picker, multi-select, single-select, radio, numeric input, query-bound dropdown, and checkbox — to demonstrate the full range of paginated parameter UX patterns.

#### Dataset Queries (DAX queries against the same Power BI semantic model)

`dsBatchDetail` — main tablix dataset:
```dax
EVALUATE
SUMMARIZECOLUMNS(
    Dim_Date[Date],
    Fact_Productivity[Batch],
    Dim_Operator[Operator],
    Dim_Product[Product],
    Dim_Product[Flavor],
    FILTER(
        VALUES(Dim_Date[Date]),
        Dim_Date[Date] >= @pStartDate && Dim_Date[Date] <= @pEndDate
    ),
    TREATAS({@pOperator1, @pOperator2 /* multi-value handled via IN */}, Dim_Operator[Operator]),
    "Actual Min", [Total Actual Duration Minutes],
    "Min Target", [Total Min Batch Time],
    "Downtime Min", [Total Downtime Minutes],
    "Efficiency", [Batch Efficiency %]
)
ORDER BY Dim_Date[Date], Fact_Productivity[Batch]
```

`dsDowntimeDetail` — downtime sub-table:
```dax
EVALUATE
FILTER(
    SUMMARIZECOLUMNS(
        Fact_Productivity[Batch],
        Dim_DowntimeFactor[Description],
        Dim_DowntimeFactor[Operator Error],
        "Downtime Min", [Total Downtime Minutes]
    ),
    [Downtime Min] >= @pMinDowntimeMinutes
)
```

`dsTopFactors` — Top N chart dataset, parameterized by `@pTopNFactors`:
```dax
EVALUATE
TOPN(
    @pTopNFactors,
    SUMMARIZECOLUMNS(
        Dim_DowntimeFactor[Description],
        "Downtime Min", [Total Downtime Minutes]
    ),
    [Downtime Min], DESC
)
```

`dsOperatorList` — populates `pOperator` dropdown:
```dax
EVALUATE VALUES(Dim_Operator[Operator]) ORDER BY Dim_Operator[Operator]
```

#### Report Builder Layout (Letter, landscape, 11"×8.5")

```
┌──────────────────────────────────────────────────────────────────┐
│  [Logo]  Manufacturing Downtime — Batch Detail Report            │
│          Period: =Parameters!pStartDate.Value & " to " & ...     │
│          Operators: =Join(Parameters!pOperator.Value, ", ")      │
│          Min Downtime Filter: ≥ =Parameters!pMinDowntimeMinutes  │
├──────────────────────────────────────────────────────────────────┤
│  [Summary Tablix — 1 row]                                        │
│  Total Batches | Total Downtime | Avg Eff % | OE Downtime %      │
├──────────────────────────────────────────────────────────────────┤
│  [Conditional: Show only if pIncludeCharts = True]               │
│  [Bar Chart: Top @pTopNFactors Downtime Factors]                 │
├──────────────────────────────────────────────────────────────────┤
│  [Tablix: Batch Detail — grouped by Date, then Batch]            │
│   Date | Batch | Operator | Product | Flavor | Actual | Target  │
│        | Downtime | Eff %                                        │
│   (Sub-row group: Downtime Factors per batch from dsDowntime)    │
├──────────────────────────────────────────────────────────────────┤
│  Page =Globals!PageNumber of =Globals!TotalPages                 │
│  Generated: =Globals!ExecutionTime                               │
└──────────────────────────────────────────────────────────────────┘
```

**Tablix formatting**:
- Group by `Date` → sub-group by `Batch` → detail rows from `dsDowntimeDetail`
- Conditional row color: light green if `Efficiency ≥ 0.85`, light red if `< 0.70`
- Number formats: `Downtime Min` = `#,##0`, `Eff %` = `0.0%`
- Repeat header on each page; enable "KeepTogether" on group

**Visibility expressions**:
- Chart visibility: `=IIF(Parameters!pIncludeCharts.Value, false, true)`  (Hidden = expression)
- "Operator Error rows only" filter: `=IIF(Parameters!pOperatorErrorOnly.Value = "Both", True, Fields!Operator_Error.Value = Parameters!pOperatorErrorOnly.Value)`

#### Embedding on Power BI Page 7

**Canvas Layout**:

```
┌──────────────────────────────────────────────────────────────────┐
│  [Header: icon-scenario.svg + "Batch Detail — Export"]            │
│  [Date Slicer]  [Operator Slicer]  [Flavor Slicer]               │
├──────────────────────────────────────────────────────────────────┤
│  [Paginated Report visual — full canvas]                         │
│  Bound to manufacturing-downtime-paginated.rdl                   │
│  Slicers above are mapped to the .rdl parameters via             │
│  the visual's Parameters pane (Field → Parameter binding)        │
└──────────────────────────────────────────────────────────────────┘
```

**Steps to embed**:
1. Publish `manufacturing-downtime-paginated.rdl` to the same Power BI workspace as the semantic model.
2. In Power BI Desktop on Page 7: **Insert → Power BI paginated report**.
3. Select the published `.rdl`.
4. In the visual's **Parameters** pane, map each `.rdl` parameter:
   - `pStartDate` ← `MIN(Dim_Date[Date])` (driven by date slicer)
   - `pEndDate` ← `MAX(Dim_Date[Date])`
   - `pOperator` ← `Dim_Operator[Operator]` (multi-select slicer)
   - `pFlavor` ← `Dim_Product[Flavor]`
   - `pOperatorErrorOnly`, `pMinDowntimeMinutes`, `pTopNFactors`, `pIncludeCharts` ← static defaults or dedicated control slicers on the canvas.
5. Add an **Export** button (Insert → Buttons → Blank, label "Export to PDF") with action: navigate to a bookmark that triggers the visual's built-in export, OR rely on the visual's native ⋯ → Export menu (PDF, Excel, Word, CSV).

**Slicers** (mapped to paginated parameters):
- `Dim_Date[Date]` — between range → maps to `pStartDate` / `pEndDate`
- `Dim_Operator[Operator]` — multi-select dropdown → `pOperator`
- `Dim_Product[Flavor]` — single-select dropdown → `pFlavor`
- (Optional canvas controls) Numeric input via a small What-If `Min Downtime Filter` parameter for `pMinDowntimeMinutes`

**Why this delivers value**:
- Auditable, archivable record of every batch with downtime breakdown — something the interactive matrix cannot deliver in a printable form.
- Plant managers get **one-click PDF/Excel export** of filtered data without DAX or Power BI knowledge.
- Demonstrates **8 distinct paginated-parameter UX patterns** in a single deliverable.

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

### What-If Simulation Measures (Page 6)
| Measure | Description |
|---|---|
| `Downtime Reduction Value` | Selected reduction % (0–1) from slider |
| `Target Efficiency Value` | Selected target efficiency % (0–1) from slider |
| `Downtime Cost Per Hour Value` | Selected $/hr cost of downtime |
| `Simulated Downtime Minutes` | Total downtime after applying reduction to operator-error portion |
| `Simulated Actual Duration Minutes` | Actual duration after removing saved minutes |
| `Simulated Batch Efficiency %` | Efficiency under the reduction scenario |
| `Efficiency Gap To Target %` | Simulated efficiency minus user's target |
| `Downtime Hours Saved` | Hours of production reclaimed |
| `Downtime Cost Avoided` | $ saved = hours saved × $/hr |
| `Residual Downtime Cost` | $ cost of remaining downtime under the scenario |
| `Target Status Icon` | KPI image: meets target (green) vs missed (red) |

### Styling Measures (🎨 Styling folder)
Bind these to **Format → Conditional formatting → Background / Font / Data colors → Field value** for sentiment-aware formatting.

| Measure | Returns | Bind To |
|---|---|---|
| `Sentiment Color Positive` | `#34A85A` | Background or accent bar of efficiency / savings KPI cards |
| `Sentiment Color Negative` | `#A4262C` | Background or accent bar of downtime / cost KPI cards |
| `Sentiment Color Neutral` | `#605E5C` | Background or accent bar of count KPI cards |
| `Efficiency Band Color` | Banded color by `[Batch Efficiency %]` (red/amber/green) | Conditional cell background in Operator/Product tables |
| `Operator Error Color` | Red/Green based on `Dim_DowntimeFactor[Operator Error]` | "Color by category" on Overview-page bar chart and donut |

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

### Downtime Reduction % (What-If Parameter — Page 6)
DAX Table: `SELECTCOLUMNS(GENERATESERIES(0, 100, 5), "Downtime Reduction %", [Value])`.  
Measure: `[Downtime Reduction Value]`. Slider range 0%–100%, step 5%, default 25%.

### Target Efficiency % (What-If Parameter — Page 6)
DAX Table: `SELECTCOLUMNS(GENERATESERIES(50, 100, 5), "Target Efficiency %", [Value])`.  
Measure: `[Target Efficiency Value]`. Slider range 50%–100%, step 5%, default 85%.

### Downtime Cost Per Hour (What-If Parameter — Page 6)
DAX Table: `SELECTCOLUMNS(GENERATESERIES(50, 500, 25), "Downtime Cost Per Hour", [Value])`.  
Measure: `[Downtime Cost Per Hour Value]`. Slider range $50–$500/hr, step $25, default $150.

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
- [ ] All 7 pages created with correct names  
- [ ] Page navigation buttons on Cover page functional (include Pages 6 & 7)  
- [ ] `Top N` slicer shows values 1–12  
- [ ] Time Intelligence slicer switches measure context correctly  
- [ ] Dimension Selector and Measure Selector slicers drive dynamic bar chart  
- [ ] Conditional formatting applied to Matrix on Page 3  
- [ ] `Last Refresh` card shows correct timestamp  
- [ ] **Page 6**: Three What-If sliders (Downtime Reduction %, Target Efficiency %, Cost/Hr) recalculate cards live  
- [ ] **Page 6**: `Simulated Batch Efficiency %` card turns green when scenario meets target  
- [ ] **Page 6**: Waterfall chart shows Current Cost → Cost Avoided → Net  
- [ ] **Page 7**: `manufacturing-downtime-paginated.rdl` published to the workspace  
- [ ] **Page 7**: Paginated Report visual embedded with all 8 parameters mapped (date picker, multi-select, single-select, radio, numeric input, query dropdown, checkbox)  
- [ ] **Page 7**: Native export to PDF / Excel / CSV verified from the visual’s ⋯ menu  
- [ ] Report saved as `.pbip` format  
