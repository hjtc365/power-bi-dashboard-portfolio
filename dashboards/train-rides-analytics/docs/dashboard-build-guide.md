# UK National Rail Dashboard — Complete Build Guide

> **Who is this for?** Junior Power BI developers building their first professional multi-page report.
> **What you'll build:** A 7-page interactive dashboard analysing National Rail UK performance
> (Jan–Apr 2024) — routes, peak travel, revenue, on-time performance — styled with the
> **Office Slipstream** colour palette.

---

## Table of Contents

1. [Prerequisites & Setup](#1-prerequisites--setup)
2. [Apply the Custom Theme](#2-apply-the-custom-theme)
3. [Page Layout System](#3-page-layout-system)
4. [Page 0 — Cover / Landing Page](#4-page-0--cover--landing-page)
5. [Page 1 — Executive Overview](#5-page-1--executive-overview)
6. [Page 2 — Passenger Usage & Peak Travel](#6-page-2--passenger-usage--peak-travel)
7. [Page 3 — Sales & Revenue](#7-page-3--sales--revenue)
8. [Page 4 — Railway Performance](#8-page-4--railway-performance)
9. [Page 5 — Route Analysis](#9-page-5--route-analysis)
10. [Page 6 — Scenario Simulator](#10-page-6--scenario-simulator)
11. [Navigation Buttons](#11-navigation-buttons)
12. [Sync Slicers Across Pages](#12-sync-slicers-across-pages)
13. [Conditional Formatting on KPI Cards](#13-conditional-formatting-on-kpi-cards)
14. [Tooltip Pages](#14-tooltip-pages)
15. [Final Polish Checklist](#15-final-polish-checklist)
16. [Field Parameters & Dynamic Metric Switching](#16-field-parameters--dynamic-metric-switching)
17. [Button Slicers Setup](#17-button-slicers-setup)
18. [What-If Parameters (Numeric Sliders)](#18-what-if-parameters-numeric-sliders)

---

## Colour Reference (Slipstream Palette)

| Role | Hex | Use |
|---|---|---|
| **Dark Navy** | `#1F3864` | Header bars, nav sidebar, column headers |
| **Medium Blue** | `#4059AC` | Secondary headings, category labels |
| **Bright Cyan** | `#00B0F0` | Primary data colour, highlight bars |
| **Light Blue** | `#9DC3E6` | Card borders, slicer backgrounds |
| **Very Light Blue** | `#EBF3FB` | Card & slicer fill, alternate row colour |
| **Page Background** | `#F0F5FB` | Canvas background |
| **Lime Green** | `#92D050` | "Good" / on-time indicators |
| **Teal** | `#29AB87` | Secondary positive metric |
| **Orange** | `#F4A442` | "Neutral" / warning indicators |
| **Red-Orange** | `#E74C3C` | "Bad" / cancelled / delay indicators |
| **Text Dark** | `#2C3E50` | Body text, axis labels |

---

## 1. Prerequisites & Setup

### 1.1 Open the PBIP project

1. In **File Explorer**, navigate to:
   ```
   dashboards/train-rides-analytics/pbip/
   ```
2. Double-click `uk-train-rides-analysis.pbip` to open it in Power BI Desktop.
3. When prompted about privacy levels, select **Ignore Privacy Levels** and click **Save**.
4. Click **Refresh** in the **Home** ribbon to load all data.
5. Verify all tables loaded: in the **Data** pane you should see:
   - `Dim_Date`, `Dim_Journey_Status`, `Dim_Payment`, `Dim_Station`, `Dim_Ticket`
   - `Fact_Transactions`, `_Measures`

### 1.2 Set canvas size (16:9 widescreen)

1. Click on an empty area of the canvas.
2. In the **Format** pane → **Page information** → set **Name** = `Overview`.
3. Go to **View** ribbon → **Page view** → select **Fit to page**.
4. Go to **View** → **Canvas settings**:
   - Type: **Custom**
   - Width: **1280**
   - Height: **720**
5. Repeat for every new page you create.

> **Tip:** 1280 × 720 is standard HD (16:9). It looks great on both laptops and large monitors.

---

## 2. Apply the Custom Theme

1. In the **View** ribbon, click **Themes** → **Browse for themes**.
2. Navigate to:
   ```
   dashboards/train-rides-analytics/assets/Slipstream-Theme.json
   ```
3. Click **Open**. Power BI will apply the Slipstream theme immediately.
4. Click **Apply anyway** if prompted about existing formatting.

The theme sets:
- Page background to soft blue-grey `#F0F5FB`
- All visual titles to dark navy `#1F3864` Segoe UI Bold
- Card backgrounds to light blue `#EBF3FB`
- Table/matrix headers to dark navy with white text
- Data colour cycle: Cyan → Orange → Lime Green → Teal → Blue → Red

---

## 3. Page Layout System

Every page uses the **same three-zone layout**. Set it up once and duplicate it.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  HEADER BANNER  (1280 × 60 px, Dark Navy #1F3864)                       │
│  Page Icon  │  PAGE TITLE (white, Segoe UI 18pt Bold)     │ Date Slicer  │
├──────┬──────────────────────────────────────────────────────────────────┤
│ NAV  │                                                                   │
│ BAR  │                  MAIN CONTENT AREA                               │
│ 160  │                  (1120 × 620 px)                                 │
│  px  │                                                                   │
│      │                                                                   │
│(Dark │                                                                   │
│Navy) │                                                                   │
└──────┴──────────────────────────────────────────────────────────────────┘
```

### 3.1 Create the Header Banner

1. Insert **Rectangle** shape: `Insert` → `Shapes` → `Rectangle`
2. **Position:** X = 0, Y = 0 · **Size:** W = 1280, H = 60
3. **Format Shape:**
   - Fill: `#1F3864` (Dark Navy), 100% opacity
   - Border: off

#### Step 2 — Add the page icon

Each analysis page has a matching icon in `assets/`. The icon sits in the left portion of the header bar, vertically centred.

| Page | Icon file |
|---|---|
| Page 1 — Executive Overview | `icon-overview.svg` |
| Page 2 — Passenger Usage | `icon-passengers.svg` |
| Page 3 — Sales & Revenue | `icon-revenue.svg` |
| Page 4 — Railway Performance | `icon-performance.svg` |
| Page 5 — Route Analysis | `icon-route.svg` |
| Page 6 — Scenario Simulator | `icon-simulator.svg` |

1. **Insert** → **Image** → navigate to `dashboards/train-rides-analytics/assets/` → select the matching icon file for this page.
2. In **Format visual** → **Image** → Image fit: **Fit** (preserves the circular shape).
3. Set exact size and position:
   - **W = 44, H = 44**
   - **X = 8, Y = 8** (8px from the left edge, 8px from the top — perfectly centres the 44px icon in the 60px banner height)
4. Right-click the icon image → **Send to back** (then bring it forward one step above the banner rectangle, so it sits on top of the rectangle but below any text).

> **Why these coordinates?** The banner is 60px tall. A 44px icon centred vertically: (60 − 44) ÷ 2 = 8px top offset. The larger 44px size makes the icon clearly visible without overflowing the banner.

#### Step 3 — Add the page title text box

4. Add a **Text Box** on top of the banner:
   - Text: your page title in ALL CAPS, e.g. **"EXECUTIVE OVERVIEW"**
   - Font: Segoe UI, 18pt, **Bold**, White (`#FFFFFF`)
   - **Position:** X = 60, Y = 18 (immediately right of the icon, vertically centred)
   - **Size:** W = 700, H = 28

> **X = 60** = icon X (8) + icon width (44) + 8px gap. **Y = 18** centres the 24px-tall 18pt text in the 60px banner: (60 − 24) ÷ 2 = 18.

### 3.2 Create the Navigation Sidebar

1. Insert **Rectangle** shape:
   - **Position:** X = 0, Y = 60 · **Size:** W = 160, H = 660
   - Fill: `#1F3864`, no border
2. Add **Text Box** for the report branding:
   - Text: `🚆 UK Rail`
   - Font: Segoe UI, 13pt, Bold, White
   - Position: X = 5, Y = 65
3. Leave space for navigation buttons (added in [Section 10](#10-navigation-buttons)).

### 3.3 Save as a template page

After creating the layout on Page 1:
1. Right-click the page tab → **Duplicate Page**.
2. Use this duplicate as the starting point for every new page.

---

## 4. Page 0 — Cover / Landing Page

> **Purpose:** A professional title screen that makes a great first impression and provides one-click navigation to each analysis page.

```
Canvas: 1280 × 720  ·  no sidebar on the Cover page
┌──────────────────────────────────────────────────────────────────────────┐ ← Y = 0
│                  COVER BANNER SVG  (1280 × 400 px)                       │
│        [title, subtitle, date pill, tagline — all embedded in SVG]       │
│                   [🚆 icon straddles seam at Y = 368]                    │
├──────────────────────────────────────────────────────────────────────────┤ ← Y = 400
│                        dark zone  (#0D1E38)                               │
│                                                                           │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────┐ │ ← Y = 504
│  │ [icon]  │ │ [icon]  │ │ [icon]  │ │ [icon]  │ │ [icon]  │ │[icon] │ │
│  │Overview │ │Passenger│ │Revenue  │ │Perf.    │ │Routes   │ │Simultr│ │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └───────┘ │ ← Y = 604
│                  205 × 100 px each  ·  6 tiles total                     │
├───────────────── footer strip (13 px, #00B0F0) ──────────────────────────┤ ← Y = 707
└──────────────────────────────────────────────────────────────────────────┘ ← Y = 720
```

### 4.1 Page Setup

1. **Rename** the first page tab: right-click → **Rename** → `Cover`.
2. Set canvas to 1280 × 720 (see Section 3 if not already done).
3. Format pane → **Canvas background** → Colour: `#0D1E38` (deep navy — fills the visible gap below the banner).

### 4.2 Insert the Cover Banner

The file `assets/cover-banner.svg` is a **1280 × 400 px** pre-built hero image. It contains the train silhouette, perspective track, typography hierarchy, and Slipstream colour scheme — you do **not** need to add any text on top of it.

> **Why 400 px and not 720?** This is intentional. The banner fills the top 55% of the canvas (0–400px), leaving the bottom 320px as a clean dark zone for your navigation tiles. This is a standard app-cover pattern: hero image on top, action grid below.

1. **Insert** → **Image** → navigate to `dashboards/train-rides-analytics/assets/cover-banner.svg`.
2. In **Format visual** → **Image** → Image fit: **Fill**.
3. Set exact size and position:
   - **X = 0, Y = 0, W = 1280, H = 400**
4. Right-click the image → **Send to back**.

> **Do not add any text boxes over the banner.** All title text ("UK NATIONAL RAIL", "Performance Dashboard", the date pill, and the tagline) is already embedded in the SVG.

### 4.3 Train Icon Accent (optional)

For a subtle branding flourish at the join between banner and nav zone:

1. **Insert** → **Image** → select `assets/icon-train.svg`.
2. Size: **W = 64, H = 64**. Position: **X = 86, Y = 368** (overlapping the bottom edge of the banner, half in / half out).
3. This creates the visual effect of the train "entering" the page.

### 4.4 Navigation Tile Grid

The bottom zone (Y = 415 to Y = 690) holds 6 launch tiles for the analysis pages. Each tile contains three layers: a background rectangle, an SVG icon, and a text label.

#### Tile icon reference

Each tile has a dedicated SVG icon in the `assets/` folder:

| Tile | Icon File | Icon colour theme |
|---|---|---|
| Executive Overview | `icon-overview.svg` | Dark Navy + Cyan bars + Green trend |
| Passenger Usage | `icon-passengers.svg` | Medium Blue + white people |
| Sales & Revenue | `icon-revenue.svg` | Dark Navy + Cyan £ symbol |
| Railway Performance | `icon-performance.svg` | Cyan gauge with needle |
| Route Analysis | `icon-route.svg` | Teal + departure/arrival pins |
| Scenario Simulator | `icon-simulator.svg` | Teal + two sliders + delta arrow |

#### Step 1 — Build the background rectangle (tile template)

1. **Insert** → **Shapes** → **Rectangle**.
2. Size: **W = 205, H = 100**. Rounded corners: 10px.
3. Fill: `#1F3864` at 55% opacity. Border: `#00B0F0`, 2px.
   > **Why 55% opacity?** The cover canvas is deep navy `#0D1E38`. At 8% white the tile was invisible — white text had no anchor. At 55% navy the tile reads as a clear frosted-glass card while the border and icon still pop.
4. Select the rectangle → **Format visual** → **Action** → toggle **ON**:
   - Type: **Page navigation** · Destination: *(set per tile — see table below)*

#### Step 2 — Add the icon image to each tile

Each icon sits in the **left third** of the tile, vertically centred.

1. With the rectangle placed, go to **Insert** → **Image** → select the matching icon file.
2. Set Image fit: **Fit** (preserves the circle shape).
3. Set size: **W = 44, H = 44**.
4. Position the icon at: `tile_X + 10`, `tile_Y + 28` (10px from left edge, 28px from top — centres the 44px icon inside the 100px tile height: (100 − 44) ÷ 2 = 28).

#### Step 3 — Add the text label

1. **Insert** → **Text box**.
2. Font: Segoe UI **11pt** Bold, colour `#EBF3FB` (very light blue — sharper against the navy tile than pure white).
3. Size: **W = 125, H = 44**.
4. Position: `tile_X + 62`, `tile_Y + 28` (immediately right of the icon with 8px gap, vertically aligned with the icon).
5. Type the label text (see table below).

#### Tile positions, icons, and labels

| Tile | Rect X | Rect Y | Icon file | Label text | Target page |
|---|---|---|---|---|---|
| 1 | 1 | 504 | `icon-overview.svg` | Executive Overview | Page 1 |
| 2 | 214 | 504 | `icon-passengers.svg` | Passenger Usage | Page 2 |
| 3 | 427 | 504 | `icon-revenue.svg` | Sales & Revenue | Page 3 |
| 4 | 640 | 504 | `icon-performance.svg` | Railway Performance | Page 4 |
| 5 | 853 | 504 | `icon-route.svg` | Route Analysis | Page 5 |
| 6 | 1066 | 504 | `icon-simulator.svg` | Scenario Simulator | Page 6 |

> **Efficient workflow:** Build Tile 1 completely (rectangle + icon + text box). Select all three → right-click → **Group**. Then Ctrl+D five times to duplicate. On each copy, update: (a) the rectangle's page navigation destination, (b) the icon image file, (c) the text label. Move each copy to its correct X position — Y stays at 504 for all six.

#### Step 4 — Group and lock each tile

1. Select the rectangle, icon image, and text box for a tile.
2. Right-click → **Group** (or Format → Group).
3. Right-click the group → **Lock** so you don't accidentally move it while building page content.

#### Step 5 — Hover style

1. Select the rectangle within a group → Format visual → **On hover** state.
2. Fill: `#00B0F0` at 25% opacity. This gives a subtle cyan glow on mouse-over.
3. The Scenario Simulator tile (Tile 6) gets a different hover: `#29AB87` (teal) at 30% — a visual signal that it's a different type of page.

### 4.5 Footer

1. Insert a **Rectangle**: X = 0, Y = 707, W = 1280, H = 13.
2. Fill: left third `#29AB87` (teal), middle `#4059AC` (blue), right `#00B0F0` (cyan).
   - Simplest approach: three separate thin rectangles at W = 427 each, side by side, all at Y = 707.
3. This mirrors the three-colour accent strip in the bottom of the banner SVG, creating a visual echo.

---

## 5. Page 1 — Executive Overview

> **Purpose:** One-screen summary. A busy exec can see total journeys, revenue, on-time %, and key trends without clicking anything.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] EXECUTIVE OVERVIEW                         [Month ▼]  [Ticket Class ▼]   │ ← Y = 0–60
├────────┬─────────────────────────────────────────────────────────────────────────┤
│        │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │ ← Y = 80
│  NAV   │  │Journeys │ │Revenue  │ │On-Time% │ │Disruptns│ │Refund £ │          │
│  160px │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │ ← Y = 180
│        ├───────────────────────────────────────┬─────────────────────────────────┤
│  UK    │  Monthly Journey Trend                │  Journey Status Split           │ ← Y = 200
│  Rail  │  (line + MoM % secondary axis)        │  (donut)                        │
│        │  X=168, W=540, H=220                  │  X=720, W=250, H=220            │
│        │                                       │                                 │ ← Y = 420
│        ├───────────────────────────────────────┴─────────────────────────────────┤
│        │  Revenue by Month, Ticket Class           Top 5 Routes by Journeys      │ ← Y = 440
│        │  (stacked column)  X=168, W=540, H=230    (h-bar)  X=720, W=540, H=230  │
│        │  ─────────────────────────────────────────────────────────────────────  │
│        │  §5.7: replace bottom row → [Metric Selector tiles]                     │ ← Y = 460
│        │  + Dynamic KPI bar (X=168, W=800, H=190) + KPI Label card (X=980)      │ ← Y = 670
└────────┴─────────────────────────────────────────────────────────────────────────┘
```

### 5.1 KPI Cards Row (top of content area)

Place **5 Card visuals** in a single row at Y = 80, each 196 × 100 px, spaced 8 px apart.
Start at X = 168.

| # | Measure | Card Label | Notes |
|---|---|---|---|
| 1 | `Total Transactions (Journey Date)` | Total Journeys | |
| 2 | `Total Revenue` | Net Revenue | Format: £#,##0 |
| 3 | `On Time Rate %` | On-Time Rate | Format: 0.0% |
| 4 | `Total Disruptions` | Disruptions | Delayed + Cancelled |
| 5 | `Refund Exposure` | Refund Exposure | Format: £#,##0 |

**Card formatting (applies to all 5):**
1. Select a card → **Format Visual** pane:
2. **Callout value** → Font: Segoe UI 26pt Bold, Color: `#1F3864`
3. **Category label** → Font: Segoe UI 11pt, Color: `#4059AC`, show: ON
4. **Visual background** → Color: `#EBF3FB`, Transparency: 0%
5. **Border** → Color: `#9DC3E6`, Rounded corners: 8px, Width: 1px
6. **Shadow** → ON (outer, top-right)

### 5.2 Monthly Journey Trend — Line Chart

**Position:** X = 168, Y = 200 · **Size:** W = 540, H = 220

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` (sorted by `Month Number`) |
| Y-axis | `Total Transactions (Journey Date)` |
| Line | Add second Y line: `Journeys MoM %` (secondary axis) |
| Tooltip | `On Time Rate %`, `Delay Rate %`, `Cancel Rate %` |

**Formatting:**
- Line colour: `#00B0F0` (primary), `#F4A442` (MoM % secondary)
- Stroke width: 3px
- Show markers: ON, filled circles
- Data labels: ON for the primary series
- Title: `Monthly Journey Trend`
- Add a **constant line** at the 4-month average for reference

### 5.3 Journey Status Donut

**Position:** X = 720, Y = 200 · **Size:** W = 250, H = 220

| Field Well | Value |
|---|---|
| Legend | `Dim_Journey_Status[Journey Status]` |
| Values | `Total Transactions (Journey Date)` |

**Formatting:**
- Inner radius: 55% (so it looks like a donut, not a pie)
- Data colours: On Time = `#92D050`, Delayed = `#F4A442`, Cancelled = `#E74C3C`
- Legend: Bottom, 10pt Segoe UI, color `#4D5F7A`
- Detail labels: Category + percent, 10pt
- Title: `Journey Status Split`
- **Detail label:** `On Time Rate %` displayed in the donut hole using a **Card** visual
  (W = 100 × H = 50 placed over the donut centre; label text "On-Time"; no background)

### 5.4 Revenue by Month — Clustered Column Chart

**Position:** X = 168, Y = 440 · **Size:** W = 540, H = 230

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` (sorted by Month Number) |
| Y-axis | `Total Revenue` |
| Legend | `Dim_Ticket[Ticket Class]` |

Use **Stacked Column** sub-type so Standard vs First Class are visible per month.

**Formatting:**
- Standard Class: `#00B0F0`, First Class: `#4059AC`
- Data labels: ON (show total per bar)
- Y-axis format: `£#,##0`
- Title: `Monthly Revenue by Ticket Class`
- Add **constant line** at average monthly revenue

### 5.5 Top 5 Routes — Horizontal Bar Chart

**Position:** X = 720, Y = 440 · **Size:** W = 540, H = 230

| Field Well | Value |
|---|---|
| Y-axis | `Fact_Transactions[Route]` |
| X-axis | `Total Transactions (Journey Date)` |
| Tooltip | `Total Revenue (Journey Date)`, `On Time Rate %` |

**Formatting:**
- Sort: **Descending** by `Total Transactions (Journey Date)`
- Show top 5 only: In **Filters** pane → `Route` filter → **Top N** → Top 5 by `Total Transactions (Journey Date)`
- Bar colour: `#00B0F0`
- Data labels: ON (end of bar)
- Title: `Top 5 Routes by Journeys`
- Gridlines: Vertical off, Horizontal `#E8EEF7`

### 5.6 Slicers

Add **2 slicers** in the header area (Y = 15, right-aligned):

| Slicer | Field | Style | Position |
|---|---|---|---|
| Month | `Dim_Date[Month Name]` | Dropdown | X = 950, Y = 10, W = 160, H = 40 |
| Ticket Class | `Dim_Ticket[Ticket Class]` | Dropdown | X = 1115, Y = 10, W = 155, H = 40 |

**Slicer formatting:**
- Background: `#FFFFFF` · Border: `#9DC3E6` · Radius: 6px
- Header: hidden (use text box above as label)
- Items font: Segoe UI 10pt `#2C3E50`

### 5.7 Dynamic KPI Trend — Metric Selector

> **Design principle:** One chart that becomes six different charts based on a button click. This is the signature interactive element of the Overview page — it replaces six separate cluttered visuals and makes the report feel genuinely alive.

**Components:** 1 button-tile slicer + 1 bar chart + 1 KPI summary card

#### Step 1: Add the Metric Selector button slicer

**Position:** X = 168, Y = 460 · **Size:** W = 800, H = 40

1. **Insert** → **Slicer** · Drag `Metric Selector[Metric Name]` onto the field well.
2. Format → Slicer settings → Options → Style: **Tile**.
3. Turn off the **Slicer header**.
4. Apply Slipstream button styling:

| State | Fill | Font Color |
|---|---|---|
| Default | `#EBF3FB` | `#1F3864` Bold |
| Selected | `#00B0F0` | `#FFFFFF` Bold |
| Hover | `#9DC3E6` | `#1F3864` |

5. Right-click slicer → **Sort by** → `Sort Order` → Ascending (so buttons read: Journeys → Revenue → On-Time → Delay → Cancel → Avg Price).
6. With the slicer selected, go to **Format** → **Edit interactions** → click **None** for all existing KPI cards above (so clicking a metric button doesn't cross-filter the static summary cards).

#### Step 2: Add the Dynamic KPI bar chart

**Position:** X = 168, Y = 510 · **Size:** W = 800, H = 190

1. Insert a **Clustered Bar Chart** (horizontal).
2. Configure fields:

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Date[Month Name]` |
| X-axis | `[Dynamic KPI]` |

3. Bar colour: `#00B0F0` · Data labels: ON · Title: `Monthly KPI Breakdown`
4. Sort: by `Dim_Date[Month Number]` to keep months in chronological order (not alphabetical).

#### Step 3: Add a Dynamic KPI Label summary card

**Position:** X = 980, Y = 460 · **Size:** W = 278, H = 60

1. Insert a **New Card** · Field: `[Dynamic KPI Label]`
2. Category label: `Selected KPI Total`
3. Card fill: `#EBF3FB` · Border: `#9DC3E6` 1px, radius 6px

> **How it works:** Click "Net Revenue" → the bar chart redraws to show revenue by month, the card shows `£741,921`. Click "Delay Rate %" → both update instantly. All DAX-driven — no bookmarks, no page navigation, no duplicated visuals.

---

## 6. Page 2 — Passenger Usage & Peak Travel

> **Purpose:** Show when people travel, what type of ticket they use, and railcard adoption.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] PASSENGER USAGE & PEAK TRAVEL              [Month ▼]  [Ticket Class ▼]   │ ← Y = 0–60
├────────┬─────────────────────────────────────────────────────────────────────────┤
│        │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │ ← Y = 80
│  NAV   │  │Journeys │ │Peak Jrny│ │Peak Rate│ │Railcard │ │Railcd % │          │
│  160px │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │ ← Y = 180
│        ├──────────────────────────────────────────────────┬──────────────────────┤
│  UK    │  Journeys by Hour — "Heartbeat"                  │  Journeys by         │ ← Y = 200
│  Rail  │  (col chart, peak hours highlighted cyan)        │  Day of Week         │
│        │  X=168, W=740, H=250                             │  (col chart)         │
│        │                                                  │  X=920, W=350, H=250 │
│        ├──────────────────────┬───────────────────────────┴──────────────────────┤ ← Y = 450
│        │  Time Band           │  Railcard Split       │  Ticket Type Mix         │ ← Y = 470
│        │  (donut)             │  (donut)              │  (stacked h-bar)         │
│        │  X=168, W=280, H=220 │  X=460, W=280, H=220  │  X=752, W=520, H=220     │
│        │  ── §6.8: replace bottom-left+mid with [Month|Day|Band|Hour] tiles ─── │ ← Y = 690
└────────┴──────────────────────┴───────────────────────────────────────────────────┘
```

### 6.1 KPI Cards Row

Same row layout as Page 1 (5 cards, Y = 80):

| Measure | Label |
|---|---|
| `Total Transactions (Journey Date)` | Total Journeys |
| `Total Peak Journeys` | Peak Journeys |
| `Peak Journey Rate %` | Peak Rate |
| `Railcard Transactions` | Railcard Users |
| `Railcard Usage Rate %` | Railcard Rate |

### 6.2 Journeys by Hour — Column Chart ("Heartbeat Chart")

> This is the signature visual for this page — a dramatic bar chart showing the twin peaks of commuter travel.

**Position:** X = 168, Y = 200 · **Size:** W = 740, H = 250

| Field Well | Value |
|---|---|
| X-axis | `Fact_Transactions[Departure Hour]` |
| Y-axis | `Total Transactions (Journey Date)` |

**Formatting:**
- Default bar colour: `#9DC3E6` (light blue)
- **Conditional formatting on data colours:**
  - Create a measure:
    ```dax
    Hour Color Flag =
    VAR h = SELECTEDVALUE(Fact_Transactions[Departure Hour])
    RETURN IF(
        (h >= 6 && h <= 8) || (h >= 16 && h <= 18),
        "#00B0F0",
        "#9DC3E6"
    )
    ```
  - Apply this measure as the conditional colour on the bars
- This makes the peak hours stand out in bright cyan while off-peak is light blue
- Add **Reference Lines:**
  - Line 1: X = 6, Label = "Morning Peak", Colour `#1F3864`, dashed
  - Line 2: X = 16, Label = "Afternoon Peak", Colour `#1F3864`, dashed
- X-axis: show 0–23 with label every 2 hours
- Data labels: ON (rotate 90° to fit)
- Title: `Journeys by Departure Hour`

### 6.3 Journeys by Day of Week — Column Chart

**Position:** X = 920, Y = 200 · **Size:** W = 350, H = 250

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Day Short]` (sorted by `Day of Week Number`) |
| Y-axis | `Total Transactions (Journey Date)` |

**Formatting:**
- Weekend bars: `#F4A442` (orange), weekday bars: `#00B0F0` (cyan)
  - Use conditional colour measure based on `Dim_Date[Is Weekend]`
- Title: `Journeys by Day of Week`
- Data labels: ON

### 6.4 Journeys by Time Band — Donut Chart

**Position:** X = 168, Y = 470 · **Size:** W = 280, H = 220

| Field Well | Value |
|---|---|
| Legend | `Fact_Transactions[Time Band]` |
| Values | `Total Transactions (Journey Date)` |

**Data colours:**
| Time Band | Colour |
|---|---|
| Morning Peak | `#00B0F0` |
| Afternoon Peak | `#4059AC` |
| Daytime Off-Peak | `#9DC3E6` |
| Evening | `#29AB87` |
| Night / Early Morning | `#EBF3FB` (with dark border) |

Title: `Journeys by Time Band`

### 6.5 Railcard Usage — Donut Chart

**Position:** X = 460, Y = 470 · **Size:** W = 280, H = 220

| Field Well | Value |
|---|---|
| Legend | `Dim_Ticket[Railcard]` |
| Values | `Total Transactions` |

Data colours: None = `#9DC3E6`, Adult = `#00B0F0`, Senior = `#29AB87`, Disabled = `#F4A442`
Title: `Railcard Holder Split`

### 6.6 Ticket Type Mix — Stacked Bar Chart

**Position:** X = 752, Y = 470 · **Size:** W = 520, H = 220

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Date[Month Name]` |
| X-axis | `Total Transactions (Journey Date)` |
| Legend | `Dim_Ticket[Ticket Type]` |

Ticket Type colours: Advance = `#00B0F0`, Off-Peak = `#29AB87`, Anytime = `#F4A442`
Title: `Monthly Ticket Type Mix`
Sort: Month chronological

### 6.7 Slicer — Purchase Type

Add a **Button slicer** for `Dim_Payment[Purchase Type]`:
- Position: X = 168, Y = 10 (header area) · W = 200, H = 38
- Style: Dropdown
- Background: `#FFFFFF`, border `#9DC3E6`

### 6.8 Dynamic Time Breakdown — Time Dimension Selector

> **Design principle:** Replace four separate static charts (Hour / Day of Week / Time Band / Month) with one chart whose X-axis the user can switch via a button. This is the feature that separates a junior layout from a professional interactive report.

#### Option A — Native Field Parameters (recommended)

1. **File** → **Options and settings** → **Options** → **Preview features** → enable **Field parameters** → restart Power BI Desktop.
2. **Modelling** tab → **New parameter** → **Fields**. Name it `Time Axis`.
3. Add these fields and rename each:

| Source Field | Display Name |
|---|---|
| `Dim_Date[Month Name]` | Month |
| `Dim_Date[Day Short]` | Day of Week |
| `Fact_Transactions[Time Band]` | Time Band |
| `Fact_Transactions[Departure Hour]` | Hour |

4. Power BI auto-generates a parameter table and a slicer. Dismiss the auto-placed slicer.
5. Place the slicer at **X = 168, Y = 460, W = 480, H = 40** · Style: **Tile** · Same Slipstream button styling as §5.7 Step 1.
6. Insert a **Column Chart** at **X = 168, Y = 510, W = 480, H = 185**:
   - X-axis: `Time Axis` (the generated parameter field)
   - Y-axis: `Total Transactions (Journey Date)`
   - Bar colour: `#00B0F0` · Title: `Journeys by Time Dimension`

#### Option B — Bookmark approach (no preview feature required)

1. Create 4 **Column Charts** all at the same position (X = 168, Y = 510, W = 480, H = 185). Stack them exactly on top of each other:
   - Chart A · X-axis: `Dim_Date[Month Name]`
   - Chart B · X-axis: `Dim_Date[Day Short]` (sort by `Dim_Date[Day of Week Number]`)
   - Chart C · X-axis: `Fact_Transactions[Time Band]`
   - Chart D · X-axis: `Fact_Transactions[Departure Hour]`
2. All charts: Y-axis = `Total Transactions (Journey Date)` · Bar colour `#00B0F0`
3. Go to **View** → **Bookmarks** → create 4 bookmarks. For each, hide the other 3 charts then capture:
   - `TD_Month` — Chart A only
   - `TD_DayOfWeek` — Chart B only
   - `TD_TimeBand` — Chart C only
   - `TD_Hour` — Chart D only
4. Add 4 **Buttons** at X = 168, Y = 460, each W = 114, H = 40 with 6px gap:
   - Labels: `Month` · `Day of Week` · `Time Band` · `Hour`
   - Action → Type: **Bookmark** → select matching bookmark
   - Style: same Slipstream tile buttons as §5.7 Step 1
5. Set `TD_Month` as the default state (Chart A visible, B/C/D hidden).

> **Pro tip (Option B):** Right-click each bookmark → edit → keep only **Display** checked (uncheck Data and Filters). This prevents the bookmark from freezing page-level slicer selections.

---

## 7. Page 3 — Sales & Revenue

> **Purpose:** Detailed breakdown of where money comes from by ticket class, type, payment method, and month.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] SALES & REVENUE                            [Month ▼]  [Ticket Class ▼]   │ ← Y = 0–60
├────────┬─────────────────────────────────────────────────────────────────────────┤
│        │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │ ← Y = 80
│  NAV   │  │Revenue  │ │Avg Price│ │1st Cl % │ │Advance £│ │Rev/Jrny │          │
│  160px │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │ ← Y = 180
│        ├────────────────────────────────────────────────┬─────────────────────────┤
│  UK    │  Revenue by Month & Ticket Type                │  Revenue vs Avg Price   │ ← Y = 200
│  Rail  │  (line + stacked col combo)                    │  (scatter chart)        │
│        │  X=168, W=700, H=250                           │  X=880, W=390, H=250    │
│        │                                                │                         │ ← Y = 450
│        ├──────────────────┬─────────────────────────────┼─────────────────────────┤
│        │  Rev by Ticket   │  Avg Ticket Price            │  Revenue by Payment     │ ← Y = 466
│        │  Class (donut)   │  Comparison (clustered bar)  │  Method (h-bar)         │
│        │  X=168, W=260    │  X=440, W=380                │  X=832, W=438           │
│        │                  │                              │                         │ ← Y = 686
└────────┴──────────────────┴──────────────────────────────┴─────────────────────────┘
```

### 7.1 KPI Cards Row (5 cards)

| Measure | Label |
|---|---|
| `Total Revenue` | Net Revenue |
| `Avg Ticket Price` | Avg Ticket Price |
| `First Class Revenue %` | First Class Share |
| `Advance Revenue` | Advance Revenue |
| `Revenue per Journey` | Revenue/Journey |

### 7.2 Revenue by Month & Ticket Type — Combo Chart

**Position:** X = 168, Y = 200 · **Size:** W = 700, H = 250

Use a **Line and Stacked Column Chart**:

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` |
| Column Y-axis | `Advance Revenue`, `Off-Peak Revenue`, `Anytime Revenue` |
| Line Y-axis | `Total Revenue` |

Column colours: Advance `#00B0F0`, Off-Peak `#29AB87`, Anytime `#F4A442`
Line colour: `#1F3864` (dark navy), width 3px
Title: `Monthly Revenue by Ticket Type`
Legend: Bottom

### 7.3 Revenue vs Average Price — Scatter Chart

**Position:** X = 880, Y = 200 · **Size:** W = 390, H = 250

| Field Well | Value |
|---|---|
| X-axis | `Avg Ticket Price` |
| Y-axis | `Total Revenue` |
| Legend | `Dim_Ticket[Ticket Class]` |
| Details | `Dim_Ticket[Ticket Type]` |
| Size | `Total Transactions` |

This shows which ticket combinations generate the most money and at what price point.
Colours: Standard `#00B0F0`, First Class `#4059AC`
Title: `Revenue vs Avg Price by Ticket Type`

### 7.4 Revenue by Ticket Class — Donut Chart

**Position:** X = 168, Y = 466 · **Size:** W = 260, H = 220

| Field Well | Value |
|---|---|
| Legend | `Dim_Ticket[Ticket Class]` |
| Values | `Total Revenue` |

Standard = `#00B0F0`, First Class = `#4059AC`
Inner label: card with `Standard Class %` over the hole
Title: `Revenue by Ticket Class`

### 7.5 Average Price Comparison — Clustered Bar Chart

**Position:** X = 440, Y = 466 · **Size:** W = 380, H = 220

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Ticket[Ticket Type]` |
| X-axis | `Avg Price Advance`, `Avg Price Off-Peak`, `Avg Price Anytime` |

> **How to do this:** Create a small **unpivot table** in a new page (hidden) or use a matrix visual:
>
> Alternatively, use a **Matrix** with:
> - Rows: `Dim_Ticket[Ticket Class]`
> - Columns: `Dim_Ticket[Ticket Type]`
> - Values: `Avg Ticket Price`
> - Apply data bars in Format → Cell elements → Data bars → on

Colours: Advance `#00B0F0`, Off-Peak `#29AB87`, Anytime `#F4A442`
Title: `Avg Ticket Price by Type`

### 7.6 Payment Method Revenue — Horizontal Bar

**Position:** X = 832, Y = 466 · **Size:** W = 438, H = 220

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Payment[Payment Method]` |
| X-axis | `Total Revenue` |
| Small multiples | `Dim_Payment[Purchase Type]` |

Colours: Contactless `#00B0F0`, Credit Card `#4059AC`, Debit Card `#29AB87`
Title: `Revenue by Payment Method`

### 7.7 Revenue by Journey Status — KPI-style cards

Add 3 small cards showing revenue by journey status:
- `CALCULATE([Total Revenue], Dim_Journey_Status[Journey Status]="On Time")` → `On-Time Revenue`
- `CALCULATE([Total Revenue], Dim_Journey_Status[Journey Status]="Delayed")` → `Delayed Revenue`
- `CALCULATE([Total Revenue], Dim_Journey_Status[Journey Status]="Cancelled")` → `Cancelled Revenue`

Position them as a small row at the bottom right of the page.

---

## 8. Page 4 — Railway Performance

> **Purpose:** Deep-dive into on-time performance, delay causes, refund patterns, and the impact of disruptions.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] RAILWAY PERFORMANCE                        [Month ▼]  [Ticket Class ▼]   │ ← Y = 0–60
├────────┬─────────────────────────────────────────────────────────────────────────┤
│        │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │ ← Y = 80
│  NAV   │  │On-Time% │ │Delay %  │ │Cancel % │ │Avg Delay│ │Refund £ │          │
│  160px │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │ ← Y = 180
│        ├────────────────────────────────────────┬────────────────────────────────┤
│  UK    │  Journey Status by Month               │  Disruptions by Reason         │ ← Y = 200
│  Rail  │  (100% stacked col + raw col pair)     │  (h-bar, reason not blank)     │
│        │  X=168, W=560, H=230                   │  X=740, W=530, H=230           │
│        │                                        │                                │ ← Y = 430
│        ├──────────────────────┬─────────────────┼────────────────────────────────┤
│        │  Disruptions vs      │  Delays >15 min │  Refund Rate by                │ ← Y = 446
│        │  Refund Requests     │  (gauge)        │  Delay Reason                  │
│        │  (clustered h-bar)   │  X=720, W=260   │  (matrix + conditional fmt)    │
│        │  X=168, W=540, H=240 │  H=240          │  X=992, W=278, H=240           │ ← Y = 686
└────────┴──────────────────────┴─────────────────┴────────────────────────────────┘
```

### 8.1 KPI Cards Row (5 cards)

| Measure | Label |
|---|---|
| `On Time Rate %` | On-Time Rate |
| `Delay Rate %` | Delay Rate |
| `Cancel Rate %` | Cancel Rate |
| `Avg Delay Minutes` | Avg Delay (min) |
| `Refund Exposure` | Refund Exposure |

**Conditional formatting on these cards:**
- `On Time Rate %`: background `#EBF3FB` if > 85%, `#FFF3CD` if 75-85%, `#FDEDEC` if < 75%
- `Delay Rate %` and `Cancel Rate %`: background `#FDEDEC` if above average

### 8.2 Journey Status by Month — Stacked Column Chart

**Position:** X = 168, Y = 200 · **Size:** W = 560, H = 230

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` (sorted) |
| Y-axis | `On Time Journeys`, `Delayed Journeys`, `Cancelled Journeys` |

Colours: On Time `#92D050`, Delayed `#F4A442`, Cancelled `#E74C3C`
Chart type: **100% Stacked Column** (shows proportion, not raw numbers)
Title: `Journey Status by Month (% Share)`
Add second chart alongside as clustered column showing raw numbers.

### 8.3 Delay Reasons — Horizontal Bar Chart

**Position:** X = 740, Y = 200 · **Size:** W = 530, H = 230

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Journey_Status[Reason for Delay]` |
| X-axis | `Total Disruptions` |
| Tooltip | `Delay Refund Rate %`, `Avg Delay Minutes` |

Sort: Descending by `Total Disruptions`
Filter: Exclude blank reason (on-time journeys) — add visual-level filter: `Reason for Delay is not blank`
Bar colour: `#E74C3C`
Title: `Disruptions by Reason`

**Pro tip:** Add a second value to show refund rate as data labels:
- Enable data labels → customise to show `Delay Refund Rate %`
  
### 8.4 Delay Reasons × Refund Rate — Clustered Bar (comparison)

**Position:** X = 168, Y = 446 · **Size:** W = 540, H = 240

| Field Well | Value |
|---|---|
| Y-axis | `Dim_Journey_Status[Reason for Delay]` |
| X-axis (bar 1) | `Total Disruptions` |
| X-axis (bar 2) | `Refund Requests` |

Use **Clustered Bar** chart
Filter: Reason not blank
Bar colours: Disruptions `#E74C3C`, Refund Requests `#F4A442`
Title: `Disruptions vs Refund Requests by Reason`

### 8.5 Delays Over 15 Min — Gauge or KPI Visual

**Position:** X = 720, Y = 446 · **Size:** W = 260, H = 240

Use a **Gauge** visual:
| Field Well | Value |
|---|---|
| Value | `Delays Over 15 Min` |
| Maximum value | `Delayed Journeys` |
| Target | (50% of delayed as a benchmark) |

Gauge needle colour: `#E74C3C`
Call-out text below gauge: `Delays Over 15 Min %` formatted as percentage
Title: `Delays > 15 Minutes`

### 8.6 Refund Analysis — Matrix

**Position:** X = 992, Y = 446 · **Size:** W = 278, H = 240

| Field Well | Value |
|---|---|
| Rows | `Dim_Journey_Status[Reason for Delay]` |
| Values | `Total Disruptions`, `Refund Requests`, `Delay Refund Rate %` |

Format:
- Column headers: Dark Navy background, white text
- Apply **data bars** on `Total Disruptions` column
- Apply **conditional formatting** (background colour) on `Delay Refund Rate %`:
  - Red scale: 0% = white → 100% = `#E74C3C`
- Filter: Reason not blank
- Title: `Refund Rate by Delay Reason`

---

## 9. Page 5 — Route Analysis

> **Purpose:** Identify the most popular and most profitable routes, and where delays concentrate.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] ROUTE ANALYSIS                             [Month ▼]  [Ticket Class ▼]   │ ← Y = 0–60
├────────┬─────────────────────────────────────────────────────────────────────────┤
│        │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │ ← Y = 80
│  NAV   │  │Total Routes │ │Total Journys│ │Total Revenue│ │Top Rt Jrnys │       │
│  160px │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘       │ ← Y = 180
│        ├──────────────────────────────────────────┬─────────────────────────────┤
│  UK    │  Top 10 Routes by Journeys               │  Top 10 Routes by Revenue    │ ← Y = 200
│  Rail  │  (h-bar, top-N filter, descending)       │  (h-bar, gradient colour)    │
│        │  X=168, W=530, H=480                     │  X=710, W=560, H=480         │
│        │                                          │                              │
│        │  Both charts span full content height    │                              │
│        │  — intentionally tall for readability    │                              │ ← Y = 680
└────────┴──────────────────────────────────────────┴─────────────────────────────┘
```

### 9.1 KPI Cards Row (4 cards)

| Measure | Label |
|---|---|
| `Unique Routes` | Total Routes |
| `Total Transactions (Journey Date)` | Total Journeys |
| `Total Revenue (Journey Date)` | Total Revenue |
| `Top Route Journeys` | Top Route Journeys |

### 9.2 Top 10 Routes by Journeys — Horizontal Bar Chart

**Position:** X = 168, Y = 200 · **Size:** W = 530, H = 480

| Field Well | Value |
|---|---|
| Y-axis | `Fact_Transactions[Route]` |
| X-axis | `Total Transactions (Journey Date)` |
| Tooltip | `Total Revenue (Journey Date)`, `On Time Rate %`, `Delay Rate %` |

- Filter (visual level): Top N = 10 by `Total Transactions (Journey Date)`
- Sort: Descending
- Bar colour: `#00B0F0`
- Data labels: ON
- Title: `Top 10 Routes by Journeys`

### 9.3 Top 10 Routes by Revenue — Horizontal Bar Chart

**Position:** X = 710, Y = 200 · **Size:** W = 560, H = 480

| Field Well | Value |
|---|---|
| Y-axis | `Fact_Transactions[Route]` |
| X-axis | `Total Revenue (Journey Date)` |
| Tooltip | `Total Transactions (Journey Date)`, `Avg Ticket Price`, `On Time Rate %` |

- Filter: Top 10 by `Total Revenue (Journey Date)`
- Sort: Descending
- Bar colour: gradient: low = `#9DC3E6`, high = `#4059AC`
  - Enable: Format → Data colours → **Conditional formatting** → Gradient → Min colour `#9DC3E6`, Max colour `#4059AC`
- Data labels: ON (formatted as £#,##0)
- Title: `Top 10 Routes by Revenue`

### 9.4 Route On-Time Performance — Matrix

Place a **Matrix** visual on a second row (if space allows on this page, or add it as a drill-through destination):

| Field Well | Value |
|---|---|
| Rows | `Fact_Transactions[Route]` |
| Values | `Total Transactions (Journey Date)`, `On Time Rate %`, `Delay Rate %`, `Cancel Rate %`, `Total Revenue (Journey Date)` |

- Filter: Top 15 routes by journeys
- Conditional formatting:
  - `On Time Rate %`: Green scale (low `#FDEDEC` → high `#92D050`)
  - `Delay Rate %`: Red scale (low white → high `#E74C3C`)
- Column headers: Dark Navy, white text
- Title: `Route Performance Summary`

---

## 10. Page 6 — Scenario Simulator

> **Purpose:** A dedicated "what could happen" page for live scenario modelling. Users drag two sliders to model different refund eligibility thresholds and ticket price adjustments — watching key metrics update in real time.
>
> **Why a separate page?** Field Parameters (§16) enhance existing charts in-place — they are UI polish. What-If sliders change the mathematical *assumptions* of the model. Mixing descriptive analytics ("what happened") with prescriptive modelling ("what if") on the same page creates cognitive confusion. A dedicated Scenario page signals clearly: you have left the historical record and entered the simulator.

```
Canvas: 1280 × 720
┌──────────────────────────────────────────────────────────────────────────────────┐
│ [icon] SCENARIO SIMULATOR  ·  subtitle: operational impact modelling             │ ← Y = 0–55
├──────────────────────────────────────────────────────────────────────────────────┤
│ Readout: Delay Threshold: [card]  │  Price Adjustment: [card]  (live values)    │ ← Y = 55–87
├────────┬────────────────────────────────────────┬───────────────────────────────┤
│        │   REFUND POLICY SIMULATOR              │   PRICE SCENARIO SIMULATOR    │ ← Y = 90
│  NAV   │   ──────────────────────────────       │   ─────────────────────────   │
│  160px │  [ Delay Threshold slider ]            │  [ Price Adjustment slider ]  │ ← Y = 140
│        │                                        │                               │
│  UK    │  ┌───────┐  ┌────────┐  ┌────────┐    │  ┌─────┐┌─────┐┌─────┐┌─────┐│ ← Y = 215
│  Rail  │  │ Jrnys │  │Refund  │  │Policy  │    │  │Actl ││Siml.││ Δ £ ││ Δ % ││
│        │  │ Above │  │Exposre │  │Saving  │    │  │ Rev ││ Rev ││     ││     ││ ← Y = 305
│        │  └───────┘  └────────┘  └────────┘    │  └─────┘└─────┘└─────┘└─────┘│
│        │                                        │                               │ ← Y = 336
│        │  Affected Journeys vs Total            │  Actual vs Simulated Revenue  │
│        │  (line + col combo chart)              │  (line + col combo chart)     │
│        │  X=178, W=516, H=370                   │  X=730, W=530, H=354          │ ← Y = 710
└────────┴────────────────────────────────────────┴───────────────────────────────┘
```

### 10.1 Page Setup

1. Right-click any page tab → **Add page** → rename it `Scenario Simulator`.
2. Set canvas: W = 1280, H = 720.
3. Background: `#F0F5FB`.

### 10.2 Page Header

1. Insert a **Rectangle**: X = 0, Y = 0, W = 1280, H = 55, fill `#1F3864`, no border.
2. Insert a **Text box** on top: X = 168, Y = 8, W = 850, H = 26.
   - Text: `SCENARIO SIMULATOR` · Segoe UI Bold · 20pt · White
3. Insert a subtitle text box: X = 168, Y = 32, W = 850, H = 18.
   - Text: `Model the revenue and policy impact of key operational decisions`
   - Segoe UI Light · 10pt · `#9DC3E6`

### 10.3 Active Scenario Readout Bar

This thin strip immediately below the header shows the user exactly which slider values are active — critical UX when the sliders are positioned lower on the page and the user has scrolled or navigated away from them.

1. Insert a **Rectangle**: X = 168, Y = 57, W = 1102, H = 30, fill `#EBF3FB`, border `#9DC3E6` 1px.
2. Insert a **Text box**: X = 178, Y = 61, W = 180, H = 20.
   - Text: `Delay Threshold (mins):` · Segoe UI · 10pt · `#4059AC`
3. Insert a **Card** visual (transparent fill, no border): X = 360, Y = 57, W = 60, H = 30.
   - Field: `[Selected Delay Threshold]` · Category label: hidden · Value: Segoe UI Bold 12pt `#1F3864`
4. Insert a separator text box: X = 428, Y = 61 · Text: `│` · `#9DC3E6`
5. Insert a **Text box**: X = 444, Y = 61, W = 180, H = 20.
   - Text: `Price Adjustment (%):` · Segoe UI · 10pt · `#4059AC`
6. Insert a **Card** visual: X = 626, Y = 57, W = 60, H = 30.
   - Field: `[Selected Price Adjustment]` · Category label: hidden · Value: Segoe UI Bold 12pt `#1F3864`
7. Insert a text box at the right edge: X = 900, Y = 61, W = 360, H = 20.
   - Text: `Drag the sliders below — all visuals update in real time`
   - Segoe UI Italic · 9pt · `#4D5F7A`

### 10.4 Panel Divider

1. Insert a **Line** shape: X = 712, Y = 90, H = 620.
2. Colour: `#9DC3E6`, weight: 1.5px.
3. This cleanly divides the canvas into the two scenario panels.

### 10.5 Left Panel — Refund Policy Simulator

**Panel area:** X = 168 · Y = 90 · W = 536 · H = 620

#### Panel header strip
1. Insert a **Rectangle**: X = 168, Y = 90, W = 536, H = 36, fill `#4059AC`, no border.
2. Insert a **Text box**: X = 178, Y = 96, W = 516, H = 24.
   - Text: `REFUND POLICY SIMULATOR` · Segoe UI Bold · 12pt · White

#### Delay Threshold slider

**Position:** X = 178, Y = 140 · **Size:** W = 516, H = 58

1. **Insert** → **Slicer** · Drag `Delay Threshold Minutes[Value]` onto the field.
2. Format → Slicer settings → Options → Style: **Greater than or equal to**.
3. Slicer header: `Delay Eligibility Threshold (minutes)` · Segoe UI Bold · 11pt · `#1F3864`
4. Slider fill: `#00B0F0` · Values font: Segoe UI 10pt `#4059AC`.

#### KPI Cards row — 3 cards

**Row position:** Y = 215 · Each card W = 165, H = 90 · 6px gap between cards

| Card | X | Measure | Category Label | Accent Border |
|---|---|---|---|---|
| 1 | 178 | `[Journeys Above Threshold]` | Journeys Above Threshold | `#F4A442` |
| 2 | 349 | `[Refund Exposure Above Threshold]` | Refund Exposure | `#E74C3C` |
| 3 | 520 | `[Refund Policy Saving]` | Policy Saving | `#92D050` |

- Card fill: `#FFFFFF` · Value: Segoe UI Bold 20pt `#1F3864`
- **Conditional background on Card 2** (`[Refund Exposure Above Threshold]`):
  - ≥ 10000 → `#FDEDEC` | ≥ 5000 → `#FEF9E7` | < 5000 → `#EAFAF1`

#### Trend chart — Affected Journeys vs Total by Month

**Position:** X = 178, Y = 320 · **Size:** W = 516, H = 370

Use a **Line and Clustered Column Chart**:

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` |
| Column Y-axis | `[Total Transactions (Journey Date)]` |
| Line Y-axis | `[Journeys Above Threshold]` |

- Columns: `#9DC3E6` at 50% opacity (context — total journey count stays fixed)
- Line: `#E74C3C` dashed, weight 2.5px (the affected count — eye goes here first)
- Data labels: ON for line only
- Title: `Affected Journeys vs Total — by Month`
- Legend: ON (bottom)

> **Reading this chart:** Drag the threshold slider right (stricter policy) and the red line drops — fewer journeys qualify for a refund. Drag it left (lenient policy) and it rises. The grey columns stay constant, providing a reference baseline.

### 10.6 Right Panel — Price Scenario Simulator

**Panel area:** X = 720 · Y = 90 · W = 550 · H = 620

#### Panel header strip
1. Insert a **Rectangle**: X = 720, Y = 90, W = 550, H = 36, fill `#4059AC`, no border.
2. Insert a **Text box**: X = 730, Y = 96, W = 530, H = 24.
   - Text: `PRICE SCENARIO SIMULATOR` · Segoe UI Bold · 12pt · White

#### Price Adjustment slider

**Position:** X = 730, Y = 140 · **Size:** W = 530, H = 58

1. **Insert** → **Slicer** · Drag `Price Adjustment %[Value]` onto the field.
2. Format → Slicer settings → Options → Style: **Greater than or equal to**.
3. Slicer header: `Price Adjustment Scenario (%)` · Segoe UI Bold · 11pt · `#1F3864`
4. Slider fill: `#00B0F0` · Values font: Segoe UI 10pt `#4059AC`.
5. Add a note text box: X = 730, Y = 200, W = 530, H = 16.
   - Text: `0 = baseline  ·  Positive = price increase  ·  Negative = discount scenario`
   - Segoe UI Italic · 9pt · `#4D5F7A`

#### KPI Cards row — 4 cards

**Row position:** Y = 230 · Each card W = 124, H = 90 · 6px gap

| Card | X | Measure | Category Label |
|---|---|---|---|
| 1 | 730 | `[Total Revenue]` | Actual Revenue |
| 2 | 860 | `[Simulated Revenue]` | Simulated Revenue |
| 3 | 990 | `[Revenue Delta]` | Delta (£) |
| 4 | 1120 | `[Revenue Delta %]` | Delta (%) |

- Cards fill: `#FFFFFF` · Value: Segoe UI Bold 18pt `#1F3864`
- Card 1 border: `#9DC3E6` · Card 2 border: `#00B0F0` (highlights the scenario value)
- **Conditional font colour on Cards 3 and 4:**
  - Value > 0 → `#27AE60` (green) | Value < 0 → `#E74C3C` (red) | Value = 0 → `#4059AC`

#### Combo chart — Actual vs Simulated Revenue by Month

**Position:** X = 730, Y = 336 · **Size:** W = 530, H = 354

Use a **Line and Clustered Column Chart**:

| Field Well | Value |
|---|---|
| X-axis | `Dim_Date[Month Name]` |
| Column Y-axis | `[Total Revenue]` |
| Line Y-axis | `[Simulated Revenue]` |

- Columns: `#4059AC` (actual — authoritative, solid)
- Line: `#92D050` dashed, weight 2.5px (simulated — the "could be" line)
- Data labels: ON for both
- Tooltip: add `[Revenue Delta]`, `[Revenue Delta %]`
- Title: `Actual vs Simulated Revenue — by Month`
- Legend: ON (bottom)

> **Reading this chart:** At 0% the green line sits exactly on the blue bars. Move to +10% and the line floats above — you see the potential uplift month by month. Move to −15% and it dips below — the cost of a discount strategy made visible before you commit to it.

### 10.7 Navigation

Add the Simulator nav button to every page's sidebar — see Section 11 for button creation steps. Suggested label: `📈 Simulator`. Apply a **teal** (`#29AB87`) hover fill instead of the standard cyan — this small difference signals visually that Simulator is a different type of page from the descriptive analysis pages.

---

## 11. Navigation Buttons

Add consistent page-navigation buttons on the **sidebar** of every page (except Cover).

### 11.1 Create a button

1. Go to **Insert** → **Buttons** → **Blank**.
2. Set size: W = 140, H = 42.
3. Position in the left sidebar (X = 10), stacked vertically starting at Y = 120, with 8px gap.

### 11.2 Style the button

In **Format Visual** → **Button**:
- **Shape:** Rounded corner (radius 6px)
- **Default state:**
  - Fill: `#2A4A7F` (slightly lighter than sidebar)
  - Font: Segoe UI 10pt, White, Bold OFF
  - Border: off
- **On hover state:**
  - Fill: `#00B0F0` (cyan)
  - Font: White, Bold ON

### 11.3 Add page navigation action

1. Select the button.
2. In **Format Visual** → **Action**: toggle ON.
3. Type: **Page navigation**.
4. Destination: select the target page.

### 11.4 Button labels (with icons)

Create one button per page with these labels:

```
📊 Overview
🧑 Passengers
💰 Revenue
🚦 Performance
🗺️ Routes
📈 Simulator
```

> **Tip:** Copy the button 6 times (Ctrl+D), then change the text and destination on each copy. For the Simulator button, change the **hover fill** to `#29AB87` (teal) to visually distinguish it from the analytical pages.

### 11.5 Active page indicator

Power BI button borders apply uniformly to all four sides — use this to add a cyan outline to the active page's button.

#### Active vs inactive button styles

| Property | Active (current page) | Inactive (other pages) |
|---|---|---|
| Fill | `#1F3864` (dark navy) | `#2A4A7F` (mid navy) |
| Border | `#00B0F0`, 2px, all sides | None |
| Font weight | Bold ON | Bold OFF |

#### Step-by-step

**On each analysis page, style the active button's Default state:**

1. Select the nav button for the current page.
2. **Format Visual** → **Button** → **Default** state:
   - Fill: `#1F3864`
   - Border: ON · Colour `#00B0F0` · Width: **2px**
   - Font: Bold ON
3. Leave all other nav buttons at the standard inactive Default style (no border).
4. Repeat on every page — one button per page gets the active border style.

> **Why not the native Selected state?** Power BI's Selected format is designed for within-page toggle groups. It does not survive cross-page navigation — the border approach baked into Default state is the correct method.

---

## 12. Sync Slicers Across Pages

The **Month** and **Ticket Class** slicers should filter all pages simultaneously.

1. Go to **View** → **Sync Slicers**.
2. Select the **Month slicer** on Page 1.
3. In the Sync Slicers panel, tick the **Sync** checkbox for Pages 1, 2, 3, 4, 5, and 6 (Scenario Simulator).
4. Tick the **Visible** checkbox for Pages 1–6 (show the slicer on all content pages including Simulator — month context makes the scenario trend charts more insightful).
5. Repeat for the **Ticket Class** slicer (Pages 1–5 only — the Simulator works best unfiltered by class so users see the aggregate).

> **Note:** The Cover page should NOT be synced since it has no data visuals. The `Metric Selector` and `Time Dimension` slicers on Pages 1 and 2 are **not** synced globally — they are page-specific controls.

---

## 13. Conditional Formatting on KPI Cards

Make cards change colour based on performance thresholds.

### 12.1 On-Time Rate card

1. Select the **On Time Rate %** card.
2. In **Format** pane → **Callout value** → Font colour → **Conditional formatting** (fx button).
3. Set rules:
   - If value ≥ 0.85 → Colour `#27AE60` (green)
   - If value ≥ 0.75 and < 0.85 → Colour `#F4A442` (orange)
   - If value < 0.75 → Colour `#E74C3C` (red)

### 12.2 MoM change cards

For the `Revenue MoM %` card:
1. Background conditional format:
   - Value > 0 → `#E8F8F0` (light green tint)
   - Value < 0 → `#FDEDEC` (light red tint)
2. Font colour:
   - Value > 0 → `#27AE60`
   - Value < 0 → `#E74C3C`

---

## 14. Tooltip Pages

Tooltip pages appear when you hover over a data point and show extra detail.

### 13.1 Create a Route Tooltip page

1. **Right-click** any page tab → **Add page**.
2. Rename it `Tooltip_Route`.
3. In **Format** pane → **Page information**:
   - Allow use as tooltip: **ON**
   - Canvas size: **Tooltip** (320 × 240)
   - Background: `#1F3864` (dark navy)
4. Add these visuals (small, compact):
   - Card: `Total Transactions (Journey Date)` — white font
   - Card: `Total Revenue (Journey Date)` — white font
   - Card: `On Time Rate %` — conditional font colour
   - Mini bar: `Dim_Journey_Status[Journey Status]` × `Total Transactions`

5. **Apply the tooltip** to the Route bar charts:
   - Select the Route bar chart visual
   - Format → **Tooltip** → Type: **Report page** → Page: `Tooltip_Route`

### 13.2 Create a Delay Reason Tooltip page

Repeat the process with page name `Tooltip_Delay`:
- Cards: `Total Disruptions`, `Delay Refund Rate %`, `Avg Delay Minutes`
- Mini bar: monthly trend of that specific delay reason

---

## 15. Final Polish Checklist

Work through this list before publishing or sharing your report.

### Typography
- [ ] All visual titles: Segoe UI, 12pt, Bold, `#1F3864`
- [ ] All axis labels: Segoe UI, 10pt, `#4D5F7A`
- [ ] All data labels: Segoe UI, 9pt, `#2C3E50`
- [ ] All card category labels: Segoe UI, 11pt, `#4059AC`
- [ ] All card values: Segoe UI, 26pt, Bold, `#1F3864`

### Alignment
- [ ] Use **Format** → **Align** to align all cards in a row to their top edges
- [ ] Use equal **Distribute horizontally** spacing between cards
- [ ] All charts within a row should share the same top Y position and height
- [ ] All sidebar buttons are evenly spaced vertically

### Accessibility
- [ ] All visuals have descriptive **Alt text** (Format → General → Alt text)
- [ ] Colour-only differences always have a second indicator (shape, pattern, or label)
- [ ] Tab order set via **View** → **Tab order** for screen-reader friendliness

### Interactivity
- [ ] Cross-filtering tested: clicking a bar on one chart filters the others
- [ ] All slicers synced across pages (Section 11)
- [ ] All navigation buttons tested
- [ ] Tooltip pages appear correctly on hover

### Performance
- [ ] Hidden the `_Placeholder` column in `_Measures` table (right-click → Hide)
- [ ] Set all key columns used for filtering to **Don't summarise** (not Sum/Count)
- [ ] Unnecessary columns hidden from Report view: all FK integer keys in `Fact_Transactions`
  - Right-click each key column in Data pane → **Hide in report view**:
    `Transaction SK`, `Purchase Date Key`, `Journey Date Key`, `Departure Station Key`,
    `Arrival Station Key`, `Ticket Key`, `Payment Key`, `Journey Status Key`

### Final Steps
- [ ] Save the file (Ctrl + S)
- [ ] Export TMDL folder (via MCP server or File → Save)
- [ ] Add report-level filter if needed (e.g. filter to Jan–Apr 2024 only)
- [ ] Test on a smaller screen (set canvas view to 100% and resize browser)
- [ ] Take a screenshot of each page for portfolio/README

---

## Quick Reference — All Measures

| Folder | Measure | Typical Visual |
|---|---|---|
| **Volume** | `Total Transactions (Journey Date)` | Card, Line Chart axis |
| **Volume** | `On Time Journeys` | Card, Stacked Col |
| **Volume** | `Delayed Journeys` | Card, Stacked Col |
| **Volume** | `Cancelled Journeys` | Card, Stacked Col |
| **Volume** | `Total Peak Journeys` | Card |
| **Volume** | `Morning Peak Journeys` | Card |
| **Volume** | `Afternoon Peak Journeys` | Card |
| **Volume** | `Railcard Transactions` | Card, Donut |
| **Volume** | `Standard Transactions` | Donut |
| **Volume** | `First Class Transactions` | Donut |
| **Volume** | `Total Disruptions` | Card |
| **Volume** | `Online Transactions` | Bar chart |
| **Rates** | `On Time Rate %` | Card, Gauge |
| **Rates** | `Delay Rate %` | Card |
| **Rates** | `Cancel Rate %` | Card |
| **Rates** | `Peak Journey Rate %` | Card |
| **Rates** | `Railcard Usage Rate %` | Card |
| **Rates** | `Standard Class %` | Card |
| **Rates** | `Online Purchase Rate %` | Card |
| **Revenue** | `Total Revenue` | Card, Column Chart |
| **Revenue** | `Avg Ticket Price` | Card, Scatter |
| **Revenue** | `Revenue per Journey` | Card |
| **Revenue** | `Total Revenue (Journey Date)` | Line Chart |
| **Revenue** | `Total Revenue (Arrival Station)` | Bar Chart |
| **Revenue** | `First Class Revenue` | Donut |
| **Revenue** | `Standard Revenue` | Donut |
| **Revenue** | `First Class Revenue %` | Card |
| **Revenue** | `Advance Revenue` | Stacked Col |
| **Revenue** | `Off-Peak Revenue` | Stacked Col |
| **Revenue** | `Anytime Revenue` | Stacked Col |
| **Revenue** | `Avg Price First Class` | Bar, Matrix |
| **Revenue** | `Avg Price Standard` | Bar, Matrix |
| **Revenue** | `Revenue MoM %` | Card (with CF) |
| **Refunds** | `Refund Requests` | Card |
| **Refunds** | `Refund Request Rate %` | Card, Matrix |
| **Refunds** | `Refund Exposure` | Card |
| **On-Time Performance** | `Avg Delay Minutes` | Card, Gauge |
| **On-Time Performance** | `Max Delay Minutes` | Card |
| **On-Time Performance** | `Delays Over 15 Min` | Card, Gauge |
| **On-Time Performance** | `Delays Over 15 Min %` | Card |
| **On-Time Performance** | `Delay Refund Rate %` | Matrix, Bar |
| **On-Time Performance** | `Signal Failure Disruptions` | Bar |
| **On-Time Performance** | `Weather Disruptions` | Bar |
| **On-Time Performance** | `Technical Issue Disruptions` | Bar |
| **On-Time Performance** | `Staffing Disruptions` | Bar |
| **Routes** | `Unique Routes` | Card |
| **Routes** | `Top Route Journeys` | Card |
| **Routes** | `Top Route Revenue` | Card |
| **Peak Travel** | `Peak Revenue` | Card |

---

*Happy building! The key to a great dashboard is consistent spacing, a restrained colour palette,
and telling a clear story. Every visual should answer one specific question — if it doesn't,
consider removing it.*

---

## 16. Field Parameters & Dynamic Metric Switching

> **Where these live in the report:**
> - `Metric Selector` button slicer + `[Dynamic KPI]` chart → built directly inside **Section 5.7** on the **Executive Overview** page (Page 1)
> - `Time Dimension` switcher → built directly inside **Section 6.8** on the **Passenger Usage** page (Page 2)
>
> Field Parameters are UI enhancers. They live *inside* the pages they improve and make those pages interactive. They do **not** need their own page — that would separate the control from its context, which is bad UX.

Field Parameters let a single chart display **different measures** depending on which button the
user clicks — no duplicated visuals required.

This report uses two disconnected selector tables as pseudo Field Parameters:
- **`Metric Selector`** — switches between 6 KPIs in one chart (Total Journeys, Net Revenue,
  On-Time Rate %, Delay Rate %, Cancel Rate %, Avg Ticket Price)
- **`Time Dimension`** — switches between time-axis breakdowns (Month, Day of Week, Time Band,
  Departure Hour)

### 15.1 How the pattern works

```
 Slicer (Button style)
  └─ Field: Metric Selector[Metric Name]
        │
        │  User clicks "Net Revenue"
        ▼
 SELECTEDVALUE('Metric Selector'[Metric Name], "Total Journeys")
        │  Returns "Net Revenue"
        ▼
 SWITCH([Selected Metric], ...) = [Total Revenue]
        │
        ▼
  Chart Y-axis: [Dynamic KPI]
```

The chart axis never changes — only the underlying measure returned by `[Dynamic KPI]` changes.

### 15.2 Adding the Metric Selector button slicer

1. On the **Executive Overview** page (Page 1), click **Insert** → **Slicer**.
2. In the **Field** well, drag `Metric Selector[Metric Name]` onto the slicer.
3. Open **Format visual** → **Slicer settings** → **Options** → **Style** → choose **Tile**.
4. In **Format visual** → **Slicer header**, turn **Off** (remove the header to save space).
5. Resize the slicer to be a single horizontal row of 6 buttons.
6. Apply button styling:
   - **Values** → Background color: `#EBF3FB` | Font color: `#1F3864` | Bold: On
   - **Selected state** → Background: `#00B0F0` | Font color: `#FFFFFF`
   - **Hover state** → Background: `#9DC3E6`
   - Border: `#9DC3E6`, 1 px
7. Sort the slicer by `Sort Order` (right-click the slicer → Sort by → Sort Order → Ascending).

### 15.3 Connecting a chart to Dynamic KPI

1. Insert a **Clustered Bar Chart** or **Line Chart** on Page 1.
2. In the **Y-axis** field well, place `[Dynamic KPI]` (from `_Measures`).
3. In the **X-axis**, place `Dim_Date[Month Name]` (or another dimension).
4. Set the chart title to: `_Measures[Selected Metric]` — Power BI will dynamically update
   the title as the user changes selection.
   - Format visual → General → Title → Title text: enter the measure reference manually or
     type a static title like *"Selected Metric Trend"*.
5. Add a data label format that matches the selected metric. Use `[Dynamic KPI Label]` in a
   **Card** visual beside the chart to always show the correct formatted total.

### 15.4 Using Time Dimension for axis switching

Because Power BI cannot swap axis fields dynamically without native Field Parameters (which
require enabling the Preview feature), use a **bookmark + button** approach:

1. Create 4 duplicate charts — one per time breakdown (Month / Day of Week / Time Band / Hour).
2. Layer them on top of each other (same position and size).
3. Create 4 Bookmarks (View → Bookmarks) — one per chart showing only that chart.
4. Add a **Time Dimension** Button Slicer (same steps as 15.2).
5. Assign each bookmark to the corresponding slicer button via **Button** → **Action** →
   **Type: Bookmark** → select the matching bookmark.

> **Tip:** Name your bookmarks clearly: `TD_Month`, `TD_DayOfWeek`, `TD_TimeBand`, `TD_Hour`.

### 15.5 Using native Field Parameters (Preview feature)

If you want to use Power BI's built-in **Field Parameters** feature:

1. In Power BI Desktop, go to **File** → **Options and settings** → **Options** →
   **Preview features** → enable **Field parameters** → restart Power BI Desktop.
2. On the Modelling tab, click **New parameter** → **Fields**.
3. Name it `Metric Selector Fields` and add these measures:
   - `[Total Transactions (Journey Date)]`, `[Total Revenue]`, `[On Time Rate %]`,
     `[Delay Rate %]`, `[Cancel Rate %]`, `[Avg Ticket Price]`
4. Power BI creates the parameter table and slicer automatically.
5. Drop the generated `Metric Selector Fields` field onto the Y-axis of your chart.
6. The slicer now controls which measure the chart plots — no DAX SWITCH required.

---

## 17. Button Slicers Setup

Button Slicers transform standard list slicers into clickable pill/tile buttons that match the
Slipstream design system.

### 16.1 General setup steps (applies to all button slicers)

1. **Insert** → **Slicer** → drag the desired field onto the slicer.
2. **Format visual** → **Slicer settings** → **Options** → **Style** → **Tile** (for pills) or
   **Dropdown** for compact space. For navigation-style selectors use **Tile**.
3. Set orientation:
   - **Single row** (horizontal): Resize to short height (~40 px), wide width.
   - **Single column** (vertical): Narrow width (~180 px), tall height.
4. Turn off the **Slicer header** (Format → Slicer header → Off) for a cleaner look.
5. Apply Slipstream button styling:

| State | Fill | Font Color | Border |
|---|---|---|---|
| Default | `#EBF3FB` | `#1F3864` | `#9DC3E6` 1px |
| Selected | `#00B0F0` | `#FFFFFF` | `#00B0F0` 1px |
| Hover | `#9DC3E6` | `#1F3864` | `#9DC3E6` 1px |

### 16.2 Slicers used in this dashboard

| Slicer | Field | Page | Layout |
|---|---|---|---|
| Month slicer | `Dim_Date[Month Name]` | All pages | Horizontal tile row |
| Ticket class | `Dim_Ticket[Ticket Class]` | Revenue | Horizontal tile |
| Journey status | `Dim_Journey_Status[Journey Status]` | Performance | Horizontal tile |
| Metric Selector | `Metric Selector[Metric Name]` | Overview | Horizontal tile |
| Time Dimension | `Time Dimension[Dimension Name]` | Passenger Usage | Horizontal tile |
| Delay threshold | `Delay Threshold Minutes[Value]` | Performance | Slider (see §17) |
| Price adjustment | `Price Adjustment %[Value]` | Revenue | Slider (see §17) |

### 16.3 Sync slicers across pages

1. Select the **Month slicer** (or any slicer that should apply to all pages).
2. Go to **View** → **Sync slicers**.
3. In the **Sync slicers** pane, check ✅ both columns (sync + visible) for every page where
   the slicer should operate.
4. Hide the slicer on pages where it should filter silently but not be shown (check **Sync** only).

> **Important:** The `Metric Selector` and `Time Dimension` slicers are **page-specific** — do NOT
> sync them globally or they will interfere with other pages.

### 16.4 Using Select All / multi-select

- For month/status slicers: Format → Slicer settings → Selection → enable **Multi-select with Ctrl**
  or **Select all** as needed.
- For `Metric Selector` and `Time Dimension`: Force **single select** (leave Multi-select off) so
  only one dimension is active at a time.

---

## 18. What-If Parameters (Numeric Sliders)

> **Where these live in the report:**
> Both What-If scenario panels are built on **Page 6 — Scenario Simulator** (Section 10). They are deliberately **not** placed on the Revenue or Performance pages because they represent a different analytical mode: modelling *future possibilities* rather than describing *historical facts*. Mixing them into the analytical pages would dilute the clean narrative of those pages and confuse viewers about whether they are looking at actuals or projections.
>
> The sections below provide the slider and visual build instructions you will follow when constructing Page 6.

The model includes two What-If sliders that let viewers run live scenarios without editing any data.

### 17.1 What-If tables in the model

| Table | Range | Step | Column name | Default |
|---|---|---|---|---|
| `Delay Threshold Minutes` | 0 – 60 | 5 min | `[Value]` | 15 |
| `Price Adjustment %` | −30 – +30 | 5% | `[Value]` | 0 |

### 17.2 Converting the slicer to a Slider style

1. **Insert** → **Slicer** → drag `Delay Threshold Minutes[Value]` onto the slicer.
2. **Format visual** → **Slicer settings** → **Options** → **Style** → **Between** (this gives
   a min/max range slider) — or choose **Slider** if available in your PBI version.
3. Resize to a narrow horizontal strip (~400 px wide, ~50 px tall).
4. Apply styling:
   - **Slicer header**: "Refund Eligibility Threshold (mins)" — Segoe UI, 11pt, Bold, `#1F3864`
   - **Values**: Segoe UI, 10pt, `#4059AC`
   - **Slider bar fill**: `#00B0F0`
5. Repeat for `Price Adjustment %[Value]` with header "Price Adjustment Scenario (%)".

### 17.3 Delay Threshold scenario panel

Build a scenario panel on the **Railway Performance** page (Page 4):

```
┌──────────────────────────────────────────────────────┐
│  REFUND POLICY SCENARIO                              │
│  ──────────────────────────────────────────────────  │
│  Refund Eligibility Threshold (mins)                 │
│  [═══════●══════════════════════] 15                 │
│                                                      │
│  ┌─────────────────┐  ┌─────────────────┐           │
│  │ Journeys Above  │  │ Refund Exposure │           │
│  │    Threshold    │  │  Above Threshold│           │
│  │     1,234       │  │    £12,450      │           │
│  └─────────────────┘  └─────────────────┘           │
│                                                      │
│  ┌─────────────────────────────────────┐            │
│  │ Refund Policy Saving                │            │
│  │ (Refunds paid below threshold)      │            │
│  │             £3,210                  │            │
│  └─────────────────────────────────────┘            │
└──────────────────────────────────────────────────────┘
```

**Step-by-step:**
1. Draw a **Rectangle** shape: fill `#EBF3FB`, border `#9DC3E6` 1px, corner radius 6px.
2. Add a **Text box** inside: "REFUND POLICY SCENARIO", Segoe UI Bold 12pt `#1F3864`.
3. Insert the `Delay Threshold Minutes` slider (§17.2) inside the rectangle.
4. Insert 3 **New Card** visuals:
   - Card 1: `[Journeys Above Threshold]` — Category label: "Journeys Above Threshold"
   - Card 2: `[Refund Exposure Above Threshold]` — Category label: "Refund Exposure"
   - Card 3: `[Refund Policy Saving]` — Category label: "Saving (Below Threshold)"
5. Apply conditional formatting to Card 2 (`Refund Exposure Above Threshold`):
   - Background → Rules: value > 10000 = `#E74C3C` opacity 20%; value > 5000 = `#F4A442` 20%

### 17.4 Price Adjustment scenario panel

Build on the **Sales & Revenue** page (Page 3):

```
┌──────────────────────────────────────────────────────┐
│  PRICE SIMULATION                                    │
│  ──────────────────────────────────────────────────  │
│  Price Adjustment (%)                                │
│  [══════════●════════════════════] 0%               │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │ Actual       │  │ Simulated    │  │  Delta    │ │
│  │ Revenue      │  │ Revenue      │  │           │ │
│  │  £741,921    │  │  £741,921    │  │  £0       │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
└──────────────────────────────────────────────────────┘
```

**Step-by-step:**
1. Draw a **Rectangle** shape same as §17.3.
2. Add header text box: "PRICE SIMULATION".
3. Insert the `Price Adjustment %` slider (§17.2).
4. Insert 3 **New Card** visuals:
   - Card 1: `[Total Revenue]` — Category label: "Actual Revenue"
   - Card 2: `[Simulated Revenue]` — Category label: "Simulated Revenue"
   - Card 3: `[Revenue Delta]` — Category label: "Δ Revenue"
5. Apply conditional formatting to Card 3 (`Revenue Delta`):
   - **Font color** rules:
     - If value > 0 → `#92D050` (green)
     - If value < 0 → `#E74C3C` (red)
     - If value = 0 → `#4059AC` (blue)
6. Add an optional **Line & Stacked Column** combo chart below the cards:
   - Columns: `Dim_Date[Month Name]` on X-axis, `[Total Revenue]` as column
   - Line: `[Simulated Revenue]`
   - This lets viewers see month-by-month how the simulation plays out.

### 17.5 Measures reference for What-If scenarios

| Measure | Table | Description |
|---|---|---|
| `Selected Delay Threshold` | `_Measures` | Current slider value (default 15) |
| `Journeys Above Threshold` | `_Measures` | Count where Delay Minutes > threshold |
| `Refund Exposure Above Threshold` | `_Measures` | Revenue where refund request & above threshold |
| `Refund Policy Saving` | `_Measures` | Revenue saved by NOT paying refunds below threshold |
| `Selected Price Adjustment` | `_Measures` | Current % slider value (default 0) |
| `Simulated Revenue` | `_Measures` | Total Revenue × (1 + adjustment/100) |
| `Revenue Delta` | `_Measures` | Simulated − Actual revenue |
| `Revenue Delta %` | `_Measures` | Delta as % of actual revenue |

---

## Quick Reference — Dynamic & What-If Measures

| Folder | Measure | Typical Visual |
|---|---|---|
| **Dynamic Selectors** | `Selected Metric` | Used internally by Dynamic KPI |
| **Dynamic Selectors** | `Dynamic KPI` | Chart Y-axis (driven by Metric Selector slicer) |
| **Dynamic Selectors** | `Dynamic KPI Label` | Card (formatted value for selected metric) |
| **Dynamic Selectors** | `Selected Time Dimension` | Used in bookmark logic |
| **What-If** | `Selected Delay Threshold` | Used internally; display on slider card |
| **What-If** | `Journeys Above Threshold` | Card in scenario panel |
| **What-If** | `Refund Exposure Above Threshold` | Card in scenario panel |
| **What-If** | `Refund Policy Saving` | Card in scenario panel |
| **What-If** | `Selected Price Adjustment` | Used internally; display on slider card |
| **What-If** | `Simulated Revenue` | Card + combo chart |
| **What-If** | `Revenue Delta` | Card with conditional font color |
| **What-If** | `Revenue Delta %` | Card (alternative to absolute delta) |

---

## SVG Image Assets

All SVG files are pre-built and located in `dashboards/train-rides-analytics/assets/`. Detailed placement instructions for each file are given in the relevant page section of this guide. The table below is your quick reference.

### File reference

| File | Dimensions | Where used | Guide section |
|---|---|---|---|
| `cover-banner.svg` | 1280×400 | Cover page — full-width hero image | §4.2 |
| `icon-train.svg` | 80×80 | Cover page accent, bridging banner and nav zone | §4.3 |
| `icon-overview.svg` | 64×64 | Executive Overview nav tile (Cover page) | §4.4 |
| `icon-passengers.svg` | 64×64 | Passenger Usage nav tile + Page 2 section header | §4.4, §6 |
| `icon-revenue.svg` | 64×64 | Sales & Revenue nav tile + Page 3 KPI card icon | §4.4, §7 |
| `icon-performance.svg` | 64×64 | Railway Performance nav tile + Page 4 section header | §4.4, §8 |
| `icon-route.svg` | 64×64 | Route Analysis nav tile + Page 5 section header | §4.4, §9 |
| `icon-simulator.svg` | 64×64 | Scenario Simulator nav tile (Cover page) | §4.4 |
| `icon-ontime.svg` | 64×64 | On-Time Rate % KPI card icon (Page 4) | §8 |
| `icon-delayed.svg` | 64×64 | Delay Rate % KPI card icon (Page 4) | §8 |
| `icon-cancelled.svg` | 64×64 | Cancel Rate % KPI card icon (Page 4) | §8 |

### How to insert any SVG in Power BI

1. **Insert** → **Image** → navigate to the file in `assets/`.
2. **Format visual** → **Image** → Image fit:
   - Use **Fill** for the cover banner (fills the defined W/H exactly).
   - Use **Fit** for icons (preserves the circle shape without cropping).
3. Type the exact position and size in **Format visual → General → Properties** (X, Y, W, H) for pixel-perfect placement.

### How to place an icon beside a KPI card

> Apply this pattern to any 64×64 icon on the analytical pages.

1. Note the **top-left X, Y** of the KPI card you want to decorate.
2. Insert the icon image. Set **W = 44, H = 44**.
3. Position it at: `card_X − 52`, `card_Y + (card_H − 44) / 2` (vertically centred, 8px gap to the left of the card).
4. Alternatively, place it **inside the top-right corner** of a wide card: `card_X + card_W − 52`, `card_Y + 8`.

> **Important:** SVG images in Power BI are **static** — they do not respond to slicers, cross-filtering, or conditional formatting. Use them purely for decoration and visual hierarchy.

