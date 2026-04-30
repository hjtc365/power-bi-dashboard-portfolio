---
name: power-bi-skill
description: "Use this skill whenever the user wants to build, design, or document a Power BI report from CSV data files and/or a data dictionary. Triggers include: any mention of 'Power BI', '.pbix', 'star schema', 'Kimball', 'DAX measures', 'Power Query', 'M code', 'data model', 'field parameters', 'what-if analysis', 'numeric parameters', 'MCP server', 'powerbi-modeling-mcp', 'calculation groups', 'visual calculations', or requests to analyze CSV data and turn it into an interactive report. When MCP is connected, ALL phases (3–8) are fully automated via MCP — including Power Query M code (via named_expression_operations + table_operations mExpression), Field Parameters (DAX calculated tables with PBI_ResultType annotation), and What-If Parameters (DAX calculated tables with PBI_NavigationStepName annotation). Phase 9 report canvas is the ONLY phase requiring manual Desktop UI work. Always use fn_GenerateSurrogateKey for hashed surrogate keys — never use Table.AddIndexColumn or JOIN-based key lookups in fact tables. Always use Calculation Groups for time intelligence and format string switching. Always suggest Visual Calculations for within-visual running totals, ranks, and period comparisons. Do NOT use for Excel pivot tables, Tableau, Looker, or other BI tools."
---

# Power BI Report Building Skill

## Overview

This skill guides you through building a professional, multi-page Power BI report from raw data files and a data dictionary. It follows **Kimball-style dimensional modeling**, uses Power Query (M) for all ETL, and writes production-quality DAX — with full automation via the Power BI Modeling MCP server wherever possible.

> **This skill is data-driven by design.** Every table name, column name, measure, relationship, and report page must be derived from analysing the **actual source data and data dictionary provided by the user** (Phase 1). Nothing is assumed or copied from examples. The examples shown throughout this document are generic illustrations only — they will not match your dataset.

**Three mandatory patterns always applied:**

1. **`fn_GenerateSurrogateKey`** — All surrogate keys in dimension and fact tables use this stable, position-independent hashed key function. Never use `Table.AddIndexColumn` (position-dependent, breaks incremental refresh) or `Table.NestedJoin` (full-table scan on every row).
2. **Calculation Groups** — Time intelligence (PY, YoY, YTD, MTD, QTD, MoM) and format-string switching are always Calculation Groups. Never create per-metric time intelligence measures — that creates unmaintainable sprawl.
3. **Visual Calculations** — Running totals, ranks, period deltas, and moving averages that are scoped to a specific visual are Visual Calculations, not model measures.

---

## Build Sequence at a Glance

The correct sequence is: **understand the data first, then design, then build, then configure, then layer interactivity, then report**.

| Phase | What happens | Primary tool |
|---|---|---|
| **Prerequisites** | Set up MCP connection | MCP + VS Code |
| **Phase 1** | Profile source data, read dictionary, define business questions | Analyst work |
| **Phase 2** | Design Kimball star schema from Phase 1 findings | Analyst work |
| **Phase 3** | Write Power Query ETL for every table | **MCP** (preferred) or Power Query Editor |
| **Phase 4** | Scaffold model via MCP — tables, relationships, keys | **MCP** |
| **Phase 5** | Configure model — sort-by, categories, hierarchies, RLS | **MCP** |
| **Phase 6** | Build DAX measure library | **MCP** |
| **Phase 7** | Add Calculation Groups (time intelligence + format switching) | **MCP** |
| **Phase 8** | Add Field Parameters and What-If Parameters | **MCP** |
| **Phase 9** | Build report pages and format visuals | Desktop UI — ⚠️ **ONLY MANUAL PHASE** |
| **Phase 10** | Quality audit and performance check | **MCP** + Desktop UI |

> ⚠️ **Phase 9 is the only phase that requires manual work in Power BI Desktop.** Phases 3 through 8 — including Power Query ETL, parameters, and calculation groups — are fully automated via MCP when connected.

> **Why this order?** You cannot scaffold tables (Phase 4) until you have designed the schema (Phase 2), and you cannot design the schema until you have profiled the data (Phase 1). Phases 1 and 2 are pure analysis — no tools required. Every tool-based phase depends on the analysis output before it.

---

## Prerequisites — MCP Environment Setup

This section is one-time environment setup. Read it before Phase 1. Come back to it only if you need to connect MCP for the first time.

### What Is the Power BI Modeling MCP Server?

Microsoft's `powerbi-modeling-mcp` is an official local MCP server that connects AI agents directly to a live Power BI semantic model open in Power BI Desktop or a Fabric Workspace. It uses the same TOM (Tabular Object Model) API as external tools — every change made through MCP is identical to making the change in the Desktop UI.

**GitHub**: https://github.com/microsoft/powerbi-modeling-mcp
**VS Code Extension**: Search "Power BI Modeling MCP" in the VS Code Marketplace

### Two Power BI MCP Servers — Know the Difference

| Server | Purpose | When to use |
|---|---|---|
| **Modeling MCP** (local) | Build and modify semantic models — tables, columns, measures, relationships, calc groups | All build/edit operations in this skill |
| **Remote MCP** (cloud) | Query existing published models with natural language → DAX | Ad-hoc data questions against a published model |

**This skill uses the Modeling MCP server exclusively.**

### Setup Steps

**Prerequisites:**
1. Power BI Desktop (latest from Microsoft Store or powerbi.com)
2. Visual Studio Code with GitHub Copilot extension
3. Power BI Modeling MCP VS Code Extension (search "Power BI Modeling MCP" in Extensions)

**Connect to a Desktop model:**
```
In GitHub Copilot Chat (VS Code):
"Connect to my Power BI file [filename.pbix]"
MCP auto-discovers the Analysis Services instance running behind Desktop
```

**Connect to a Fabric Workspace model:**
```
In GitHub Copilot Chat (VS Code):
"Connect to semantic model [ModelName] in workspace [WorkspaceName]"
Uses your existing Entra ID credentials — no extra login required
```

### MCP Mode Check — Do This at the Start of Every Session

```
IF Power BI Modeling MCP server is connected AND a .pbix or Fabric model is open:
    → MCP mode: execute ALL model operations directly via MCP — including Power Query M, parameters, and calc groups
    → Manual UI ONLY for: Phase 9 report canvas layout, visual placement, slicer configuration, theme upload, and conditional formatting

ELSE:
    → Manual mode: produce full M code, DAX, and step-by-step UI instructions
    → Note at top of response: "Connect Power BI Modeling MCP to automate these steps"
```

Always declare which mode is active at the start of your response.

**Security note:** The Modeling MCP server operates within your existing Power BI permissions. It cannot bypass Row-Level Security or access models you do not have permission to edit. Data and metadata retrieved during a session may be sent to the LLM provider as conversation context — always back up your .pbix file before running MCP operations.

---

## Phase 1 — Understand the Source Data

> **This is the most important phase.** Everything that follows — the schema design, the ETL, the measures, the report pages — is derived from what you learn here. Do not skip or rush this phase. Do not start Phase 2 until this phase is fully documented.

### 1.1 Profile Every Source File

For each CSV, Excel, database table, or other source file, capture the following:

| Check | What to capture |
|---|---|
| **File / table name** | Exact filename, sheet name, or table name |
| **Row count** | Approximate number of rows — determines performance strategy |
| **Grain** | What does **one row** represent? (e.g., one order line, one patient visit, one train journey) |
| **Column names** | Exact spelling, casing, spaces — do not normalise yet |
| **Data types** | Text, whole number, decimal, date, boolean — as they appear in the raw file |
| **Candidate keys** | Which column(s) uniquely identify a row? |
| **Foreign keys** | Which columns reference entities in other files? |
| **Null / blank rates** | Which columns have missing values and how often? |
| **Date columns** | Format string (YYYY-MM-DD, MM/DD/YYYY, epoch, Unix timestamp, etc.) |
| **Numeric columns** | Which are additive facts (sum)? Which are rates or ratios (average)? Which are non-additive? |
| **Categorical columns** | Cardinality, allowed values, naming inconsistencies |
| **Data quality issues** | Duplicates, out-of-range values, encoding problems, mixed types in one column |

### 1.2 Read the Data Dictionary

Map every field in the data dictionary to the corresponding source column. For each field note:

- **Business definition** — use this as the measure description and column tooltip in the model
- **Allowed values / enumerations** — use for data validation rules in Power Query
- **Calculated or derived fields** — decide whether to build in Power Query (during ETL) or DAX (at query time); never both
- **Hierarchies** — e.g., Year → Quarter → Month → Date; Country → Region → City; Category → Sub-Category → Product
- **Sensitive fields** — flag any PII or commercially sensitive columns that need RLS or exclusion

### 1.3 Identify and Document Business Questions

List every analytical question the report must answer. This drives everything: schema decisions, which measures to write, which pages to build, which slicers to include.

For each question, document:
- The **metric** (what is being measured — sum, count, ratio, rank)
- The **dimensions** to slice by (who, what, where, when)
- The **granularity** of the answer (daily, monthly, by category, by region)
- The **time intelligence** needed (YTD, YoY %, prior month, etc.)
- The **target audience** (executive summary vs operational detail)

Group related questions into **themes** — each theme typically becomes one report page.

### 1.4 Phase 1 Output — Document Before Proceeding

Before starting Phase 2, you must be able to answer:

- [ ] What is the grain of each source table?
- [ ] Which columns are facts (numeric measures) vs dimensions (descriptive attributes)?
- [ ] Which columns are candidate keys and foreign keys linking files together?
- [ ] Are there any many-to-many relationships in the source that need a bridge table?
- [ ] Which date columns exist and what format are they in?
- [ ] What are the 10–15 most important business questions the report must answer?
- [ ] Are there any data quality problems that must be fixed in Power Query before modeling?

---

## Phase 2 — Design the Star Schema

> **Design on paper (or whiteboard) before opening Power BI Desktop.** The schema you design here is the blueprint for everything in Phases 3–10. A good schema makes every downstream phase straightforward. A poor schema makes every downstream phase painful.

### 2.1 Kimball Dimensional Modeling Principles

Always follow these rules — they are not optional:

1. **Fact tables** contain only numeric, additive measurements and foreign keys. No descriptive text, no labels, no attributes.
2. **Dimension tables** contain descriptive attributes and a single surrogate key as the primary key.
3. Every relationship is **one-to-many** from dimension (one side) to fact (many side), single direction, unless a bridge table is explicitly required for a many-to-many.
4. Use a **Date dimension** for every date column in a fact table. Never filter directly on a raw date column.
5. **Conformed dimensions** are shared across multiple fact tables (e.g., the same `Dim_Date` serves all facts).
6. **Avoid snowflaking** — keep all dimension attributes flat in one table unless a hierarchy node has over 1 million distinct rows.
7. **One row per grain** in every fact table — if your fact table has duplicate rows at the intended grain, fix it in Power Query before loading.

### 2.2 Table Naming Convention

Apply this convention to the tables you identify from your Phase 1 analysis:

```
Fact_<Subject>          e.g., Fact_Sales, Fact_Encounters, Fact_Journeys, Fact_Orders
Dim_<Entity>            e.g., Dim_Date, Dim_Patient, Dim_Product, Dim_Station
Bridge_<Relationship>   e.g., Bridge_OrderProduct  (for many-to-many only)
Raw_<Source>            e.g., Raw_Orders  (staging query — load disabled)
_Measures               DAX measure container (no columns)
<Param Name>            Field Parameter and What-If tables (DAX calculated tables created via MCP)
```

> `Dim_Date` and `_Measures` are present in **every** model regardless of the dataset. Everything else is determined by your Phase 1 analysis.

### 2.3 Star Schema Design Process

Work through these steps using your Phase 1 findings:

**Step 1 — Identify fact tables**
Each process or event that produces numeric measurements becomes a fact table. One grain = one fact table. If you have two very different grains (e.g., order header and order line), use separate fact tables.

**Step 2 — Identify dimension tables**
For every descriptive attribute group in a fact row (who bought it, what was bought, where, when), create a dimension table. Common dimensions: Date, Customer/Patient/User, Product/Service, Location/Geography, Channel, Payer, Provider.

**Step 3 — Identify surrogate keys**
Every dimension needs a surrogate key. Use `fn_GenerateSurrogateKey` (see Phase 3) — never sequential integers.

**Step 4 — Identify foreign keys in fact tables**
The fact table contains one foreign key column per dimension it relates to. These foreign keys are computed using the same hash function as the dimension's surrogate key — no JOIN required.

**Step 5 — Handle many-to-many relationships**
If a fact row legitimately relates to multiple dimension rows (e.g., one order can have multiple promotions), use a Bridge table. Do not create bidirectional relationships as a workaround.

**Step 6 — Draw the ERD**
Sketch the full entity-relationship diagram before writing any M code. Every table, every key, every relationship direction must be explicit on paper.

### 2.4 Dim_Date — Always Required

Every model includes a full calendar date dimension. This M code is a fixed template — it does not change based on the dataset. After loading, mark it as the Date Table in Phase 4.

Adjust `StartDate` and `EndDate` to bracket your data's actual date range plus a reasonable future buffer.

```m
// Dim_Date — paste into a Blank Query named Dim_Date
// Adjust StartDate and EndDate to match your dataset's date range
let
    StartDate     = #date(2018, 1, 1),   // <-- set to earliest date in your data
    EndDate       = #date(2030, 12, 31), // <-- set to reasonable future date
    DayCount      = Duration.Days(EndDate - StartDate) + 1,
    Source        = List.Dates(StartDate, DayCount, #duration(1,0,0,0)),
    AsTable       = Table.FromList(Source, Splitter.SplitByNothing()),
    TypedDate     = Table.TransformColumnTypes(AsTable,{{"Column1", type date}}),
    Renamed       = Table.RenameColumns(TypedDate,{{"Column1","Date"}}),
    YearCol       = Table.AddColumn(Renamed,      "Year",            each Date.Year([Date]),            Int64.Type),
    QuarterNum    = Table.AddColumn(YearCol,       "Quarter Number",  each Date.QuarterOfYear([Date]),   Int64.Type),
    QuarterLabel  = Table.AddColumn(QuarterNum,    "Quarter",         each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    MonthNum      = Table.AddColumn(QuarterLabel,  "Month Number",    each Date.Month([Date]),            Int64.Type),
    MonthName     = Table.AddColumn(MonthNum,      "Month Name",      each Date.ToText([Date],"MMMM","en-US"),   type text),
    MonthShort    = Table.AddColumn(MonthName,     "Month Short",     each Date.ToText([Date],"MMM","en-US"),    type text),
    YearMonth     = Table.AddColumn(MonthShort,    "Year-Month",      each Date.ToText([Date],"yyyy-MM","en-US"), type text),
    WeekNum       = Table.AddColumn(YearMonth,     "Week Number",     each Date.WeekOfYear([Date]),       Int64.Type),
    DayOfWeekNum  = Table.AddColumn(WeekNum,       "Day of Week",     each Date.DayOfWeek([Date], Day.Monday) + 1, Int64.Type),
    DayName       = Table.AddColumn(DayOfWeekNum,  "Day Name",        each Date.ToText([Date],"dddd","en-US"),   type text),
    DayShort      = Table.AddColumn(DayName,       "Day Short",       each Date.ToText([Date],"ddd","en-US"),    type text),
    IsWeekend     = Table.AddColumn(DayShort,      "Is Weekend",      each Date.DayOfWeek([Date]) = Day.Saturday or Date.DayOfWeek([Date]) = Day.Sunday, type logical),
    FiscalYear    = Table.AddColumn(IsWeekend,     "Fiscal Year",     each if Date.Month([Date]) >= 10 then Date.Year([Date]) + 1 else Date.Year([Date]), Int64.Type),
    IsCurrentDay  = Table.AddColumn(FiscalYear,    "Is Current Day",  each [Date] = Date.From(DateTime.LocalNow()), type logical),
    IsCurrentMth  = Table.AddColumn(IsCurrentDay,  "Is Current Month",each Date.Year([Date]) = Date.Year(DateTime.LocalNow()) and Date.Month([Date]) = Date.Month(DateTime.LocalNow()), type logical),
    IsCurrentYr   = Table.AddColumn(IsCurrentMth,  "Is Current Year", each Date.Year([Date]) = Date.Year(DateTime.LocalNow()), type logical),
    DateKey       = Table.AddColumn(IsCurrentYr,   "DateKey",         each Date.Year([Date]) * 10000 + Date.Month([Date]) * 100 + Date.Day([Date]), Int64.Type)
in
    DateKey
```

### 2.5 Surrogate Keys — Always Use `fn_GenerateSurrogateKey`

**Rule: never use `Table.AddIndexColumn` for surrogate keys. Never use `Table.NestedJoin` to resolve foreign keys in fact tables.**

- `Table.AddIndexColumn` is position-dependent — it changes if source row order changes, breaks incremental refresh, and is meaningless across multiple source files.
- `Table.NestedJoin` on a large fact table performs a full table scan on every row — it is a serious performance bottleneck at scale.

Instead, generate a **stable, deterministic hashed key** from the natural key column(s) using `fn_GenerateSurrogateKey`. The same natural key always produces the same hash regardless of row order or load sequence. Apply the identical hash in both the dimension table (as surrogate key) and the fact table (as foreign key) — no join needed.

#### Define `fn_GenerateSurrogateKey` (one-time per .pbix)

Create a Blank Query named `fn_GenerateSurrogateKey`, paste this code, and set **Enable Load = off**:

```m
// fn_GenerateSurrogateKey
// Generates a stable Base64-encoded surrogate key from one or more natural key values.
// Usage: fn_GenerateSurrogateKey({[ColA], [ColB]}, "PREFIX")
//        fn_GenerateSurrogateKey({[NaturalID]}, null)
(
    values as list,
    optional prefix as nullable text
) as text =>
let
    Cleaned =
        List.Transform(values, each
            let
                v       = if _ = null then "NULL" else Text.From(_),
                trimmed = Text.Trim(v),
                lowered = Text.Lower(trimmed),
                safe    = Text.Replace(lowered, "|", "")
            in safe),
    keyText  = Text.Combine(Cleaned, "|"),
    encoded  = Binary.ToText(Text.ToBinary(keyText), BinaryEncoding.Base64),
    finalKey = if prefix = null then encoded else prefix & "|" & encoded
in
    finalKey
```

#### Key Design Rules

| Rule | Why |
|---|---|
| Pass natural key columns in the **same fixed order** in dim and fact | Order matters — `{[A],[B]}` ≠ `{[B],[A]}` |
| Use the **same prefix** string in dim and fact for the same entity | Prefix is part of the hash input |
| Keep `DateKey` as `Int64` (`YYYYMMDD` integer) | Integer date joins are faster than text hash joins |
| Surrogate key column type is always `type text` | Base64 output is always text |
| Verify key overlap before Close & Apply | Count distinct keys in dim vs fact — any fact key with no dim match will break filters |

> ⚠️ **Duplicate surrogate key trap — always deduplicate AFTER key generation, not before.**
> `fn_GenerateSurrogateKey` lowercases all input values before hashing. If `Table.Distinct` is applied on the natural key column **before** key generation, case-variant duplicates (e.g., `"Chateau X"` and `"CHATEAU X"`) survive into the dimension, producing two rows with **identical surrogate keys** — breaking the relationship. Always generate the key first, then deduplicate on the key column:
>
> ```m
> // ✅ Correct — distinct on natural key first, then generate key, then dedup on key
> Distinct    = Table.Distinct(Selected, {"title"}),          // removes exact-text duplicates
> AddKey      = Table.AddColumn(Distinct, "WineKey",
>                   each fn_GenerateSurrogateKey({[title]}, "WINE"), type text),
> DedupByKey  = Table.Distinct(AddKey, {"WineKey"}),          // removes case-variant hash collisions
>
> // ❌ Wrong — distinct on natural key before key generation leaves case-variant dupes
> //           that hash to the same key, breaking the relationship
> Distinct    = Table.Distinct(Selected, {"title"}),
> AddKey      = Table.AddColumn(Distinct, "WineKey", ...),    // still has hash duplicates
> ```

---

## Phase 3 — Power Query ETL

> **Power Query M is the ETL layer — and MCP can deliver it entirely without opening the Power Query Editor.** All data shaping, cleaning, type assignments, surrogate key generation, and column derivations happen in M — never in DAX calculated columns. When MCP is connected, use `named_expression_operations` (kind=M) for helper functions and staging queries, and `table_operations` with `mExpression` for dimension and fact tables. Fall back to the Power Query Editor only when MCP is not available.

### 3.1 Architecture Rules

1. **Load each source file once.** Create one `Raw_<Source>` query per file with **Enable Load = off**. All downstream queries reference this staging query — never re-read the file.
2. **One query per output table.** Separate queries for each `Fact_*` and `Dim_*` table, all referencing the Raw query.
3. **Explicit types everywhere.** Every column in every loaded query must have an explicit type transformation. Never rely on Power BI's auto-detect.
4. **No calculated columns in fact tables.** If a value can be derived in Power Query, derive it in Power Query — not in DAX.
5. **Define `fn_GenerateSurrogateKey` first.** All dimension and fact queries depend on it. Create and save the function before creating any other query.
6. **Organise all queries into Query Groups.** Assign every query to a named folder group so the Power Query Editor pane stays navigable as the model grows (see section 3.8).

### 3.2 Load a Source File

> Replace column names, count, and encoding with the actual values from your source file.

```m
// Raw_<SourceName> — staging query, Enable Load = off
// Adjust Encoding: 65001 = UTF-8 (most modern files), 1252 = Windows-1252 (Excel/legacy)
// If encoding is unknown: omit the Encoding parameter and let Power Query auto-detect
let
    Source          = Csv.Document(File.Contents("C:\Data\your-file.csv"),
                          [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.None]),
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    TypedColumns    = Table.TransformColumnTypes(PromotedHeaders,{
                          // Replace with your actual columns and types:
                          {"<KeyColumn>",   Int64.Type},
                          {"<DateColumn>",  type date},
                          {"<TextColumn>",  type text},
                          {"<AmountCol>",   type number}
                      })
in
    TypedColumns
```

### 3.3 Build a Dimension Table

```m
// Dim_<Entity> — reference Raw_<Source>, derive surrogate key
// Replace column names with the actual columns for this dimension
let
    Source       = Raw_<SourceName>,
    SelectedCols = Table.SelectColumns(Source, {
                       "<NaturalKeyCol>", "<AttrCol1>", "<AttrCol2>"
                       // include only columns that belong in this dimension
                   }),
    Distinct     = Table.Distinct(SelectedCols, {"<NaturalKeyCol>"}),
    Sorted       = Table.Sort(Distinct, {{"<NaturalKeyCol>", Order.Ascending}}),
    // Hashed surrogate key — stable across refreshes, same prefix used in fact table
    AddKey       = Table.AddColumn(Sorted, "<Entity>Key",
                       each fn_GenerateSurrogateKey({[<NaturalKeyCol>]}, "<PREFIX>"),
                       type text),
    Reordered    = Table.ReorderColumns(AddKey,
                       List.Combine({{"<Entity>Key"},
                           List.RemoveItems(Table.ColumnNames(AddKey), {"<Entity>Key"})}))
in
    Reordered
```

### 3.4 Build a Fact Table

```m
// Fact_<Subject> — reference Raw_<Source>, compute all foreign keys directly
// No Table.NestedJoin — recompute hashes using same natural keys and prefixes as dimensions
let
    Source       = Raw_<SourceName>,
    FactCols     = Table.SelectColumns(Source, {
                       "<GrainKeyCol>",
                       "<DateCol>",
                       "<NaturalKeyForDim1>",
                       "<NaturalKeyForDim2>",
                       "<Measure1>",
                       "<Measure2>"
                       // include only fact-level columns: keys + additive measures
                   }),
    // Recompute each dimension foreign key using same inputs as that dimension's surrogate key
    AddDim1Key   = Table.AddColumn(FactCols, "<Dim1>Key",
                       each fn_GenerateSurrogateKey({[<NaturalKeyForDim1>]}, "<PREFIX1>"), type text),
    AddDim2Key   = Table.AddColumn(AddDim1Key, "<Dim2>Key",
                       each fn_GenerateSurrogateKey({[<NaturalKeyForDim2>]}, "<PREFIX2>"), type text),
    // DateKey as integer — faster for relationships than text hash
    AddDateKey   = Table.AddColumn(AddDim2Key, "DateKey",
                       each Date.Year([<DateCol>])*10000 + Date.Month([<DateCol>])*100 + Date.Day([<DateCol>]),
                       Int64.Type),
    // Drop natural key columns — fact table must contain only surrogate keys + measures
    DropNatural  = Table.RemoveColumns(AddDateKey,
                       {"<NaturalKeyForDim1>", "<NaturalKeyForDim2>", "<DateCol>"}),
    // ⚠️ Always use JoinKind.LeftOuter — JoinKind.Left is NOT valid M syntax and causes a refresh error
    FinalTypes   = Table.TransformColumnTypes(DropNatural, {
                       {"<GrainKeyCol>", Int64.Type},
                       {"<Measure1>",    type number},
                       {"<Measure2>",    Int64.Type},
                       {"<Dim1>Key",     type text},
                       {"<Dim2>Key",     type text},
                       {"DateKey",       Int64.Type}
                   })
in
    FinalTypes
```

### 3.5 Common Cleaning Patterns

```m
// Standardize text casing
CleanCol = Table.TransformColumns(Source, {{"<Col>", Text.Proper}}),

// Parse non-standard date format
ParsedDate = Table.TransformColumns(Source, {{"<DateCol>",
    each try Date.FromText(_, [Format="dd-MMM-yyyy"]) otherwise null, type date}}),

// Split a combined "City, State" column
SplitCityState = Table.SplitColumn(Source, "<CityStateCol>",
    Splitter.SplitTextByDelimiter(", ", QuoteStyle.None), {"City", "State"}),

// Replace empty strings with null
NullBlanks = Table.ReplaceValue(Source, "", null, Replacer.ReplaceValue,
    Table.ColumnNames(Source)),

// Derive a classification band — replace thresholds and labels for your data
AddBand = Table.AddColumn(Source, "<BandColName>",
    each if [<NumericCol>] >= <High> then "<HighLabel>"
         else if [<NumericCol>] >= <Mid>  then "<MidLabel>"
         else "<LowLabel>", type text)
```

### 3.5a Excel Multi-Header Row Pitfall

> ⚠️ **Some Excel sheets have a label row above the actual column header row.** For example, a wide/pivoted table where row 1 contains a merged cell label (e.g., "Downtime factor") and row 2 contains the real column names (e.g., "Batch", "1", "2", ... "12"). Using `Excel.Workbook(..., null, true)` (`useHeaders=true`) consumes row 1 as Power Query's column names — but those names are meaningless. Use `useHeaders=false`, then `Table.Skip(1)` to discard the label row, then `Table.PromoteHeaders` to promote the real header row.

```m
// ✅ Correct — skip merged/label row, then promote real header row
Source          = Excel.Workbook(File.Contents("..."), null, false),  // useHeaders=false
Sheet           = Source{[Item="<SheetName>",Kind="Sheet"]}[Data],
SkipLabelRow    = Table.Skip(Sheet, 1),          // discard merged/label header row
Promoted        = Table.PromoteHeaders(SkipLabelRow, [PromoteAllScalars=true]),

// ❌ Wrong — useHeaders=true consumes the label row, leaving real headers as first data row
Source          = Excel.Workbook(File.Contents("..."), null, true),   // useHeaders=true
Sheet           = Source{[Item="<SheetName>",Kind="Sheet"]}[Data],
Promoted        = Table.PromoteHeaders(Sheet, ...)  // promotes real data as headers, not the label row
```

**How to detect this in Phase 1:** Open the Excel file. If the row containing actual column names is row 2 or lower, the sheet has a label row above it.

### 3.5b Integer Column Names After `Table.PromoteHeaders`

> ⚠️ **`Table.PromoteHeaders` with `PromoteAllScalars=true` preserves the original cell type for column names.** If a header row contains integer values (e.g., `1`, `2`, ... `12` in a pivoted table), the resulting column names are integer-typed, not text. `Table.TransformColumnTypes` lookups like `{"1", type number}` will then fail at refresh with `[DataFormat.Error] We couldn't convert to Number` because the column is actually named the integer `1`, not the text `"1"`.

**Fix:** Immediately after `Table.PromoteHeaders`, rename all columns by position before any type transformation:

```m
// ✅ Correct — rename integer-named columns by position before TransformColumnTypes
Promoted        = Table.PromoteHeaders(SkipLabelRow, [PromoteAllScalars=true]),
AllCols         = Table.ColumnNames(Promoted),
RenameList      = List.Transform(List.Positions(AllCols), each
                      if _ = 0 then {AllCols{_}, "<KeyColName>"}
                      else {AllCols{_}, Text.From(_)}),  // integers -> "1", "2" ...
Renamed         = Table.RenameColumns(Promoted, RenameList),
TypedKey        = Table.TransformColumnTypes(Renamed, {{"<KeyColName>", Int64.Type}}),

// Then unpivot using the now-text column names:
Unpivoted       = Table.UnpivotOtherColumns(TypedKey, {"<KeyColName>"}, "CategoryNum", "Value")
```

| Pattern | Error trigger | Correct replacement |
|---|---|---|
| `{"a".."z"}` list range operator | `{"a".."z"}`, `{"0".."9"}`, any `{"x".."y"}` range | `Text.ToList("abcdefghijklmnopqrstuvwxyz")` or `Character.ToNumber(_) >= 97 and Character.ToNumber(_) <= 122` |
| `type {type text}` inline list type | Used as the type parameter of `Table.AddColumn` | Use `type list` for list columns |
| Nested `let … in` inside an `each` lambda | `each let x = … in x` as a column expression | Extract nested logic into a separate query step **before** the `Table.AddColumn` call |
| Measure references in `CALCULATE` boolean filters | `CALCULATE([MyMeasure], [Price] <= [Budget Measure])` | Store the measure value in a `VAR` first, then use `FILTER(ALLSELECTED(Table), [Column] <= _var)` |

**Compliant patterns — use these instead:**

```m
// ✅ Allowed — character range via Text.ToList (no ".." operator)
Alphabet = Text.ToList("abcdefghijklmnopqrstuvwxyz "),
CleanText = Text.Select(Text.Lower([description]), Alphabet),

// ✅ Allowed — character range via Character.ToNumber comparison
CleanText = Text.Select(Text.Lower([description]), each
    (Character.ToNumber(_) >= 97 and Character.ToNumber(_) <= 122) or _ = " "),

// ✅ Allowed — type list instead of type {type text}
AddWords = Table.AddColumn(Source, "Words",
    each Text.Split([CleanDesc], " "),
    type list),

// ✅ Allowed — pre-clean then split in separate steps (avoid nested let/in inside each)
AddClean = Table.AddColumn(Source, "CleanDesc",
    each Text.Select(Text.Lower([description]), Alphabet), type text),
AddWords = Table.AddColumn(AddClean, "Words",
    each Text.Split([CleanDesc], " "), type list),
```

**Rule of thumb:** Never write a `let … in` expression as the value argument of `each` in `Table.AddColumn`. Always break complex multi-step per-row logic into separate query steps that operate on the full table.

### 3.5 Query Groups — Organise the Queries Pane

Assign every query to a **Query Group** (folder) in the Power Query Queries pane. Groups are visual only — they do not affect query logic or load order. Apply this standard grouping to every model:

| Group name | Queries that belong here |
|---|---|
| `🔧 Functions` | `fn_GenerateSurrogateKey` and any other helper functions |
| `📥 Staging` | All `Raw_*` queries (Enable Load = off) |
| `📅 Dim Tables` | All `Dim_*` queries |
| `📊 Fact Tables` | All `Fact_*` queries |
| `📦 Parameters` | Any Power Query `Parameter` objects (file paths, server names, date ranges) |

> Add sub-groups (e.g. `Dim Tables / Geography`, `Dim Tables / Time`) only when a group contains more than 8 queries.

**Setting Query Groups via MCP (`named_expression_operations` — for staging/function queries):**

Named expressions do not directly support folder metadata. Set the group using the Power Query Editor UI for manually-created queries, or apply it after Close & Apply via MCP `named_expression_operations` Update by setting the `queryGroup` property if your MCP version supports it.

**Setting Query Groups in Power Query Editor (manual path):**
1. Right-click the query in the Queries pane → **Move to Group → New Group**
2. Type the group name from the table above
3. Repeat for all queries — this takes under 2 minutes for a typical 10-query model

**MCP `named_expression_operations` queryGroup field:**

When creating named expressions via MCP, include the `queryGroup` property to assign the folder at creation time:
```json
{
  "name": "Raw_Sales",
  "kind": "M",
  "queryGroup": "📥 Staging",
  "expression": "..."
}
```
> **Note:** `queryGroup` support depends on MCP server version. If the field is rejected, fall back to setting groups manually in the Power Query Editor after Close & Apply.

---

### 3.6 Delivering Power Query via MCP (Preferred when MCP is connected)

When MCP is connected, you do not need to open the Power Query Editor at all. Deliver the entire ETL layer via MCP using two operations:

**Step 1 — Create the helper function and staging queries as Named Expressions**

Use `named_expression_operations` with `kind: "M"` for:
- `fn_GenerateSurrogateKey` — the hashed key helper function (referenced by all dim/fact expressions by name)
- `Raw_<Source>` — each staging query that reads a source file (named expressions have Enable Load off automatically)

```
MCP named_expression_operations → Create:
  name: "fn_GenerateSurrogateKey",  kind: "M",  expression: <full M function body>
  name: "Raw_CustomerChurn",        kind: "M",  expression: <Csv.Document M code with typed columns>
```

**Step 2 — Create dimension and fact tables via table_operations with mExpression**

Use `table_operations` Create with `mExpression` for every `Dim_*` and `Fact_*` table. The expression references `fn_GenerateSurrogateKey` and `Raw_*` staging queries by name — no file paths needed in the table expression.

> **Critical MCP rules for M-based tables:**
> - Always provide the full `columns` array with `name`, `dataType`, and `sourceColumn` — the engine cannot auto-infer schema from `mExpression`
> - Never set `isKey: true` on Import mode tables — it causes an error
> - Use `isHidden: true` on surrogate key columns instead of `isKey`
> - Compatibility level 1600 does not support `function_operations` — always use `named_expression_operations` with `kind: "M"` for helper functions

### 3.7 Set Query Groups After Close & Apply (MCP path)

When the ETL was delivered entirely via MCP, set Query Groups in the Power Query Editor with a single pass:
1. **Transform Data** → Power Query Editor opens
2. For each query, right-click → **Move to Group** → select or create the group from the table in 3.5
3. **Close & Apply** again — no query logic changes, only folder metadata

This takes under 2 minutes and is the only manual step in the Phase 3 MCP path.

---

### 3.8 Close and Apply (Power Query Editor path only)

Use the Power Query Editor manually only when MCP is not connected:

1. Open Power Query Editor (Transform Data)
2. Define `fn_GenerateSurrogateKey`, all `Raw_*`, `Dim_*`, and `Fact_*` queries using the M code from sections 3.2–3.5
3. **File > Close & Apply** — wait for all queries to load without errors
4. Connect MCP and continue from Phase 4

---

## Phase 4 — Scaffold and Wire the Model via MCP

> **MCP is now the primary tool.** Every model operation from this point forward is executed via MCP unless it explicitly requires the Desktop UI. The schema you designed in Phase 2 is your source of truth — translate it directly into MCP prompts.

### 4.1 Connect to the Model

```
Prompt to MCP:
"Search for and connect to the Power BI Desktop file [your-filename.pbix]"
```

### 4.2 Disable Auto Date/Time

Auto Date/Time creates hidden date tables for every date column, bloating the model. Always disable it.

```
Prompt to MCP:
"Disable the Auto Date/Time setting on this semantic model"
```

### 4.3 Scaffold All Tables

> **Replace the table list with the tables you designed in Phase 2.** This prompt must exactly match your star schema ERD. Do not add tables that don't exist in your design, and do not omit any that do.

```
Prompt to MCP:
"Create the following empty tables in the semantic model.
This is a Kimball star schema — one or more fact tables and their associated dimensions.
Tables:
- <Fact table 1> (fact table — will be populated via Power Query)
- <Fact table 2> (fact table — if applicable)
- Dim_Date (dimension — will be populated via Power Query M code)
- <Dim table 1> (dimension)
- <Dim table 2> (dimension)
- [add all dimension tables from your Phase 2 schema design]
- _Measures (no columns — DAX measure container only)"
```

### 4.4 Create All Relationships

> **Replace the relationship list with the relationships from your Phase 2 ERD.** Every fact foreign key must map to a dimension surrogate key. Text-type keys (Base64 hashed) join text-to-text. `DateKey` joins Int64-to-Int64. All relationships are one-to-many, single direction, active — unless you have a legitimate role-playing dimension (use inactive + USERELATIONSHIP) or a many-to-many via bridge.

```
Prompt to MCP:
"Create the following relationships in the semantic model.
All are one-to-many, single direction filter (dimension filters fact), active.

[List every relationship from your Phase 2 design in this format:]
1. Dim_Date[DateKey] (Int64) → <FactTable>[DateKey] (Int64)
2. <DimTable1>[<DimTable1>Key] (text) → <FactTable>[<DimTable1>Key] (text)
3. <DimTable2>[<DimTable2>Key] (text) → <FactTable>[<DimTable2>Key] (text)
[... continue for all relationships ...]

If any fact table has a second date column (e.g., a close date, discharge date, arrival date),
add one inactive relationship for it:
N. Dim_Date[DateKey] → <FactTable>[<SecondDateKey>], inactive"
```

### 4.5 Mark the Date Table

```
Prompt to MCP:
"Mark Dim_Date as the Date Table using the [Date] column as the date key"
```

### 4.6 Hide Surrogate Key Columns

```
Prompt to MCP:
"Hide from Report view all columns that end in 'Key' across every table in this model.
Also hide any [Order] columns in Field Parameter tables if they exist."
```

---

## Phase 5 — Configure the Model via MCP

### 5.1 Set Column Data Categories

Apply data categories to any geography columns in your dimension tables so Power BI can render map visuals correctly.

```
Prompt to MCP:
"Set data categories on the following columns (replace with columns from your schema):
- <GeoTable>[City]      → City
- <GeoTable>[State]     → State or Province
- <GeoTable>[Country]   → Country
- <GeoTable>[PostCode]  → Postal Code  (if column exists)
- <GeoTable>[Latitude]  → Latitude     (if column exists)
- <GeoTable>[Longitude] → Longitude    (if column exists)"
```

### 5.2 Set Sort-By Columns

Dim_Date columns are always sorted by their numeric counterpart. Apply the same pattern to any other ordinal columns in your dimensions.

> ⚠️ **Sort-by column must have a unique value per text column value.** A common mistake is setting `Dim_Date[Year-Month]` (e.g., "2024-08") to sort by `Dim_Date[DateKey]` (YYYYMMDD integer). `DateKey` has up to 31 distinct values per Year-Month — Power BI rejects this with a many-to-one sort error. Always sort `Year-Month` by a `YearMonthNum` column (`Year * 100 + Month`, e.g., `202408`) which is unique per month.
>
> **Rule:** the sort-by column must have **exactly one distinct value** for every distinct value in the sorted column.

```m
// ✅ Correct — YearMonthNum is unique per Year-Month value
YearMonthNum = Table.AddColumn(MonthShort, "YearMonthNum",
                   each Date.Year([Date]) * 100 + Date.Month([Date]), Int64.Type),
YearMonth    = Table.AddColumn(YearMonthNum, "Year-Month",
                   each Date.ToText([Date], "yyyy-MM", "en-US"), type text),

// ❌ Wrong — DateKey has 28–31 distinct values per Year-Month; sort-by fails
// Do NOT sort Dim_Date[Year-Month] by Dim_Date[DateKey]
```

```
Prompt to MCP:
"Configure sort-by columns:
- Dim_Date[Month Name]  sorted by Dim_Date[Month Number]
- Dim_Date[Month Short] sorted by Dim_Date[Month Number]
- Dim_Date[Day Name]    sorted by Dim_Date[Day of Week]
- Dim_Date[Quarter]     sorted by Dim_Date[Quarter Number]
- Dim_Date[Year-Month]  sorted by Dim_Date[YearMonthNum]   ← required if Year-Month column exists
[Add any other ordinal text columns from your dimensions that need sort-by applied]"
```

### 5.3 Create Hierarchies

Create drill-down hierarchies for every natural hierarchy identified in Phase 1.

```
Prompt to MCP:
"Create the following hierarchies (replace with hierarchies from your Phase 1 analysis):
- Dim_Date: 'Date Hierarchy' with levels Year > Quarter > Month Name > Date
- <GeoDim>: '<Geo> Hierarchy' with levels <TopLevel> > <MidLevel> > <LeafLevel>
- <ProductDim>: '<Product> Hierarchy' with levels <TopLevel> > <MidLevel> > <LeafLevel>
[Add all hierarchies identified in Phase 1.2]"
```

### 5.3a MCP Broken Transaction Recovery

> ⚠️ **Deleting a column that is referenced by a sort-by dependency will fail silently or roll back with the error: `The base version must not be negative when impact is requested for a transaction.`** The model then enters a state where full `model_operations Refresh` fails with the same error on every attempt.

**Root cause:** The MCP `column_operations Delete` operation is rejected by the TOM server (because the column has a sort-by dependency), but the transaction leaves the model version counter in a negative state.

**Recovery procedure — do not retry the full model refresh:**

1. Remove the sort-by dependency **before** deleting the column:
   ```
   MCP column_operations Update → set sortByColumn = null on the text column
   MCP column_operations Delete → now safe to delete the numeric sort column
   ```
2. If the model is already stuck (negative version error), **refresh individual tables** instead of the full model:
   ```
   MCP table_operations Refresh → Dim_Date
   MCP table_operations Refresh → Dim_Product
   ... (each table separately)
   ```
   Individual table refreshes bypass the broken model-level transaction state and reload the data successfully.
3. If individual table refreshes also fail, reconnect MCP (`connection_operations Connect`) and retry the individual table sequence.

**Prevention:** Before deleting any column, check whether it is set as the `sortByColumn` for another column. Remove the sort-by link first, then delete.

### 5.4 Run the Best Practice Analyzer

```
Prompt to MCP:
"Evaluate this model against Power BI modeling best practices.
List all issues found, sorted by severity (high → medium → low).
For each issue, provide the specific object name and recommended fix."
```

### 5.5 Row-Level Security (RLS)

Define RLS whenever the report will be shared with users who must not see each other's data.

**Static RLS (simple, use for prototypes):**
```
Prompt to MCP:
"Create a Row-Level Security role named '<RoleName>'.
Apply this DAX filter on <DimensionTable>:
[<FilterColumn>] = USERPRINCIPALNAME()"
```

**Dynamic RLS (recommended for production — uses a security mapping table):**
```
Prompt to MCP:
"Create a Row-Level Security role named 'DynamicFilter'.
Create a security mapping table named 'UserAccess' with columns:
- [UserEmail] (text)
- [<FilterColumn>] (text) — matches the filter attribute in <DimensionTable>

Apply this DAX filter to <DimensionTable>:
[<FilterColumn>] IN CALCULATETABLE(
    VALUES(UserAccess[<FilterColumn>]),
    UserAccess[UserEmail] = USERPRINCIPALNAME()
)
Create an active one-to-many relationship: UserAccess[<FilterColumn>] → <DimensionTable>[<FilterColumn>]
Hide the UserAccess table from Report view."
```

**RLS checklist:**
- [ ] Roles defined and match the access requirements from Phase 1
- [ ] RLS tested using **View As** (Modeling tab > View As Roles) before publishing
- [ ] Dynamic mapping table excluded from Report view
- [ ] Row counts verified per role — no role should return 0 rows unintentionally

---

## Phase 6 — DAX Measure Library via MCP

> **All measures live in the `_Measures` table.** Every measure needs a Description and a Display Folder. Use `DIVIDE()` everywhere — never bare `/`. All measures are based on the columns that actually exist in your fact table(s) after Phase 3. Replace every table and column reference below with your actual schema.

### 6.0 Display Folders — Always Assign Every Measure to a Folder

Display Folders group measures inside the `_Measures` table in the Fields pane, making the report builder experience navigable as measure count grows. **Every measure must have a `displayFolder` when created via MCP** — do not leave it blank.

**Standard folder taxonomy (adapt names to your dataset's domain):**

| Display Folder | What goes here |
|---|---|
| `📊 Core Metrics` | Primary count and sum measures — the headline numbers |
| `💰 Revenue` | All revenue, charge, refund, and ARPU measures |
| `📈 Rates & Ratios` | Percentage and ratio measures (churn rate, retention rate, share %) |
| `📅 Time Intelligence` | Any standalone time measures not covered by the Calculation Group |
| `👥 Engagement` | Tenure, referrals, usage, activity measures |
| `🔢 Ranking` | RANKX measures, Top N measures |
| `🔧 Utility` | Last Refresh, row counts, and other admin measures |
| `⚙️ What-If` | SELECTEDVALUE measures and scenario calculation measures |

> Folder names with emoji prefixes sort predictably and visually distinguish groups in the Fields pane. Use the same taxonomy across all models in a portfolio for consistency.

**Passing Display Folder in MCP `measure_operations`:**

```json
{
  "name": "Churn Rate %",
  "tableName": "_Measures",
  "expression": "DIVIDE([Total Churned], [Total Customers], 0)",
  "formatString": "#,##0.0%",
  "displayFolder": "📈 Rates & Ratios",
  "description": "Percentage of customers who churned"
}
```

**Nesting sub-folders** (use `\` as separator — only when a folder exceeds ~8 measures):
```json
"displayFolder": "💰 Revenue\\Charges"
```

**MCP prompt pattern for bulk display folder assignment:**
```
Prompt to MCP:
"Create the following measures in _Measures. Assign displayFolder to each measure
as indicated. All measures must also have a description.

Folder: 📊 Core Metrics
  [Total Customers] = COUNTROWS(Fact_Customers)
  [Total Churned]   = SUM(Fact_Customers[Is Churned])
  ...

Folder: 📈 Rates & Ratios
  [Churn Rate %]    = DIVIDE([Total Churned], [Total Customers], 0)
  ..."
```

### 6.1 Create Base Aggregation Measures

> ⚠️ **`_Measures` table partition — use `DATATABLE("_", INTEGER, {{0}})`, not `{}`.**
> When MCP creates the `_Measures` table, the calculated partition expression must produce at least one row. Using an empty DAX table literal `{}` (or `DATATABLE("_", INTEGER, {})`) causes a fatal error on model load:
> ```
> DATATABLE: empty row set is not valid
> ```
> The correct expression uses **double braces** for a single row containing the value `0`:
> ```dax
> // ✅ Correct — one row, double-brace syntax
> DATATABLE("_", INTEGER, {{0}})
>
> // ❌ Wrong — zero rows, causes load error
> DATATABLE("_", INTEGER, {})
>
> // ❌ Wrong — {} alone has no columns
> {}
> ```
> The TMDL `column _` declaration (hidden, Int64, `summarizeBy: none`) **must also be present** in the table's TMDL file to match the column defined in the DATATABLE expression. MCP should emit it automatically; if it is missing, add it manually.

### 6.1 Create Base Aggregation Measures

```
Prompt to MCP:
"Create a table named '_Measures' with no columns (measure container only).
Then create the following DAX measures in _Measures.
Assign each measure to its displayFolder and include a description.

[Replace the examples below with measures derived from your actual fact table columns
and the business questions identified in Phase 1.3]

Example structure — adapt to your data:

Folder: 📊 Core Metrics
[Total <PrimaryMetric>] = SUM(<FactTable>[<MetricColumn>])
Description: Total <metric description> in the current filter context
DisplayFolder: 📊 Core Metrics

[Total <CountEntity>] = DISTINCTCOUNT(<FactTable>[<GrainKey>])
Description: Count of unique <entities>
DisplayFolder: 📊 Core Metrics

Folder: 📈 Rates & Ratios
[<Ratio> %] = DIVIDE([<Numerator Measure>], [<Denominator Measure>], 0)
Description: <Numerator> as a percentage of <Denominator>
DisplayFolder: 📈 Rates & Ratios

[Avg <MetricPerEntity>] = DIVIDE([Total <PrimaryMetric>], [Total <CountEntity>], 0)
Description: Average <metric> per <entity>
DisplayFolder: 📊 Core Metrics"
```

### 6.2 Do NOT Create Time Intelligence Measures Here

Time intelligence (Prior Year, YoY %, YTD, MTD, QTD, MoM, Prior Month) is handled entirely by the Calculation Group in Phase 7. Creating individual time intelligence measures per metric produces unmanageable sprawl (10 base measures × 6 time periods = 60 measures). Skip them here.

### 6.3 Create Ranking and Utility Measures

```
Prompt to MCP:
"Create these utility measures in _Measures.
[Replace dimension and measure references with your actual schema]

[<Entity> Rank] =
    RANKX(ALL(<DimTable>[<AttributeCol>]), [<Primary Measure>], , DESC, DENSE)
Description: Rank of each <entity> by <primary measure> — 1 is highest
DisplayFolder: 🔢 Ranking

[<Entity> Share %] =
    DIVIDE([<Primary Measure>],
        CALCULATE([<Primary Measure>], ALL(<DimTable>)), 0)
Description: This <entity>'s share of the total
DisplayFolder: 📈 Rates & Ratios

[Top N <Entity>] =
    VAR N = SELECTEDVALUE('Top N'[Top N Value], 10)
    RETURN IF([<Entity> Rank] <= N, [<Primary Measure>], BLANK())
Description: <Primary measure> visible only for the top N items — N driven by the Top N parameter
DisplayFolder: 🔢 Ranking

[Last Refresh] =
    CONCATENATE('Data last refreshed: ', FORMAT(NOW(), 'MMM DD, YYYY h:mm AM/PM'))
Description: Timestamp of the most recent data refresh
DisplayFolder: 🔧 Utility"
```

### 6.4 DAX Authoring Rules

```dax
// ALWAYS use DIVIDE — never bare /
Result = DIVIDE([Numerator], [Denominator], BLANK())

// Inactive relationship pattern (role-playing dimensions)
<Measure by SecondDate> =
    CALCULATE([<Base Measure>],
        USERELATIONSHIP(Dim_Date[DateKey], <FactTable>[<SecondDateKey>]))

// SWITCH status label pattern
<Metric> Status =
    SWITCH(TRUE(),
        [<Metric> Change %] >= 0.10, "▲ Strong Growth",
        [<Metric> Change %] >= 0,    "▲ Growth",
        [<Metric> Change %] >= -0.10,"▼ Slight Decline",
                                     "▼ Significant Decline")
```

### 6.5 Validate All Measures

```
Prompt to MCP:
"Execute test DAX queries to validate every measure in _Measures.
Confirm each returns a non-error, non-null value against the loaded data.
For any measure that returns an error or always returns BLANK, report the measure name and the error details."
```

---

## Phase 7 — Calculation Groups via MCP

> Calculation Groups require Compatibility Level 1500+ (Power BI Desktop 2020+). They **cannot** be created in the Desktop UI — they must be created via MCP or Tabular Editor.

A single Calculation Group applies a modifier (e.g., "Prior Year", "YTD") to **any** base measure placed in a visual — eliminating per-metric time intelligence measure duplication entirely.

### 7.1 When to Use Calculation Groups vs Individual Measures

| Scenario | Pattern |
|---|---|
| Time intelligence across multiple base measures | ✅ Calculation Group |
| Format string switching ($, $K, $M, %) | ✅ Calculation Group |
| Currency conversion across multiple metrics | ✅ Calculation Group |
| A single unique calculation with no variants | ✅ Individual measure |
| What-If scenario adjustment | ✅ Individual measure (SELECTEDVALUE pattern) |

### 7.2 Create the Time Intelligence Calculation Group

```
Prompt to MCP:
"Create a Calculation Group named 'Time Intelligence' with:
- Table display name: 'Time Intelligence'
- Calculation item column name: 'Time Period'
- Precedence: 10

Calculation Items:

Name: Actual
Expression: SELECTEDMEASURE()
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Current period value with no time modification

Name: Prior Year
Expression: CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Same measure for the equivalent period in the prior year

Name: YoY Change
Expression: SELECTEDMEASURE() - CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Absolute change versus prior year

Name: YoY %
Expression: VAR PY = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR(Dim_Date[Date])) RETURN DIVIDE(SELECTEDMEASURE() - PY, PY, BLANK())
Format String Expression: \"#,##0.0%\"
Description: Percentage growth versus prior year

Name: YTD
Expression: CALCULATE(SELECTEDMEASURE(), DATESYTD(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Year-to-date cumulative value

Name: QTD
Expression: CALCULATE(SELECTEDMEASURE(), DATESQTD(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Quarter-to-date cumulative value

Name: MTD
Expression: CALCULATE(SELECTEDMEASURE(), DATESMTD(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Month-to-date cumulative value

Name: Prior Month
Expression: CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH(Dim_Date[Date]))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Value for the prior calendar month

Name: MoM %
Expression: VAR PM = CALCULATE(SELECTEDMEASURE(), PREVIOUSMONTH(Dim_Date[Date])) RETURN DIVIDE(SELECTEDMEASURE() - PM, PM, BLANK())
Format String Expression: \"#,##0.0%\"
Description: Percentage growth versus prior month

Name: YTD Prior Year
Expression: CALCULATE(SELECTEDMEASURE(), DATESYTD(SAMEPERIODLASTYEAR(Dim_Date[Date])))
Format String Expression: SELECTEDMEASUREFORMATSTRING()
Description: Year-to-date value for the prior year"
```

### 7.3 Create the Format Switcher Calculation Group

```
Prompt to MCP:
"Create a Calculation Group named 'Format Switcher' with:
- Table display name: 'Format Switcher'
- Calculation item column name: 'Number Format'
- Precedence: 5

Calculation Items:

Name: Default
Expression: SELECTEDMEASURE()
Format String Expression: SELECTEDMEASUREFORMATSTRING()

Name: Currency $
Expression: SELECTEDMEASURE()
Format String Expression: \"$#,##0\"

Name: Currency $K
Expression: SELECTEDMEASURE() / 1000
Format String Expression: \"$#,##0.0K\"

Name: Currency $M
Expression: SELECTEDMEASURE() / 1000000
Format String Expression: \"$#,##0.0M\"

Name: Percentage
Expression: SELECTEDMEASURE()
Format String Expression: \"#,##0.0%\"

Name: Integer
Expression: SELECTEDMEASURE()
Format String Expression: \"#,##0\""
```

> **Note on $K and $M items:** These divide the actual numeric value (not just the display). This is mathematically correct for most visuals. However, avoid them on measures where the raw number must be returned accurately (e.g., XMLA consumers or API queries). For display-only scaling with no value change, apply a format string directly to the individual measure instead.

### 7.4 Verify Calculation Groups

```
Prompt to MCP:
"Verify both Calculation Groups 'Time Intelligence' and 'Format Switcher' exist in the model.
Execute a test DAX query applying each item in 'Time Intelligence' to [<your primary measure>]
for the most recent 12 months of data. Confirm no errors.
Confirm Precedence: Time Intelligence = 10, Format Switcher = 5."
```

### 7.5 How to Use Calculation Groups in Visuals

```
// Use 'Time Intelligence'[Time Period] as:
// - Legend → shows multiple time periods as series on a line chart
// - Rows in a matrix → compares time periods down rows
// - A slicer → user picks which period to display

// Use 'Format Switcher'[Number Format] in a slicer
// so users can toggle between $, $K, $M, % on any visual

// Matrix showing Actual + YoY % side by side:
// Rows: Dim_Date[Month Name]
// Columns: Time Intelligence[Time Period]  (filter slicer to Actual and YoY %)
// Values: [<Your Primary Measure>]
```

---

## Phase 8 — Field Parameters and What-If Parameters (Fully via MCP)

> **All parameters are created entirely via MCP — no Desktop UI required.** Both Field Parameters and What-If Parameters are DAX calculated tables with specific annotations. MCP creates them using `table_operations` (Create with `daxExpression`), `table_operations` Update (to add annotations), and `column_operations` Update (to hide Order columns and set `SummarizationSetBy`).

### 8.1 Create Field Parameters via MCP

Field Parameters are DAX calculated tables using `SELECTCOLUMNS` + `NAMEOF()`. Replace all column and measure references with objects from your actual schema.

**Dimension Selector:**

```
Step 1 — MCP table_operations → Create:
  name: "Dimension Selector"
  daxExpression:
    SELECTCOLUMNS(
        {
            ("<Label1>", NAMEOF('<Dim1>'[<Attribute1>]), 0),
            ("<Label2>", NAMEOF('<Dim2>'[<Attribute2>]), 1),
            ("<Label3>", NAMEOF('Dim_Date'[Month Name]),  2)
            // add all dimensions meaningful for analysis in your dataset
        },
        "Dimension Selector",        [Value1],
        "Dimension Selector Fields", [Value2],
        "Dimension Selector Order",  [Value3]
    )

Step 2 — MCP table_operations → Update (add annotation):
  name: "Dimension Selector"
  annotations: [{ key: "PBI_ResultType", value: "FieldParameter" }]

Step 3 — MCP column_operations → Update (hide Order column, mark SummarizationSetBy):
  tableName: "Dimension Selector", columnName: "Dimension Selector Order",  isHidden: true
  tableName: "Dimension Selector", columnName: "Dimension Selector",        annotations: [{ key: "SummarizationSetBy", value: "Automatic" }]
  tableName: "Dimension Selector", columnName: "Dimension Selector Fields", annotations: [{ key: "SummarizationSetBy", value: "Automatic" }]
```

**Measure Selector** — same three-step pattern, replacing column list with measures from `_Measures`:

```
Step 1 — MCP table_operations → Create:
  name: "Measure Selector"
  daxExpression:
    SELECTCOLUMNS(
        {
            ("<Label1>", NAMEOF('_Measures'[<Measure1>]), 0),
            ("<Label2>", NAMEOF('_Measures'[<Measure2>]), 1),
            ("<Label3>", NAMEOF('_Measures'[<Measure3>]), 2)
            // only measures that make sense on the same axis
        },
        "Measure Selector",        [Value1],
        "Measure Selector Fields", [Value2],
        "Measure Selector Order",  [Value3]
    )

Step 2 — MCP table_operations → Update (add annotation):
  name: "Measure Selector"
  annotations: [{ key: "PBI_ResultType", value: "FieldParameter" }]

Step 3 — MCP column_operations → Update:
  tableName: "Measure Selector", columnName: "Measure Selector Order", isHidden: true
  (same SummarizationSetBy annotations as Dimension Selector)
```

### 8.2 Create What-If Numeric Parameters via MCP

What-If parameters are DAX calculated tables using `GENERATESERIES`. After creation, add the `PBI_NavigationStepName = Navigation` annotation and `ParameterMetadata` on the value column.

```
Step 1 — MCP table_operations → Create:
  name: "Top N"
  daxExpression: SELECTCOLUMNS(GENERATESERIES(1, 50, 1), "Top N", [Value])

Step 2 — MCP table_operations → Update (add annotation):
  name: "Top N"
  annotations: [{ key: "PBI_NavigationStepName", value: "Navigation" }]

Step 3 — MCP column_operations → Update:
  tableName: "Top N", columnName: "Top N"
  annotations: [{ key: "ParameterMetadata", value: "{\"version\":3,\"kind\":0}" }]
```

Repeat for each scenario parameter. Define `GENERATESERIES(min, max, step)` from your Phase 1 business questions:

| Parameter Name | DAX GENERATESERIES | Default (in measure) | Use case |
|---|---|---|---|
| `Top N` | `GENERATESERIES(1, 50, 1)` | 10 | Limit ranked visuals to top N items |
| `<Scenario Param>` | `GENERATESERIES(<min>, <max>, <step>)` | `<default>` | `<what this models>` |

Then create a `SELECTEDVALUE` measure in `_Measures` for each parameter:

```
MCP measure_operations → Create in _Measures:
  [Top N Value]     = SELECTEDVALUE('Top N'[Top N], 10)
  [<Param> Value]   = SELECTEDVALUE('<Param Name>'[<Param Name>], <default>)
```

### 8.3 Verify and Finalise via MCP

```
Prompt to MCP:
"Verify the Field Parameter tables 'Dimension Selector' and 'Measure Selector' exist in the model.
Verify PBI_ResultType = FieldParameter annotation is set on both tables.
Verify all What-If parameter tables exist with PBI_NavigationStepName = Navigation annotation.
Hide the [Order] column from all parameter tables in Report view.
Confirm all SELECTEDVALUE What-If value measures exist in _Measures."
```

### 8.4 Create What-If Scenario Measures via MCP

> ⚠️ **Never use a measure reference directly in a `CALCULATE` boolean filter argument for What-If measures.** The Mashup Preview engine (CL 1600) rejects this pattern and produces a load error. Always capture the parameter value in a `VAR` first, then use `FILTER(ALLSELECTED(...), ...)`:
>
> ```dax
> // ❌ Wrong — measure reference in CALCULATE boolean filter, causes load error
> [Reviews in Budget] =
>     CALCULATE([Total Reviews],
>         Fact_Reviews[Price] <= [Price Budget Value],
>         NOT ISBLANK(Fact_Reviews[Price]))
>
> // ✅ Correct — VAR captures the parameter value, FILTER used instead of boolean
> [Reviews in Budget] =
>     VAR _budget = SELECTEDVALUE('Price Budget'[Price Budget], 500)
>     VAR _score  = SELECTEDVALUE('Min Score'[Min Score], 80)
>     RETURN
>         CALCULATE(
>             [Total Reviews],
>             FILTER(
>                 ALLSELECTED(Fact_Reviews),
>                 Fact_Reviews[Price] <= _budget
>                     && Fact_Reviews[Points] >= _score
>                     && NOT ISBLANK(Fact_Reviews[Price])
>             )
>         )
> ```
>
> This applies to **all** What-If scenario measures that filter a fact table column against a parameter. Always: `VAR _param = SELECTEDVALUE(...)` → `FILTER(ALLSELECTED(<FactTable>), <ColumnRef> <= _param)`.

```
Prompt to MCP:
"Create these What-If measures in _Measures.
[Replace with measures appropriate to your scenario parameters and business questions from Phase 1.3]

Assign displayFolder: ⚙️ What-If to all measures in this section.

Example patterns:

[Adjusted <Metric>] =
    [<Base Measure>] * (1 + SELECTEDVALUE('<Param Name>'[<Param Name> Value], 0))
Description: <Base metric> adjusted by the <Param Name> slider
DisplayFolder: ⚙️ What-If

[Target Attainment %] =
    DIVIDE([<Primary Measure>],
        SELECTEDVALUE('<Target Param>'[<Target Param> Value], 1), 0)
Description: Actual <metric> as a percentage of the target
DisplayFolder: ⚙️ What-If

[Gap to Target] =
    SELECTEDVALUE('<Target Param>'[<Target Param> Value], 0) - [<Primary Measure>]
Description: Shortfall between current <metric> and the target
DisplayFolder: ⚙️ What-If"
```

---

## Phase 9 — Multi-Page Report ⚠️ ONLY MANUAL PHASE

> ⚠️ **This is the ONLY phase that requires manual work in Power BI Desktop.** Everything before this — Power Query ETL (Phase 3), model scaffolding (Phase 4), configuration (Phase 5), DAX measures (Phase 6), calculation groups (Phase 7), and all parameters (Phase 8) — is fully automated via MCP. Phase 9 report canvas work — visual placement, slicer layout, theme application, conditional formatting, and cross-filter interaction configuration — must be done manually in the Desktop Report view. Do not start Phase 9 until all MCP phases are complete and the model has been refreshed successfully in Desktop.

> **Before opening the Desktop Report view, complete Phase 9.0.** Generate all SVG icons and the cover page hero image first — these must be written to `assets/icons/` and ready before you start placing Image visuals and Button icons on the canvas.

### 9.0 SVG Icons and Hero Image — Generate Before Building Pages

> Generate all SVG assets **before** starting canvas layout. SVGs are resolution-independent, fully themeable, and can power both static decorative visuals and dynamic DAX-driven status indicators. Every icon must be **semantically matched to the actual report subject and page purpose** derived from Phase 1 — never use generic placeholder icons.

**Asset inventory — generate one of each before Phase 9.1:**

| File | ViewBox | Use | How inserted in Power BI |
|---|---|---|---|
| `assets/icons/hero-cover.svg` | `0 0 1200 400` | Full-width thematic hero for the Cover page | Insert → Image |
| `assets/icons/icon-<page-slug>.svg` | `0 0 24 24` | One per report page — page navigation buttons | Insert → Button → Blank → Format → Icon → Custom |
| `assets/icons/kpi-up.svg` | `0 0 24 24` | Positive KPI direction indicator | DAX base64 data URL measure |
| `assets/icons/kpi-down.svg` | `0 0 24 24` | Negative KPI direction indicator | DAX base64 data URL measure |
| `assets/icons/kpi-neutral.svg` | `0 0 24 24` | Flat/neutral KPI direction indicator | DAX base64 data URL measure |

> Name each page icon file `icon-<page-slug>.svg` where `<page-slug>` is the page name lowercased with spaces replaced by hyphens (e.g., `icon-churn-analysis.svg`, `icon-revenue.svg`).

---

#### Icon Derivation Rules — Always Apply Before Generating Any SVG

**Step 1 — Read Phase 1 to identify the report domain and page themes.**

The report subject determines the visual language of ALL icons. Map the domain to an appropriate metaphor before drawing any paths:

| Domain | Hero image metaphor | Page icon language |
|---|---|---|
| Telecom / communications | Signal tower, radiating arcs, network nodes | Signal bars, mobile handset, network grid |
| Healthcare / hospital | EKG heartbeat line, medical cross, stethoscope | Patient bed, heartbeat arc, shield |
| Retail / e-commerce | Shopping cart, product grid, storefront, price tags | Cart, tag, product box |
| Finance / banking | Coin stacks, rising chart, currency symbol | Coins, bar chart, percent badge |
| HR / workforce | People silhouettes, org tree, briefcase | Person icon, org node, briefcase |
| Manufacturing / supply chain | Gear, factory outline, conveyor belt | Cog, package, chain link |
| Marketing / campaigns | Megaphone, funnel, target/bullseye | Funnel, target, wave |
| Transport / logistics | Vehicle silhouette, route map, map pin | Vehicle, route arrow, map pin |
| Energy / utilities | Lightning bolt, meter gauge, power grid | Bolt, gauge, plug |
| Education | Graduation cap, open book, growth curve | Cap, book, pencil |

**Step 2 — Assign a specific icon concept to every page from Phase 1.3 page themes.**

The icon must represent the analytical *question* on that page, not just a generic chart shape:
- ❌ "bar chart icon for the breakdown page" — too generic
- ✅ "funnel icon for the campaign conversion breakdown page" — semantically meaningful
- ✅ "door-with-exit-arrow icon for the churn analysis page" — maps to the business concept

**Step 3 — Use the templates below as structural starting points, then replace `<path>` elements with domain-specific shapes.**

---

#### Cover Page Hero SVG Template

The hero spans the full (or half) width of the cover page. Use geometric abstraction — not clip art — to evoke the domain. Reserve the left ~45% of the canvas for Power BI text visuals (title, subtitle, date).

```svg
<!-- assets/icons/hero-cover.svg                                              -->
<!-- CUSTOMISE: Replace bar shapes and trend line with domain-appropriate     -->
<!-- illustration (e.g. signal arcs for telecom, EKG line for healthcare).   -->
<!-- Replace #1F4E79 with your theme's primary color.                        -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1200 400">

  <!-- Solid background — match your report theme primary color -->
  <rect width="1200" height="400" fill="#1F4E79"/>

  <!-- Left text-area separator (Power BI text visuals go here) -->
  <rect x="0" y="0" width="540" height="400" fill="#1F4E79"/>
  <line x1="548" y1="40" x2="548" y2="360"
        stroke="white" stroke-width="1.5" stroke-dasharray="4,8" opacity="0.25"/>

  <!-- Subtle background grid (right half) -->
  <g opacity="0.10" stroke="white" stroke-width="0.75" fill="none">
    <line x1="560" y1="60"  x2="1160" y2="60"/>
    <line x1="560" y1="140" x2="1160" y2="140"/>
    <line x1="560" y1="220" x2="1160" y2="220"/>
    <line x1="560" y1="300" x2="1160" y2="300"/>
    <line x1="640"  y1="30" x2="640"  y2="370"/>
    <line x1="780"  y1="30" x2="780"  y2="370"/>
    <line x1="920"  y1="30" x2="920"  y2="370"/>
    <line x1="1060" y1="30" x2="1060" y2="370"/>
  </g>

  <!-- Domain illustration area — REPLACE these shapes with your subject metaphor -->
  <!-- Default: abstract bar-chart silhouette + trend line overlay              -->
  <g fill="white" opacity="0.13">
    <rect x="590" y="230" width="55" height="120" rx="4"/>
    <rect x="675" y="170" width="55" height="180" rx="4"/>
    <rect x="760" y="110" width="55" height="240" rx="4"/>
    <rect x="845" y="155" width="55" height="195" rx="4"/>
    <rect x="930" y="90"  width="55" height="260" rx="4"/>
    <rect x="1015" y="185" width="55" height="165" rx="4"/>
    <rect x="1100" y="130" width="55" height="220" rx="4"/>
  </g>

  <!-- Trend line + data-point dots -->
  <polyline
    points="617,290 702,230 787,170 872,215 957,150 1042,245 1127,190"
    fill="none" stroke="white" stroke-width="3"
    stroke-linecap="round" stroke-linejoin="round" opacity="0.65"/>
  <g fill="white" opacity="0.85">
    <circle cx="617"  cy="290" r="5"/>
    <circle cx="702"  cy="230" r="5"/>
    <circle cx="787"  cy="170" r="5"/>
    <circle cx="872"  cy="215" r="5"/>
    <circle cx="957"  cy="150" r="5"/>
    <circle cx="1042" cy="245" r="5"/>
    <circle cx="1127" cy="190" r="5"/>
  </g>

  <!-- Accent highlight circle — move/resize to suit your domain illustration -->
  <circle cx="957" cy="150" r="14"
          fill="none" stroke="white" stroke-width="2" opacity="0.5"/>

</svg>
```

> **Telecom example:** Replace the bar/trend elements with concentric signal arcs centred on a tower point, plus scattered network-node dots connected by thin lines.
> **Healthcare example:** Replace with a continuous EKG sine-spike line running across the full width, and a bold medical cross on the right third.
> **Retail example:** Replace with a grid of small product-box rectangles (right half) and a partial shopping-cart outline arc (centre).

---

#### Standard Page Navigation Icons (24 × 24)

Style conventions applied to every page icon:
- `viewBox="0 0 24 24"` · `fill="none"` · `stroke="#1F4E79"` (or `currentColor`)
- `stroke-width="1.5"` · `stroke-linecap="round"` · `stroke-linejoin="round"`

**These are structural starting points. Replace paths with domain-matched shapes when the report subject offers a richer metaphor than a generic chart icon.**

```svg
<!-- icon-cover.svg — Home / back-to-cover navigation button (all pages) -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M3 10L12 3L21 10V21H15V15H9V21H3V10z"/>
</svg>
```

```svg
<!-- icon-overview.svg — Executive Summary / Overview page              -->
<!-- Domain swap example: 4-bar signal-strength meter for telecom       -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <rect x="3"  y="3"  width="7" height="7" rx="1"/>
  <rect x="14" y="3"  width="7" height="7" rx="1"/>
  <rect x="3"  y="14" width="7" height="7" rx="1"/>
  <rect x="14" y="14" width="7" height="7" rx="1"/>
</svg>
```

```svg
<!-- icon-trend.svg — Trend / Time Analysis page -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <polyline points="3,17 7,12 12,14 17,8 21,10"/>
  <line x1="3"  y1="21" x2="21" y2="21"/>
  <line x1="3"  y1="3"  x2="3"  y2="21"/>
</svg>
```

```svg
<!-- icon-breakdown.svg — Dimensional Breakdown page -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <rect x="3"  y="13" width="4" height="8" rx="1"/>
  <rect x="10" y="7"  width="4" height="14" rx="1"/>
  <rect x="17" y="10" width="4" height="11" rx="1"/>
  <line x1="2" y1="21" x2="22" y2="21"/>
</svg>
```

```svg
<!-- icon-ranking.svg — Ranking / Detail / Top N page -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <line x1="9"  y1="6"  x2="20" y2="6"/>
  <line x1="9"  y1="12" x2="20" y2="12"/>
  <line x1="9"  y1="18" x2="20" y2="18"/>
  <circle cx="4.5" cy="6"  r="1.5" fill="#1F4E79" stroke="none"/>
  <circle cx="4.5" cy="12" r="1.5" fill="#1F4E79" stroke="none"/>
  <circle cx="4.5" cy="18" r="1.5" fill="#1F4E79" stroke="none"/>
</svg>
```

```svg
<!-- icon-scenario.svg — What-If / Scenario page -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <line x1="4" y1="6"  x2="20" y2="6"/>
  <circle cx="9"  cy="6"  r="2"/>
  <line x1="4" y1="12" x2="20" y2="12"/>
  <circle cx="15" cy="12" r="2"/>
  <line x1="4" y1="18" x2="20" y2="18"/>
  <circle cx="11" cy="18" r="2"/>
</svg>
```

```svg
<!-- icon-quality.svg — Data Quality / Glossary / Definitions page -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none"
     stroke="#1F4E79" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <path d="M12 2L4 6v6c0 5 3.5 9.7 8 11 4.5-1.3 8-6 8-11V6L12 2z"/>
  <polyline points="9,12 11,14 15,10"/>
</svg>
```

---

#### KPI Status Icons — Universal (Same for All Reports)

Used in DAX measures to render inline direction indicators. No domain customisation needed.

```svg
<!-- kpi-up.svg — positive direction (green upward triangle) -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <path d="M12 5L20 19H4L12 5Z" fill="#107C10"/>
</svg>
```

```svg
<!-- kpi-down.svg — negative direction (red downward triangle) -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <path d="M12 19L4 5H20L12 19Z" fill="#A4262C"/>
</svg>
```

```svg
<!-- kpi-neutral.svg — flat / no significant change (gray dash) -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <rect x="4" y="10" width="16" height="4" rx="2" fill="#605E5C"/>
</svg>
```

**Convert any SVG to a base64 data URL (run once per icon in PowerShell):**

```powershell
# Run from the dashboard project root
$svg = Get-Content "assets/icons/kpi-up.svg" -Raw
$b64 = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($svg))
"data:image/svg+xml;base64,$b64"   # Copy this full string into your DAX measure
```

**DAX KPI direction icon measure (add to `_Measures`, `displayFolder: 🔧 Utility`):**

```dax
KPI Direction Icon =
VAR _change = [YoY Change %]   -- replace with your period-comparison measure
VAR _up     = "data:image/svg+xml;base64,<paste kpi-up base64 here>"
VAR _down   = "data:image/svg+xml;base64,<paste kpi-down base64 here>"
VAR _flat   = "data:image/svg+xml;base64,<paste kpi-neutral base64 here>"
RETURN
    SWITCH(TRUE(),
        _change >  0.005, _up,
        _change < -0.005, _down,
        _flat)
```

> In a table or matrix visual: set the field's format to **Image URL** in the Format pane. Set the column width to 28–36px.

---

#### Domain-Specific Icon Generation — Mandatory for Real Reports

When building a report (not reviewing this template), **always generate actual domain-specific SVG code** by following these steps:

1. **Read Phase 1** subject, page themes, and the business questions for every page.
2. **Pick the visual metaphor** for the domain from the derivation table above (or infer one if the domain is not listed).
3. **Generate the cover hero SVG** — an 1200×400 abstract illustration evoking the domain. Left 45% reserved for text. Use geometric shapes, arcs, and lines — not clip art or literal photographs.
4. **Generate one page icon per page** identified in Phase 1.3. Each icon's shape must directly represent the analytical concept of that page. Name each file `icon-<page-slug>.svg`.
5. **Write all files** to `assets/icons/` in the dashboard project folder (alongside the existing theme JSON file).
6. **Output a summary table** listing every generated file, its SVG content description, and its intended placement in the report.

**Example summary table for a telecom churn report:**

| File | SVG content | Placement in Power BI |
|---|---|---|
| `assets/icons/hero-cover.svg` | Signal tower with radiating concentric arcs + scattered network-node dots | Cover page — full-width Image visual |
| `assets/icons/icon-overview.svg` | 4-bar signal-strength meter (tallest bar on right) | Nav button → Overview page |
| `assets/icons/icon-churn-analysis.svg` | Door silhouette with an exit arrow pointing right | Nav button → Churn Analysis page |
| `assets/icons/icon-revenue.svg` | Coin stack with a small upward tick mark above it | Nav button → Revenue Analysis page |
| `assets/icons/icon-geography.svg` | Map pin with a small signal-arc radius circle | Nav button → Geographic Distribution page |
| `assets/icons/icon-customer-profile.svg` | Person silhouette with 3 signal bars to the right | Nav button → Customer Profile page |
| `assets/icons/icon-scenario.svg` | Three horizontal lines with moveable circle handles | Nav button → What-If Scenario page |
| `assets/icons/icon-quality.svg` | Shield outline with a checkmark path inside | Nav button → Data Quality page |
| `assets/icons/kpi-up.svg` | Green upward-pointing triangle | DAX `KPI Direction Icon` measure |
| `assets/icons/kpi-down.svg` | Red downward-pointing triangle | DAX `KPI Direction Icon` measure |
| `assets/icons/kpi-neutral.svg` | Gray horizontal rectangle (dash) | DAX `KPI Direction Icon` measure |

> When this is a real report build: generate the complete SVG code for every row in this table, write each file to `assets/icons/`, and confirm the file list before proceeding to Phase 9.1.

---

### 9.1 Standard Page Layout Template

```
┌─────────────────────────────────────────────────────────────────┐
│  [Logo / Title]                             [Date Range Label]  │  Header 40px
├─────────────┬───────────────────────────────────────────────────┤
│             │  KPI Card 1  │ KPI Card 2  │ KPI Card 3  │ Card 4 │  KPI row 120px
│  Slicer     ├───────────────────────────────────────────────────┤
│  Panel      │                                                   │
│             │           Primary Visual (largest area)           │
│  Slicers    │                                                   │
│  driven by  ├──────────────────────┬────────────────────────────┤
│  Phase 1    │  Secondary Visual    │   Secondary Visual         │
│  questions  │                      │                            │
└─────────────┴──────────────────────┴────────────────────────────┘
```

### 9.2 Page Design Principles

> Pages are determined by the question themes from Phase 1.3. Below are generic page types — include only the pages that answer real questions from your dataset.

**Executive Summary** — KPI cards for the 4–6 most important metrics, trended over time, sliced by the highest-level dimensions. Designed for stakeholders who need 30-second answers.

**Trend / Time Analysis** — Time series visuals using the Time Intelligence Calculation Group. Show Actual, Prior Year, and YoY % in the same view. Add the Time Period slicer.

**Dimensional Breakdown** — Bar/column charts using Field Parameter Dimension Selector and Measure Selector. Enables self-service "slice by anything" analysis.

**Detail / Ranking** — Top N ranked table or bar chart using the Top N parameter and ranking measures. Scatter plots to find outliers.

**What-If Scenario** — All What-If parameter sliders on one page. Visuals show actual vs. adjusted metric side by side. Include explanatory text for each slider.

**Data Quality & Glossary** — `[Last Refresh]` card, row count per source table, data dictionary table, data owner and refresh schedule text box.

### 9.3 Slicers

- Every page has a **Date Range slicer** (range slider or between dates)
- Dimension Selector and Measure Selector slicers on every page with a field parameter visual
- What-If sliders only on the Scenario page
- Slicer headers are always visible and clearly labelled

### 9.4 Visual Formatting Standards

**Color palette:**
- Primary metric: `#1F4E79` (deep blue)
- Positive variance: `#107C10` (green)
- Negative variance: `#A4262C` (red)
- Neutral / secondary: `#605E5C` (gray)
- Background: `#FFFFFF` or `#F5F5F5`

**KPI Cards — always use the new Card (v2) visual (Insert → Card (new)):**
- Primary value: 20pt+ bold (the Callout value)
- Use `BLANK()` not 0 for no-data states — prevents misleading zero display
- **Reference Labels** — always evaluate whether one or more Reference Labels add useful context to each card. Reference Labels are secondary values displayed alongside the main KPI and are one of the most valuable features of the Card v2 visual.

  | Reference Label use-case | Example measure to attach | When to use |
  |---|---|---|
  | **Prior period comparison** | `[Total Sales PY]` or any time intelligence measure via Calc Group | Any card where YoY or MoM change is meaningful |
  | **Target / goal** | A budget or target measure | When a target exists in the model |
  | **Benchmark / average** | `[Avg Points]` on a max or min card | To show context for an extreme value |
  | **Variance (difference or %)** | `[YoY Variance]` or `[YoY %]` | When the absolute change is more useful than the raw prior value |

  **How to add a Reference Label:**
  1. Select the card visual → Format pane → **Reference labels** section
  2. Click **+ Add label** → choose the measure or field to display
  3. Set **Label**: a short display name (e.g. "vs Prior Year", "Target", "Avg")
  4. Set **Value**: the measure name from the model
  5. Optionally enable **Trend indicator** (arrow up/down) and set positive-direction to match business meaning (e.g., revenue up = good, defect rate up = bad)
  6. Apply conditional colour to the reference label value: green for favourable, red for unfavourable, using the same thresholds as the report's conditional formatting palette

  > **Rule**: A card showing a KPI without any period comparison is almost always a missed opportunity. If the model has a Calculation Group with `Prior Year` or `MoM` items, attach a reference label using that Calc Group measure on every summary KPI card unless the metric is genuinely point-in-time with no comparison meaning (e.g. a count of active records).

**Conditional Formatting:**
- Variance columns → background color scale from red (worst) through white (neutral) to green (best)
- Margin / ratio columns → font color rules at defined thresholds
- Rank columns → data bars (reversed so rank 1 = widest)

**Accessibility:**
- Minimum 11pt body text, 14pt KPI values, 18pt page titles
- Every visual: set Alt Text in Format pane > General > Alt Text
- Never rely on red/green alone — add icons or labels for colorblind users

---

### 9.5 Generate Report Build Guide — `docs/report-build.md` ⚠️ MANDATORY FINAL STEP

> ⚠️ **Generate this document BEFORE the junior developer begins any canvas work.** This is the single most important deliverable of Phase 9 — without it, the manual build steps are undocumented and error-prone. Do not skip or abbreviate it.

Create a file at `dashboards/<project-folder>/docs/report-build.md`. This document is the complete instructions a junior Power BI developer needs to build every page of the report from scratch inside Power BI Desktop.

**The guide must be comprehensive. Every section listed below is required:**

#### Required Sections

**1. Prerequisites**
- How to open the PBIP file and verify it loads cleanly
- How to apply the custom JSON theme file (path relative to the dashboard folder)
- How to trigger a data refresh and confirm row counts per table
- Canvas size setting (View → Page view → Actual size; set to 1280×720 unless otherwise specified)
- Note any model warnings expected on first load and how to dismiss them

**2. Theme Colour Reference**
- A named table of every hex colour used in the report — primary background, secondary background, accent, positive, negative, neutral, text light, text dark
- Which colour maps to which Power BI formatting property (e.g. "use `#ff9900` for data bars, data point highlights, and KPI titles")

**3. Data Model Quick Reference**
- A brief table of every table in the model: table name, grain (one row = one what?), row count (approximate), key column
- List of all relationships in the format: `Dim_X[Key] → Fact_Y[Key] (1:*)` — one per line

**4. Full Measure Cheat-Sheet**
- Every measure in `_Measures`, one row per measure: Measure Name | Display Folder | Format String | Plain-English Description
- Include calculation group items (Time Intelligence, Format Switcher) in a separate sub-table
- Include all Field Parameter tables and What-If parameter tables with their SELECTEDVALUE column names

**5. Navigation Bar Setup (step-by-step)**
- How to add a blank button, load a custom SVG icon, set tooltip text, set the Page Navigation action
- Exact icon filename to use for each page (match the `assets/icons/icon-<page-slug>.svg` files)
- Recommended button size, spacing, and alignment
- How to group the nav bar and paste it onto every page

**6. Standard Page Layout Instructions**
- ASCII art template of the standard grid (header strip, slicer panel left, KPI row, primary visual, secondary visuals)
- Exact pixel dimensions for each zone at 1280×720 canvas
- Margin and padding conventions (e.g., 12px outer margin, 8px gap between visuals)
- How to align multiple visuals using Format → Align

**7. Page-by-Page Build Specs**

For EVERY page identified in Phase 1.3, include a dedicated sub-section containing ALL of the following:

- **Page name and purpose** (1–2 sentences)
- **Slicers**: field, slicer type (dropdown / list / slider / date range), multi-select on/off, default value
- **KPI Cards**: one row per card — title, Value field (measure name), Reference Label measure (if used — see Phase 9 Reference Labels rule), Reference Label display label text, trend indicator direction (positive = good / positive = bad), trend axis (if used)
- **Every other visual on the page**: visual type, exact fields on each well (Axis / Legend / Values / Tooltips / Details), sort order, whether cross-filter is On or Off, any drill-through configured
- **Conditional formatting**: which column/measure gets it, what type (background colour / font colour / data bars / icons), rules (scale bounds or specific thresholds), colours used
- **Calculation group slicers**: which calculation group slicer to place on this page, which field it targets
- **Field Parameter slicers** (if applicable): which parameter, slicer type, default selection
- **What-If parameter slicers** (if applicable): parameter name, min/max display, default value label
- **Page-level filters**: any filters set in the Filters pane for this specific page
- **Any special notes**: e.g., "set Cross-filter interaction from Slicer A to Visual B to None", "disable visual header for all slicers on this page"

> Do not merge pages. Each page gets its own `###` sub-section. The junior developer reads one section then builds that page before moving on.

**8. Conditional Formatting Recipes**
- One sub-section per conditional formatting rule used anywhere in the report
- Include: which visual and column it applies to, step-by-step instructions to open the dialog, exact settings to enter (scale min/max colours, threshold values, colour hex codes)

**9. Accessibility Checklist**
- Alt text to add for each major visual (suggested text written out — not just "add alt text")
- Tab order guidance per page
- Confirmation that no information is conveyed by colour alone

**10. Common Troubleshooting**
- A table of 6–10 common issues a junior developer might hit, with cause and fix:
  - Visuals showing blank / "no data"
  - Slicers not cross-filtering correctly
  - Calculation group not changing the displayed measure
  - Field Parameter slicer showing wrong fields
  - What-If slider not affecting the KPI
  - Theme not applied correctly
  - Navigation button not navigating to the correct page

**11. Final Handoff Checklist**
- Every page has a navigation bar
- All slicer selections reset to defaults (clear all slicers before saving)
- Report saved as both `.pbip` (source) and exported as `.pbix` (if required)
- `docs/report-build.md` committed to the repository
- Performance Analyzer run — all visuals under 2 seconds
- File tested with no MCP connection (standalone Desktop)

#### Tone and Level of Detail

Write for a **junior Power BI developer** who:
- Knows how to open Power BI Desktop and create basic visuals
- Does not know the specific fields, measures, or design decisions for THIS report
- Will follow the guide step by step without asking questions

Every instruction must be explicit. Never write "add a bar chart" — write "Insert → Bar chart → set Axis = `Dim_Variety[Variety]`, Values = `[Avg Points]`, sort descending by `[Avg Points]`". Never say "apply the theme colours" — name the exact hex code and the exact Format pane property.

#### Length Expectation

A complete `report-build.md` for a 6–8 page report is typically **700–1 200 lines** of Markdown. If the guide is shorter than ~600 lines it is almost certainly missing page-by-page visual specs. Do not truncate — generate the full document in one pass.

---

## Phase 10 — Quality Audit

### 10.1 MCP Model Audit

```
Prompt to MCP:
"Run a full model quality review and report on each item:
1. Any bidirectional relationships (flag as warning — should be unidirectional unless a bridge table requires it)
2. Any measures using / instead of DIVIDE()
3. Any calculated columns in Fact tables (should not exist — compute in Power Query)
4. Any visible columns ending in 'Key' in Report view (should all be hidden)
5. Any measures missing a Description property
6. Any tables with no relationship to any other table (orphaned tables)
7. Whether Dim_Date is marked as the Date Table
8. Whether Auto Date/Time is disabled
9. No standalone time intelligence measures in _Measures (YoY, PY, YTD etc. should be in Calculation Group)
10. Precedence on Calculation Groups — Time Intelligence = 10, Format Switcher = 5
11. Every measure in _Measures has a non-blank displayFolder assigned
12. Query Groups are assigned in Power Query — Raw_* in '📥 Staging', Dim_* in '📅 Dim Tables', Fact_* in '📊 Fact Tables', fn_* in '🔧 Functions'
```

```
Prompt to MCP:
"Execute validation DAX queries for all measures in _Measures.
Confirm each returns a non-null, non-error value for a known date range in the loaded data.
Report any that return errors or always return BLANK."
```

### 10.2 Checklists

**Data Model:**
- [ ] All relationships are active, single-direction, one-to-many from dimension to fact
- [ ] `Dim_Date` is marked as Date Table
- [ ] Auto Date/Time is disabled
- [ ] All `*Key` and `DateKey` columns hidden from Report view
- [ ] No many-to-many relationships without a bridge table
- [ ] RLS roles defined and tested if multi-user access is required

**Power Query:**
- [ ] Every query has explicit data type transformations on all columns
- [ ] No source file is loaded more than once — all downstream queries use references
- [ ] All `Fact_*` tables contain only keys and numeric measures — no descriptive text columns
- [ ] All `Dim_*` tables have a surrogate key as the first column
- [ ] `fn_GenerateSurrogateKey` has Enable Load = off
- [ ] All `Raw_*` staging queries have Enable Load = off

**DAX:**
- [ ] All measures in `_Measures` table
- [ ] All measures have Description property (verified by MCP audit)
- [ ] All measures have a non-blank displayFolder (verified by MCP audit)
- [ ] displayFolder taxonomy follows the standard grouping: 📊 Core Metrics, 💰 Revenue, 📈 Rates & Ratios, 🔢 Ranking, ⚙️ What-If, 🔧 Utility (adapt to dataset domain)
- [ ] `DIVIDE()` used everywhere — no bare `/` (verified by MCP audit)
- [ ] No calculated columns in fact tables (verified by MCP audit)
- [ ] No standalone time intelligence measures — all in Time Intelligence Calculation Group

**Report:**
- [ ] SVG assets exist in `assets/icons/` — hero cover + one icon per page + kpi-up/down/neutral
- [ ] All page icons are domain-matched (not generic chart shapes) and named `icon-<page-slug>.svg`
- [ ] Hero cover SVG is inserted as a full-width Image visual on the Cover page
- [ ] Every report page has a navigation button bar using page-specific SVG icons
- [ ] KPI Direction Icon DAX measure uses base64 data URLs; format set to Image URL in table/matrix visuals
- [ ] Every page has a clear title
- [ ] Executive Summary loads in under 3 seconds (Performance Analyzer)
- [ ] Every slicer has a visible header label
- [ ] Field Parameter slicers on every page that uses them
- [ ] What-If page has explanatory text for every slider
- [ ] Cross-filter interactions intentionally configured (Edit Interactions)

**Performance:**
- [ ] No calculated columns in fact tables
- [ ] No bidirectional relationships unless bridge table requires it
- [ ] Surrogate key types consistent: `type text` (Base64) for hashed keys, `Int64.Type` for `DateKey`
- [ ] Performance Analyzer run — no visual exceeds 2 seconds

---

## Appendix A — MCP Prompt Library

Quick-copy prompts for recurring tasks:

```
// Add a single new measure
"Add a measure named [Name] to _Measures with this DAX:
<DAX>
Description: <description>"

// Bulk rename measures
"Rename all measures in _Measures that start with '<OldPrefix>' to start with '<NewPrefix>' instead"

// Generate model documentation
"Generate a complete markdown document for this semantic model including:
- Mermaid ERD of all tables and relationships
- All measures with DAX and business descriptions
- All Power Query data sources and row counts
- Row-level security roles and filter logic if any"

// DAX performance audit
"Analyse the performance of this DAX measure and suggest optimisations:
<paste measure DAX here>"

// Validate all relationships
"List all relationships: from table, from column, to table, to column,
cardinality, filter direction, and active/inactive status"

// Check for unused objects
"List all tables, columns, and measures that have no references anywhere
in the model — potential candidates for removal"

// Apply best practices
"Evaluate this model against Microsoft Power BI best practices
and provide a prioritised remediation list with specific object names and fixes"

// Add translations
"Add <language> translations for all measure display names and column display names in this model"
```

---

## Appendix B — Output Deliverable Format

When producing a full build response, always structure output in this order:

1. **MCP mode declaration** — "MCP connected: executing directly" or "MCP not connected: providing manual instructions"
2. **Phase 1 summary** — profiling findings, data quality issues, grain of each table, list of business questions
3. **Phase 2 schema design** — ERD (text or Mermaid), all tables, all columns, all relationships
4. **Phase 3 Power Query M code** — all queries, copy-paste ready
5. **Phase 4 MCP prompts** — scaffold tables, relationships, mark date table, hide keys
6. **Phase 5 MCP prompts** — sort-by, data categories, hierarchies, best practice check, RLS
7. **Phase 6 MCP prompts** — all DAX measures with descriptions, validation queries
8. **Phase 7 MCP prompts** — Calculation Groups creation and verification
9. **Phase 8** — Field Parameter definitions, What-If parameter specs, MCP validation
10. **Phase 9** — page-by-page visual specifications, slicer list, formatting instructions
11. **Phase 10 MCP audit prompts** — full quality checklist execution

### Complete Build Sequence

| Step | Tool |
|---|---|
| Profile source files and read data dictionary | Analyst |
| Document business questions and group into page themes | Analyst |
| Design star schema ERD on paper | Analyst |
| Open Power BI Desktop — create blank .pbix | Desktop UI |
| Define `fn_GenerateSurrogateKey` function query | Power Query Editor or **MCP** |
| Build `Raw_*` staging queries (Enable Load = off) | Power Query Editor or **MCP** |
| Build all `Dim_*` queries via references | Power Query Editor or **MCP** |
| Build all `Fact_*` queries via references | Power Query Editor or **MCP** |
| Assign all queries to Query Groups (📥 Staging, 📅 Dim Tables, 📊 Fact Tables, etc.) | Power Query Editor |
| Close and Apply — load all queries | Desktop UI |
| Connect MCP to the open .pbix | MCP |
| Disable Auto Date/Time | **MCP** |
| Scaffold empty tables (if not already created by Power Query) | **MCP** |
| Create all relationships | **MCP** |
| Mark Dim_Date as Date Table | **MCP** |
| Hide all key columns | **MCP** |
| Set sort-by columns | **MCP** |
| Set data categories on geography columns | **MCP** |
| Create hierarchies | **MCP** |
| Run Best Practice Analyzer | **MCP** |
| Define RLS roles (if required) | **MCP** |
| Create `_Measures` table with measures grouped by displayFolder | **MCP** |
| Validate all measures with test queries | **MCP** |
| Create Time Intelligence Calculation Group | **MCP** |
| Create Format Switcher Calculation Group | **MCP** |
| Verify Calculation Groups | **MCP** |
| Create Field Parameters (DAX calculated tables + PBI_ResultType annotation) | **MCP** |
| Create What-If Numeric Parameters (DAX calculated tables + PBI_NavigationStepName annotation) | **MCP** |
| Hide [Order] columns, verify field parameter measures | **MCP** |
| Create What-If SELECTEDVALUE measures and scenario measures | **MCP** |
| Generate SVG hero image and all page navigation icons to `assets/icons/` | AI — generate SVG code |
| Generate KPI status icons (`kpi-up`, `kpi-down`, `kpi-neutral`) to `assets/icons/` | AI — generate SVG code |
| Convert KPI status SVGs to base64 data URLs; create `KPI Direction Icon` DAX measure | PowerShell + **MCP** |
| Build report pages — visuals, slicers, layout | Desktop UI — Report canvas ⚠️ ONLY MANUAL PHASE |
| Apply theme, conditional formatting, alt text | Desktop UI — Format pane |
| Configure cross-filter interactions | Desktop UI — Edit Interactions |
| Run MCP quality audit | **MCP** |
| Run Performance Analyzer | Desktop UI |
| Save and publish to Fabric / Power BI Service | Desktop UI |
