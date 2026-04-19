# Campaign Pulse Dashboard — Step-by-Step Report Build Tutorial

> **Audience:** Junior Power BI Developer
> **Prerequisite:** The semantic model is already built with all 37 measures in the `_Measures` table. Open the `.pbip` file in Power BI Desktop before starting.
> **Time estimate:** 2–4 hours for all 7 pages

---

## Table of Contents

1. [Before You Start — Understanding the Model](#1-before-you-start--understanding-the-model)
2. [Page 1: Executive Overview](#2-page-1-executive-overview)
3. [Page 2: Data Quality & Outliers](#3-page-2-data-quality--outliers)
4. [Page 3: Customer Profile](#4-page-3-customer-profile)
5. [Page 4: Product Performance](#5-page-4-product-performance)
6. [Page 5: Channel Performance](#6-page-5-channel-performance)
7. [Page 6: Campaign Effectiveness](#7-page-6-campaign-effectiveness)
8. [Page 7: Web Purchase Drivers](#8-page-7-web-purchase-drivers)
9. [Final Checklist](#9-final-checklist)

---

## 1. Before You Start — Understanding the Model

### 1.1 The Data

This report analyzes **2,240 customers** of Maven Marketing. The data covers:
- **Customer profiles** — age, income, education, marital status, children
- **Product spending** — how much each customer spent on 6 product categories (Wines, Meat, Fish, Fruits, Sweet, Gold)
- **Channel purchases** — how many purchases via 4 channels (Web, Store, Catalog, Deals)
- **Campaign responses** — whether each customer accepted 6 marketing campaigns

### 1.2 The Tables

| Table                   | Type      | Description                                                                                                                    |
| ----------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `Dim_Customer`          | Dimension | 2,240 rows. One row per customer. Contains demographics, data quality flags, derived columns like Age Group, Income Band, etc. |
| `Dim_Product`           | Dimension | 6 rows. Product categories: Fish, Fruits, Gold, Meat, Sweet, Wines                                                             |
| `Dim_Channel`           | Dimension | 4 rows. Purchase channels: Catalog, Deals, Store, Web                                                                          |
| `Dim_Campaign`          | Dimension | 6 rows. Campaign 1 through Campaign 6 (Campaign 6 = the "Response" / last campaign)                                            |
| `Dim_Date`              | Dimension | Date calendar table for time-based filtering                                                                                   |
| `Fact_ProductSpend`     | Fact      | ~13,440 rows. Each row = one customer × one product with `Amount` spent                                                        |
| `Fact_ChannelPurchases` | Fact      | ~8,960 rows. Each row = one customer × one channel with `Purchases` count                                                      |
| `Fact_CampaignResponse` | Fact      | ~13,440 rows. Each row = one customer × one campaign with `Accepted` (0 or 1)                                                  |
| `_Measures`             | Measures  | Hidden table hosting all 37 DAX measures organized in 6 display folders                                                        |

### 1.3 The 37 Measures (Already Created)

All measures live in the hidden `_Measures` table. In the Fields pane, expand `_Measures` to see 6 folders:

| Display Folder             | Measures                                                                                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core Metrics**           | Total Customers, Total Spend, Total Purchases, Total Acceptances, Total Campaign Offers                                                                 |
| **Data Quality**           | Null Income Count, Birth Year Outlier Count, Income Outlier Count, % Null Income, % Outliers, Clean Customer Count                                      |
| **Customer Profile**       | Avg Income, Median Income, Avg Age, Avg Recency, Avg Web Visits per Month, Complaint Rate, % With Children                                              |
| **Product Performance**    | Avg Spend per Customer, Product % of Total Spend, Product Rank, Spend per Product Category                                                              |
| **Channel Performance**    | Avg Purchases per Customer, Channel % of Total, Channel Rank, Total Web Purchases, Total Store Purchases, Total Catalog Purchases, Total Deal Purchases |
| **Campaign Effectiveness** | Acceptance Rate, Campaign % of Total Acceptances, Unique Accepted Customers, Campaign Rank, Avg Campaigns Accepted per Customer                         |
| **Web Purchase Drivers**   | Avg Web Purchases per Customer, Web Purchase Share, Web Visit to Purchase Ratio                                                                         |

### 1.4 Important Tips Before Building

1. **Save frequently** — Press `Ctrl + S` after completing each page
2. **Use the Fields pane** on the right side to find measures and columns
3. **Measures have a calculator icon** (🔢); columns have a table/column icon
4. **To find a measure quickly:** Click the search box at the top of the Fields pane and type the measure name
5. **Canvas size:** The default canvas is 1280 × 720 pixels. We will use this standard size
6. **Theme:** The model uses the Fluent2 theme. You don't need to change it

### 1.5 How to Navigate the Fields Pane

- **`_Measures`** → Expand this table to see all 6 folders → expand each folder to see measures
- **`Dim_Customer`** → Contains sliceable columns like `Age Group`, `Country`, `Education`, `Income Band`, `Relationship Status`, `Children Status`, `Campaign Engagement`
- **`Dim_Product`** → Contains `Product` column
- **`Dim_Channel`** → Contains `Channel` column
- **`Dim_Campaign`** → Contains `Campaign` column

> **Note:** Some columns are hidden (marked with an eye icon or not visible). You don't need hidden columns for this report. Only use the visible columns listed above.

---

## 2. Page 1: Executive Overview

**Purpose:** Give stakeholders an at-a-glance summary of all key metrics — customers, spend, purchases, and campaign performance.

### Step 2.1 — Rename the Page

1. At the bottom of the screen, you'll see a tab that says **"Page 1"**
2. **Double-click** on the tab name "Page 1"
3. Type `Executive Overview` and press **Enter**

### Step 2.2 — Add a Title Text Box

1. Go to the **Insert** tab on the top ribbon
2. Click **Text box**
3. A text box appears on the canvas. Drag it to the **top-left corner** of the canvas
4. Resize it to span the **full width** of the canvas (approximately 1280 px wide × 60 px tall)
5. Type: `Campaign Pulse Dashboard`
6. Select all the text, then in the formatting toolbar above the text box:
   - Set **Font size** to `28`
   - Set **Font weight** to **Bold**
   - Set **Font color** to a dark color (black or dark gray)
7. Press **Enter** to go to a new line within the text box
8. Type: `Maven Marketing · 2,240 Customers`
9. Select this subtitle text:
   - Set **Font size** to `14`
   - Set **Font weight** to **Regular**
   - Set **Font color** to gray
10. Click outside the text box to deselect it
11. Position it at the very top: **X = 0, Y = 0** (you can set exact positions in Format > Properties > Position)

### Step 2.3 — Add 4 KPI Cards (Top Row)

We will add 4 card visuals in a row across the top, just below the title.

#### Card 1: Total Customers

1. Click on an **empty area** of the canvas (so nothing is selected)
2. In the **Visualizations** pane (right side), click the **Card** visual icon (it looks like a single number in a box — labeled "Card" on hover). Use the **new Card visual** if available (it has rounded corners)
3. A blank card appears on the canvas
4. In the **Fields** pane, expand **`_Measures`** → expand **`Core Metrics`**
5. **Drag** `Total Customers` into the **Fields** well of the card (or just click the checkbox next to it)
6. The card now shows the total customer count
7. **Resize** the card to approximately **280 px wide × 100 px tall**
8. **Position** it at approximately **X = 30, Y = 70** (just below the title, left side)
9. **Format the card:**
   - Click the card to select it
   - In the Visualizations pane, click the **Format** tab (paint roller icon)
   - Under **Callout value**: Set font size to `28`, color to dark
   - Under **Category label**: Make sure it shows "Total Customers". Set font size to `12`

#### Card 2: Total Spend

1. **Click an empty area** of the canvas
2. Add another **Card** visual
3. Drag `Total Spend` (from `_Measures` > `Core Metrics`) into the Fields well
4. Resize to **280 × 100 px**
5. Position at approximately **X = 330, Y = 70** (next to Card 1)
6. The value should display with a `$` sign and comma formatting (e.g., `$1,240,896`) — this comes from the measure's format string

#### Card 3: Total Purchases

1. Add another **Card** visual
2. Drag `Total Purchases` into Fields
3. Resize to **280 × 100 px**
4. Position at approximately **X = 630, Y = 70**

#### Card 4: Acceptance Rate

1. Add another **Card** visual
2. Drag `Acceptance Rate` (from `_Measures` > `Campaign Effectiveness`) into Fields
3. Resize to **280 × 100 px**
4. Position at approximately **X = 930, Y = 70**
5. The value will display as a percentage (e.g., `7.5%`)

> **Alignment tip:** Select all 4 cards by holding **Ctrl** and clicking each one. Then go to **Format** tab on the ribbon → **Align** → **Align Top** to make them perfectly level. Then use **Distribute horizontally** to space them evenly.

### Step 2.4 — Product Spend Bar Chart (Middle Left)

1. Click an **empty area** of the canvas
2. In the Visualizations pane, click **Clustered bar chart** (horizontal bars icon)
3. A blank chart appears. Resize it to approximately **380 × 250 px**
4. Position at approximately **X = 30, Y = 190**
5. **Add data:**
   - Drag `Dim_Product` → `Product` to the **Y-axis** well
   - Drag `_Measures` → `Core Metrics` → `Total Spend` to the **X-axis** well
6. **Sort descending:**
   - Click the **three dots (⋯)** in the top-right corner of the visual
   - Click **Sort axis** → select **Total Spend**
   - Click the three dots again → **Sort axis** → select **Sort descending**
7. **Add data labels:**
   - In the Format pane (paint roller), expand **Data labels** → toggle **On**
8. **Add a title:**
   - In Format pane → **Title** → toggle **On**
   - Set title text to `Spend by Product`
   - Set font size to `14`, **Bold**

### Step 2.5 — Channel Purchases Bar Chart (Middle Center)

1. Click an **empty area** of the canvas
2. Add a **Clustered bar chart**
3. Resize to **380 × 250 px**
4. Position at approximately **X = 440, Y = 190**
5. **Add data:**
   - Drag `Dim_Channel` → `Channel` to the **Y-axis**
   - Drag `Total Purchases` to the **X-axis**
6. Sort descending by `Total Purchases` (same process as Step 2.4 — three dots → Sort axis)
7. Turn on **Data labels**
8. Set title to `Purchases by Channel`

### Step 2.6 — Campaign Acceptance Rate Bar Chart (Middle Right)

1. Click an **empty area** of the canvas
2. Add a **Clustered bar chart**
3. Resize to **380 × 250 px**
4. Position at approximately **X = 850, Y = 190**
5. **Add data:**
   - Drag `Dim_Campaign` → `Campaign` to the **Y-axis**
   - Drag `Acceptance Rate` to the **X-axis**
6. Sort descending by `Acceptance Rate`
7. Turn on **Data labels**
8. Set title to `Acceptance Rate by Campaign`

### Step 2.7 — Country Distribution Donut (Bottom Left)

1. Click an **empty area** of the canvas
2. In the Visualizations pane, click the **Donut chart** icon (circle with a hole)
3. Resize to **350 × 250 px**
4. Position at approximately **X = 100, Y = 460**
5. **Add data:**
   - Drag `Dim_Customer` → `Country` to the **Legend** well
   - Drag `Total Customers` to the **Values** well
6. **Format:**
   - In Format pane → **Detail labels** → toggle **On**
   - Set label style to **Category, percent of total** (or **All detail labels**)
7. Set title to `Customers by Country`

### Step 2.8 — Age Group Distribution Donut (Bottom Right)

1. Click an **empty area** of the canvas
2. Add a **Donut chart**
3. Resize to **350 × 250 px**
4. Position at approximately **X = 750, Y = 460**
5. **Add data:**
   - Drag `Dim_Customer` → `Age Group` to **Legend**
   - Drag `Total Customers` to **Values**
6. Turn on **Detail labels** with category + percent
7. Set title to `Customers by Age Group`

### Step 2.9 — Review and Save

1. Look at the page — you should see:
   - Title banner at top
   - 4 KPI cards in a row
   - 3 bar charts in the middle
   - 2 donut charts at the bottom
2. Press **Ctrl + S** to save

---

## 3. Page 2: Data Quality & Outliers

**Purpose:** Answer: *"Are there any null values or outliers? How will you handle them?"*

### Step 3.1 — Create a New Page

1. At the bottom of the screen, click the **+ (plus)** icon next to the "Executive Overview" tab
2. A new page appears called "Page 2"
3. **Double-click** the tab and rename it to `Data Quality`

### Step 3.2 — Add Page Title

1. **Insert** → **Text box**
2. Type: `Data Quality & Outlier Analysis`
3. Format: Font size `24`, **Bold**, dark color
4. Position at top of canvas: **X = 0, Y = 0**, full width, ~50 px tall

### Step 3.3 — Add 6 Data Quality KPI Cards

Create 6 cards in two rows of 3. Each card is approximately **200 × 90 px**.

**Row 1 (counts):**

| Position | Measure                    | Approximate X, Y |
| -------- | -------------------------- | ---------------- |
| Left     | `Null Income Count`        | X = 30, Y = 60   |
| Center   | `Birth Year Outlier Count` | X = 260, Y = 60  |
| Right    | `Income Outlier Count`     | X = 490, Y = 60  |

**Row 2 (percentages + clean count):**

| Position | Measure                | Approximate X, Y |
| -------- | ---------------------- | ---------------- |
| Left     | `% Null Income`        | X = 30, Y = 160  |
| Center   | `% Outliers`           | X = 260, Y = 160 |
| Right    | `Clean Customer Count` | X = 490, Y = 160 |

**For each card:**
1. Click empty canvas area
2. Add **Card** visual
3. Drag the measure from `_Measures` > `Data Quality` into the Fields well
4. Resize and position as shown above
5. Format: Callout value font size `24`

> **Pro tip:** Create the first card, format it, then **Ctrl+C** and **Ctrl+V** to duplicate it. Then just swap the measure in the Fields well. This saves time and keeps formatting consistent.

### Step 3.4 — Income Distribution Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **450 × 220 px**
3. Position at approximately **X = 30, Y = 270**
4. **Add data:**
   - Y-axis: `Dim_Customer` → `Income Band`
   - X-axis: `Total Customers`
5. **Sort:** Sort by X-axis (Total Customers) descending
6. Turn on **Data labels**
7. Title: `Income Distribution`

### Step 3.5 — Age Distribution Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **450 × 220 px**
3. Position at approximately **X = 510, Y = 270**
4. **Add data:**
   - Y-axis: `Dim_Customer` → `Age Group`
   - X-axis: `Total Customers`
5. Sort descending by `Total Customers`
6. Turn on **Data labels**
7. Title: `Age Distribution`

### Step 3.6 — Flagged Records Table

This table shows the actual records that are flagged as outliers or null.

1. Add a **Table** visual (grid icon in Visualizations pane)
2. Resize to **930 × 180 px**
3. Position at approximately **X = 30, Y = 500**
4. **Add columns** (drag each from the Fields pane):
   - `Dim_Customer` → `Customer_ID` (you may need to show hidden fields: click the three dots at top of Fields pane → **Show hidden** to reveal it, or use the View menu)
   - `Dim_Customer` → `Year_Birth`
   - `Dim_Customer` → `Age Group`
   - `Dim_Customer` → `Income Band`
   - `Dim_Customer` → `Flag_BirthYear_Outlier`
   - `Dim_Customer` → `Flag_Income_Outlier`
   - `Dim_Customer` → `Flag_IncomeNull`
5. **Add a visual-level filter** to show only flagged records:
   - Click the table visual to select it
   - In the **Filters** pane (left of Visualizations pane), find **Filters on this visual**
   - Drag `Flag_BirthYear_Outlier` to the visual filter area
   - Check **True** only
   - **OR** — since we want ANY flag to be true, an alternative approach:
     - Add `Flag_BirthYear_Outlier` as a filter → set to True
     - Then change the filter type: Click the dropdown at the top of the filter card → change from **Basic filtering** to **Advanced filtering** → set "Show items when the value: **is True**"
   - Actually, the simplest approach: Add all three flag columns to the table. Users can visually scan for TRUE values. Alternatively, sort the table by any flag column descending to push TRUE values to the top.

> **Simpler alternative:** Don't filter. Instead, sort the table by `Flag_BirthYear_Outlier` descending so the TRUE values appear first. The table will naturally show all 2,240 customers but the outliers will be at the top.

### Step 3.7 — Explanation Text Box

1. **Insert** → **Text box**
2. Position at approximately **X = 510, Y = 500**, size **450 × 180 px**
3. Type the following (each on its own line):

```
Data Quality Summary:

• 24 customers have null (missing) income values — excluded from income-based analysis
• 3 customers have birth year before 1924 (age > 100) — likely data entry errors, grouped into "60+" age group
• Income outliers above $162,000 are flagged but retained in the dataset
• Clean customer count excludes all flagged records for conservative analysis
• Handling approach: Flags are transparent — users can filter to include/exclude as needed
```

4. Format: Font size `11`, regular weight

### Step 3.8 — Save

Press **Ctrl + S**

---

## 4. Page 3: Customer Profile

**Purpose:** Answer: *"What does the average customer look like for this company?"*

### Step 4.1 — Create New Page

1. Click **+** to add a new page
2. Rename to `Customer Profile`

### Step 4.2 — Add Page Title

1. Insert → Text box
2. Type: `The Average Maven Marketing Customer`
3. Font size `24`, Bold
4. Position at top, full width, ~50 px tall

### Step 4.3 — Add 5 KPI Cards (Top Row)

Create 5 cards in a row, each approximately **220 × 90 px**.

| #   | Measure           | Source Folder    | Approx X, Y     |
| --- | ----------------- | ---------------- | --------------- |
| 1   | `Avg Age`         | Customer Profile | X = 30, Y = 60  |
| 2   | `Avg Income`      | Customer Profile | X = 270, Y = 60 |
| 3   | `Median Income`   | Customer Profile | X = 510, Y = 60 |
| 4   | `Avg Recency`     | Customer Profile | X = 750, Y = 60 |
| 5   | `% With Children` | Customer Profile | X = 990, Y = 60 |

For each: Add Card → drag measure → resize → position.

### Step 4.4 — Education Donut Chart

1. Add a **Donut chart**
2. Resize to **300 × 220 px**
3. Position: **X = 30, Y = 170**
4. Data:
   - Legend: `Dim_Customer` → `Education`
   - Values: `Total Customers`
5. Detail labels: On, show Category + Percent
6. Title: `Education Level`

### Step 4.5 — Relationship Status Donut Chart

1. Add a **Donut chart**
2. Resize to **300 × 220 px**
3. Position: **X = 350, Y = 170**
4. Data:
   - Legend: `Dim_Customer` → `Relationship Status`
   - Values: `Total Customers`
5. Detail labels: On, show Category + Percent
6. Title: `Relationship Status`

### Step 4.6 — Children Status Donut Chart

1. Add a **Donut chart**
2. Resize to **300 × 220 px**
3. Position: **X = 670, Y = 170**
4. Data:
   - Legend: `Dim_Customer` → `Children Status`
   - Values: `Total Customers`
5. Detail labels: On
6. Title: `Children Status`

### Step 4.7 — Campaign Engagement Donut Chart

1. Add a **Donut chart**
2. Resize to **300 × 220 px**
3. Position: **X = 970, Y = 170**
4. Data:
   - Legend: `Dim_Customer` → `Campaign Engagement`
   - Values: `Total Customers`
5. Detail labels: On
6. Title: `Campaign Engagement`

### Step 4.8 — Age Group Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **380 × 220 px**
3. Position: **X = 30, Y = 410**
4. Data:
   - Y-axis: `Dim_Customer` → `Age Group`
   - X-axis: `Total Customers`
5. Sort descending by `Total Customers`
6. Data labels: On
7. Title: `Customers by Age Group`

### Step 4.9 — Income Band Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **380 × 220 px**
3. Position: **X = 440, Y = 410**
4. Data:
   - Y-axis: `Dim_Customer` → `Income Band`
   - X-axis: `Total Customers`
5. Sort descending by `Total Customers`
6. Data labels: On
7. Title: `Customers by Income Band`

### Step 4.10 — Country Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **380 × 220 px**
3. Position: **X = 850, Y = 410**
4. Data:
   - Y-axis: `Dim_Customer` → `Country`
   - X-axis: `Total Customers`
5. Sort descending by `Total Customers`
6. Data labels: On
7. Title: `Customers by Country`

### Step 4.11 — Save

Press **Ctrl + S**

---

## 5. Page 4: Product Performance

**Purpose:** Answer: *"Which products are performing best?"*

### Step 5.1 — Create New Page

1. Click **+** → rename to `Product Performance`

### Step 5.2 — Add Page Title

1. Insert → Text box → `Product Performance Analysis`
2. Font size `24`, Bold, top of canvas

### Step 5.3 — Add 2 KPI Cards

| #   | Measure                  | Approx X, Y     | Size     |
| --- | ------------------------ | --------------- | -------- |
| 1   | `Total Spend`            | X = 30, Y = 60  | 280 × 90 |
| 2   | `Avg Spend per Customer` | X = 330, Y = 60 | 280 × 90 |

### Step 5.4 — Product Spend Horizontal Bar Chart (Main Visual)

This is the hero visual showing which products generate the most revenue.

1. Add a **Clustered bar chart** (horizontal bars)
2. Resize to **580 × 280 px**
3. Position: **X = 30, Y = 170**
4. Data:
   - Y-axis: `Dim_Product` → `Product`
   - X-axis: `Total Spend`
5. Sort descending by `Total Spend`
6. **Data labels:**
   - Format pane → Data labels → On
   - Display units: None (to show actual values)
7. **Add percentage as a tooltip:**
   - Drag `Product % of Total Spend` to the **Tooltips** well
   - Now when users hover over a bar, they see both the amount and the percentage
8. Title: `Total Spend by Product`

### Step 5.5 — Treemap

The treemap gives a visual sense of proportion — bigger rectangles = more spend.

1. Add a **Treemap** visual (the icon looks like nested rectangles)
2. Resize to **380 × 280 px**
3. Position: **X = 640, Y = 170**
4. Data:
   - Category: `Dim_Product` → `Product`
   - Values: `Total Spend`
5. **Format:**
   - Data labels → On
   - Show category + value
6. Title: `Spend Proportion by Product`

### Step 5.6 — Matrix: Product Spend by Age Group

This matrix shows how spending varies across age groups for each product.

1. Add a **Matrix** visual (the icon looks like a grid with a header row and column)
2. Resize to **580 × 200 px**
3. Position: **X = 30, Y = 470**
4. Data:
   - Rows: `Dim_Product` → `Product`
   - Columns: `Dim_Customer` → `Age Group`
   - Values: `Total Spend`
5. **Add conditional formatting (color scale):**
   - Click the matrix to select it
   - In the Format pane → **Cell elements** → **Background color** → toggle **On**
   - Click **fx** (advanced controls)
   - Set:
     - Format style: **Gradient**
     - Based on: **Total Spend**
     - Minimum color: light yellow or white
     - Maximum color: dark blue or dark green
   - Click **OK**
6. **Format:**
   - Column headers font size: `10`
   - Values font size: `10`
   - Turn off **Stepped layout** if it's on (Format → Row headers → Stepped layout → Off)
7. Title: `Product Spend by Age Group`

### Step 5.7 — Matrix: Product Spend by Income Band

1. Add a **Matrix** visual
2. Resize to **580 × 200 px**
3. Position: **X = 640, Y = 470**
4. Data:
   - Rows: `Dim_Product` → `Product`
   - Columns: `Dim_Customer` → `Income Band`
   - Values: `Total Spend`
5. Add conditional formatting (same steps as 5.6 — background color gradient)
6. Title: `Product Spend by Income Band`

### Step 5.8 — Save

Press **Ctrl + S**

---

## 6. Page 5: Channel Performance

**Purpose:** Answer: *"Which channels are underperforming?"*

### Step 6.1 — Create New Page

1. Click **+** → rename to `Channel Performance`

### Step 6.2 — Add Page Title

1. Insert → Text box → `Channel Performance Analysis`
2. Font size `24`, Bold

### Step 6.3 — Add 2 KPI Cards

| #   | Measure                      | Approx X, Y     | Size     |
| --- | ---------------------------- | --------------- | -------- |
| 1   | `Total Purchases`            | X = 30, Y = 60  | 280 × 90 |
| 2   | `Avg Purchases per Customer` | X = 330, Y = 60 | 280 × 90 |

### Step 6.4 — Channel Purchases Horizontal Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **450 × 260 px**
3. Position: **X = 30, Y = 170**
4. Data:
   - Y-axis: `Dim_Channel` → `Channel`
   - X-axis: `Total Purchases`
5. Sort descending by `Total Purchases`
6. Data labels: On
7. Drag `Channel % of Total` to **Tooltips**
8. Title: `Total Purchases by Channel`

### Step 6.5 — Channel Donut Chart

1. Add a **Donut chart**
2. Resize to **320 × 260 px**
3. Position: **X = 510, Y = 170**
4. Data:
   - Legend: `Dim_Channel` → `Channel`
   - Values: `Total Purchases`
5. Detail labels: On (Category + Percent)
6. Title: `Purchase Share by Channel`

### Step 6.6 — Avg Purchases per Customer by Channel

This shows which channels have the highest per-customer intensity.

1. Add a **Clustered bar chart**
2. Resize to **350 × 260 px**
3. Position: **X = 860, Y = 170**
4. Data:
   - Y-axis: `Dim_Channel` → `Channel`
   - X-axis: `Avg Purchases per Customer`
5. Sort descending by `Avg Purchases per Customer`
6. Data labels: On
7. Title: `Avg Purchases per Customer`

### Step 6.7 — Matrix: Channel by Income Band

1. Add a **Matrix** visual
2. Resize to **580 × 210 px**
3. Position: **X = 30, Y = 450**
4. Data:
   - Rows: `Dim_Channel` → `Channel`
   - Columns: `Dim_Customer` → `Income Band`
   - Values: `Total Purchases`
5. Add conditional formatting (background color gradient based on Total Purchases)
6. Title: `Channel Purchases by Income Band`

### Step 6.8 — Matrix: Channel by Age Group

1. Add a **Matrix** visual
2. Resize to **580 × 210 px**
3. Position: **X = 640, Y = 450**
4. Data:
   - Rows: `Dim_Channel` → `Channel`
   - Columns: `Dim_Customer` → `Age Group`
   - Values: `Total Purchases`
5. Add conditional formatting (background color gradient)
6. Title: `Channel Purchases by Age Group`

### Step 6.9 — Save

Press **Ctrl + S**

---

## 7. Page 6: Campaign Effectiveness

**Purpose:** Answer: *"Which marketing campaign is most successful?"*

### Step 7.1 — Create New Page

1. Click **+** → rename to `Campaign Effectiveness`

### Step 7.2 — Add Page Title

1. Insert → Text box → `Campaign Effectiveness`
2. Font size `24`, Bold

### Step 7.3 — Add 3 KPI Cards

| #   | Measure                               | Approx X, Y     | Size     |
| --- | ------------------------------------- | --------------- | -------- |
| 1   | `Acceptance Rate`                     | X = 30, Y = 60  | 280 × 90 |
| 2   | `Unique Accepted Customers`           | X = 330, Y = 60 | 280 × 90 |
| 3   | `Avg Campaigns Accepted per Customer` | X = 630, Y = 60 | 300 × 90 |

### Step 7.4 — Acceptance Rate by Campaign Bar Chart

This is the key visual — shows which campaign had the highest acceptance rate.

1. Add a **Clustered bar chart**
2. Resize to **550 × 260 px**
3. Position: **X = 30, Y = 170**
4. Data:
   - Y-axis: `Dim_Campaign` → `Campaign`
   - X-axis: `Acceptance Rate`
5. Sort descending by `Acceptance Rate`
6. **Data labels:** On (these will display as percentages like 14.9%, 7.3%, etc.)
7. Title: `Acceptance Rate by Campaign`

> **Key Insight Preview:** Campaign 5 and Campaign 6 (Response) tend to have the highest acceptance rates. Campaign 2 typically has the lowest.

### Step 7.5 — Total Acceptances by Campaign Bar Chart

1. Add a **Clustered bar chart**
2. Resize to **550 × 260 px**
3. Position: **X = 610, Y = 170**
4. Data:
   - Y-axis: `Dim_Campaign` → `Campaign`
   - X-axis: `Total Acceptances`
5. Sort descending by `Total Acceptances`
6. Data labels: On
7. Title: `Total Acceptances by Campaign`

### Step 7.6 — Campaign Donut

1. Add a **Donut chart**
2. Resize to **350 × 230 px**
3. Position: **X = 30, Y = 450**
4. Data:
   - Legend: `Dim_Campaign` → `Campaign`
   - Values: `Total Acceptances`
5. Detail labels: Category + Percent
6. Title: `Share of Total Acceptances`

### Step 7.7 — Matrix: Acceptance Rate by Education

1. Add a **Matrix** visual
2. Resize to **380 × 230 px**
3. Position: **X = 410, Y = 450**
4. Data:
   - Rows: `Dim_Campaign` → `Campaign`
   - Columns: `Dim_Customer` → `Education`
   - Values: `Acceptance Rate`
5. Add conditional formatting (background color gradient)
6. Title: `Acceptance Rate by Education`

### Step 7.8 — Matrix: Acceptance Rate by Income Band

1. Add a **Matrix** visual
2. Resize to **380 × 230 px**
3. Position: **X = 820, Y = 450**
4. Data:
   - Rows: `Dim_Campaign` → `Campaign`
   - Columns: `Dim_Customer` → `Income Band`
   - Values: `Acceptance Rate`
5. Add conditional formatting (background color gradient)
6. Title: `Acceptance Rate by Income Band`

### Step 7.9 — Save

Press **Ctrl + S**

---

## 8. Page 7: Web Purchase Drivers

**Purpose:** Answer: *"What factors are significantly related to the number of web purchases?"*

### Step 8.1 — Create New Page

1. Click **+** → rename to `Web Purchase Drivers`

### Step 8.2 — Add Page Title

1. Insert → Text box → `Web Purchase Driver Analysis`
2. Font size `24`, Bold

### Step 8.3 — Add 3 KPI Cards

| #   | Measure                          | Approx X, Y     | Size     |
| --- | -------------------------------- | --------------- | -------- |
| 1   | `Total Web Purchases`            | X = 30, Y = 60  | 280 × 90 |
| 2   | `Avg Web Purchases per Customer` | X = 330, Y = 60 | 280 × 90 |
| 3   | `Web Purchase Share`             | X = 630, Y = 60 | 280 × 90 |

### Step 8.4 — Web Purchases by Income Band

1. Add a **Clustered bar chart**
2. Resize to **380 × 230 px**
3. Position: **X = 30, Y = 170**
4. Data:
   - Y-axis: `Dim_Customer` → `Income Band`
   - X-axis: `Avg Web Purchases per Customer`
5. Sort descending by `Avg Web Purchases per Customer`
6. Data labels: On
7. Title: `Web Purchases by Income Band`

> **What to look for:** Higher income customers likely make more web purchases. This shows the income-to-web-purchase correlation.

### Step 8.5 — Web Purchases by Age Group

1. Add a **Clustered bar chart**
2. Resize to **380 × 230 px**
3. Position: **X = 440, Y = 170**
4. Data:
   - Y-axis: `Dim_Customer` → `Age Group`
   - X-axis: `Avg Web Purchases per Customer`
5. Sort descending by `Avg Web Purchases per Customer`
6. Data labels: On
7. Title: `Web Purchases by Age Group`

### Step 8.6 — Web Purchases by Education

1. Add a **Clustered bar chart**
2. Resize to **380 × 230 px**
3. Position: **X = 850, Y = 170**
4. Data:
   - Y-axis: `Dim_Customer` → `Education`
   - X-axis: `Avg Web Purchases per Customer`
5. Sort descending by `Avg Web Purchases per Customer`
6. Data labels: On
7. Title: `Web Purchases by Education`

### Step 8.7 — Web Purchases by Relationship Status

1. Add a **Clustered bar chart**
2. Resize to **280 × 220 px**
3. Position: **X = 30, Y = 420**
4. Data:
   - Y-axis: `Dim_Customer` → `Relationship Status`
   - X-axis: `Avg Web Purchases per Customer`
5. Sort descending
6. Data labels: On
7. Title: `By Relationship Status`

### Step 8.8 — Web Purchases by Children Status

1. Add a **Clustered bar chart**
2. Resize to **280 × 220 px**
3. Position: **X = 340, Y = 420**
4. Data:
   - Y-axis: `Dim_Customer` → `Children Status`
   - X-axis: `Avg Web Purchases per Customer`
5. Data labels: On
6. Title: `By Children Status`

### Step 8.9 — Scatter Chart: Web Visits vs. Web Purchases

This scatter shows the relationship between website visits and actual purchases by country.

1. Add a **Scatter chart** visual (the icon looks like dots scattered on a grid)
2. Resize to **380 × 220 px**
3. Position: **X = 650, Y = 420**
4. Data:
   - **X Axis:** `Avg Web Visits per Month`
   - **Y Axis:** `Avg Web Purchases per Customer`
   - **Details (or Legend):** `Dim_Customer` → `Country`
5. **Format:**
   - Format pane → **Markers** → set shape to circle, size `12`
   - Data labels → On (show country name next to each dot)
6. Title: `Web Visits vs. Web Purchases by Country`

> **What to look for:** Countries in the upper-left (high purchases, low visits) are highly efficient. Countries in the lower-right (high visits, low purchases) have poor conversion.

### Step 8.10 — Heat Map Matrix: Income Band × Age Group

1. Add a **Matrix** visual
2. Resize to **580 × 220 px**
3. Position: **X = 30, Y = 660** — **wait**, this would go off-canvas. Let's reposition.

**Alternative positioning:** Place this matrix below the scatter chart, or make the page scrollable. Actually, let's make the layout tighter:

Reposition the matrix at **X = 650, Y = 420** — but the scatter is there. Let's reorganize:

**Better layout for this page:**
- Move the scatter chart to **X = 340, Y = 420** and make it 580 × 220
- Move "By Relationship Status" to **X = 30, Y = 420**, size 280 × 110
- Move "By Children Status" to **X = 30, Y = 540**, size 280 × 110
- Place the matrix at **X = 340, Y = 650** — this goes off canvas

**Simplest approach:** Make the scatter chart smaller and add the matrix as a replacement for one of the bar charts, OR skip the matrix if canvas space is tight.

**Recommended final layout:**

| Visual                  | Position     | Size          |
| ----------------------- | ------------ | ------------- |
| Cards (3)               | Top row      | 280 × 90 each |
| Income Band bar         | X=30, Y=170  | 290 × 200     |
| Age Group bar           | X=340, Y=170 | 290 × 200     |
| Education bar           | X=650, Y=170 | 290 × 200     |
| Children Status bar     | X=960, Y=170 | 290 × 200     |
| Relationship Status bar | X=30, Y=390  | 290 × 170     |
| Scatter chart           | X=340, Y=390 | 450 × 310     |
| Matrix (heat map)       | X=810, Y=390 | 440 × 310     |

For the matrix:
1. Add a **Matrix** visual
2. Resize to **440 × 310 px**
3. Position: **X = 810, Y = 390**
4. Data:
   - Rows: `Dim_Customer` → `Income Band`
   - Columns: `Dim_Customer` → `Age Group`
   - Values: `Avg Web Purchases per Customer`
5. **Conditional formatting:**
   - Format pane → Cell elements → Background color → On → fx
   - Gradient based on `Avg Web Purchases per Customer`
   - Min: light (white/light yellow)
   - Max: dark (dark blue/dark red)
6. Title: `Web Purchases: Income × Age Heat Map`

> **What to look for:** The darkest cells reveal which demographic combinations drive the most web purchases (likely high-income, middle-aged customers).

### Step 8.11 — Save

Press **Ctrl + S**

---

## 9. Final Checklist

Before considering the report complete, go through this checklist:

### 9.1 — Page Check

- [ ] **7 pages** exist with correct names:
  1. Executive Overview
  2. Data Quality
  3. Customer Profile
  4. Product Performance
  5. Channel Performance
  6. Campaign Effectiveness
  7. Web Purchase Drivers

### 9.2 — Visual Check (Per Page)

Go to each page and verify:

- [ ] All visuals display data (no blank visuals or error icons)
- [ ] Card values look reasonable:
  - Total Customers ≈ 2,240
  - Total Spend ≈ $1.2M–$1.4M
  - Total Purchases ≈ 30,000–35,000
  - Acceptance Rate ≈ 7–8%
- [ ] Bar charts are sorted descending (largest bar on top)
- [ ] Data labels are visible on all bar charts
- [ ] Donut charts show category names + percentages
- [ ] Matrices show data in cells (not blank)
- [ ] Conditional formatting is visible on matrices (color gradient)

### 9.3 — Formatting Check

- [ ] All visuals have titles
- [ ] Font sizes are consistent across similar visuals
- [ ] Visuals are aligned (not overlapping or misaligned)
- [ ] Cards use consistent sizing
- [ ] No visuals extend beyond the canvas edges

### 9.4 — Interaction Check

1. On the **Executive Overview** page:
   - Click on a bar in the Product chart (e.g., "Wines")
   - Verify that the other visuals cross-filter (e.g., Channel chart should highlight wine-related purchases)
   - Click the bar again to deselect
2. On the **Customer Profile** page:
   - Click on a donut slice (e.g., "Graduation" in Education)
   - Verify bar charts filter to show only Graduation customers
3. Repeat quick interaction tests on each page

### 9.5 — Key Insights to Verify

As you review the report, these are the expected findings:

| Question                      | Expected Insight                                                                                                |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Nulls/Outliers?**           | ~24 null incomes, ~3 birth year outliers (born before 1924)                                                     |
| **Best products?**            | Wines and Meat dominate spending (~80%+ combined)                                                               |
| **Underperforming channels?** | Deals (discount purchases) is typically the weakest channel                                                     |
| **Most successful campaign?** | Campaign 5 or Campaign 6 typically have the highest acceptance rate; Campaign 2 is usually the lowest           |
| **Average customer?**         | ~50–55 years old, ~$50K–$55K income, Graduation education, mostly from Spain                                    |
| **Web purchase drivers?**     | Higher income → more web purchases; education level matters; customers without children tend to buy more online |

### 9.6 — Final Save

1. Press **Ctrl + S** one final time
2. Verify the `.pbip` project files are saved (check the file explorer in VS Code — the report definition files should show as modified)

---

## Appendix A: Common Issues & Fixes

| Problem                                 | Solution                                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| "Can't find the measure in Fields pane" | Make sure you're expanding the `_Measures` table → then the correct display folder               |
| "Card shows BLANK"                      | The measure may need filter context. Check that the measure references the correct table         |
| "Bar chart is unsorted"                 | Click ⋯ on the visual → Sort axis → choose the correct measure → Sort descending                 |
| "Matrix shows no data"                  | Verify both Rows AND Columns AND Values wells have fields. All three are required                |
| "Conditional formatting not showing"    | Make sure Background color is toggled On under Cell elements. Click fx to configure the gradient |
| "Visual is too small to read"           | Resize by dragging the corner handles. 200 px minimum for most visuals                           |
| "Hidden columns not visible"            | In Fields pane, right-click the table → Show hidden fields (or use View menu)                    |
| "Measure shows error icon (⚠)"          | The measure has a DAX error. Right-click → Edit measure → check the formula                      |

## Appendix B: Keyboard Shortcuts

| Shortcut           | Action                                            |
| ------------------ | ------------------------------------------------- |
| `Ctrl + S`         | Save                                              |
| `Ctrl + C / V`     | Copy/paste a visual (great for duplicating cards) |
| `Ctrl + Click`     | Select multiple visuals                           |
| `Tab`              | Cycle through visuals on a page                   |
| `Ctrl + Z`         | Undo                                              |
| `Ctrl + Y`         | Redo                                              |
| `Delete`           | Remove selected visual                            |
| `Ctrl + Shift + F` | Toggle focus mode for selected visual             |

## Appendix C: Measure Reference Quick Sheet

### Core Metrics
| Measure               | Formula                                 | Format |
| --------------------- | --------------------------------------- | ------ |
| Total Customers       | `COUNTROWS(Dim_Customer)`               | #,##0  |
| Total Spend           | `SUM(Fact_ProductSpend[Amount])`        | $#,##0 |
| Total Purchases       | `SUM(Fact_ChannelPurchases[Purchases])` | #,##0  |
| Total Acceptances     | `SUM(Fact_CampaignResponse[Accepted])`  | #,##0  |
| Total Campaign Offers | `COUNTROWS(Fact_CampaignResponse)`      | #,##0  |

### Data Quality
| Measure                  | Formula                                                                          | Format |
| ------------------------ | -------------------------------------------------------------------------------- | ------ |
| Null Income Count        | `CALCULATE([Total Customers], Dim_Customer[Flag_IncomeNull] = TRUE)`             | #,##0  |
| Birth Year Outlier Count | `CALCULATE([Total Customers], Dim_Customer[Flag_BirthYear_Outlier] = TRUE)`      | #,##0  |
| Income Outlier Count     | `CALCULATE([Total Customers], Dim_Customer[Flag_Income_Outlier] = TRUE)`         | #,##0  |
| % Null Income            | `DIVIDE([Null Income Count], [Total Customers])`                                 | 0.0%   |
| % Outliers               | `DIVIDE([Birth Year Outlier Count] + [Income Outlier Count], [Total Customers])` | 0.0%   |
| Clean Customer Count     | `CALCULATE([Total Customers], ...)` (all 3 flags = FALSE)                        | #,##0  |

### Customer Profile
| Measure                  | Formula                                                                         | Format  |
| ------------------------ | ------------------------------------------------------------------------------- | ------- |
| Avg Income               | `AVERAGE(Dim_Customer[ Income ])`                                               | $#,##0  |
| Median Income            | `MEDIAN(Dim_Customer[ Income ])`                                                | $#,##0  |
| Avg Age                  | `AVERAGEX(Dim_Customer, 2024 - Dim_Customer[Year_Birth])`                       | #,##0.0 |
| Avg Recency              | `AVERAGE(Dim_Customer[Recency])`                                                | #,##0.0 |
| Avg Web Visits per Month | `AVERAGE(Dim_Customer[NumWebVisitsMonth])`                                      | #,##0.0 |
| Complaint Rate           | `DIVIDE(SUM(Dim_Customer[Complain]), [Total Customers])`                        | 0.0%    |
| % With Children          | `DIVIDE(CALCULATE([Total Customers], ... = "Has Children"), [Total Customers])` | 0.0%    |

### Product Performance
| Measure                  | Formula                                                                       | Format |
| ------------------------ | ----------------------------------------------------------------------------- | ------ |
| Avg Spend per Customer   | `DIVIDE([Total Spend], [Total Customers])`                                    | $#,##0 |
| Product % of Total Spend | `DIVIDE([Total Spend], CALCULATE([Total Spend], REMOVEFILTERS(Dim_Product)))` | 0.0%   |
| Product Rank             | `RANKX(ALL(Dim_Product[Product]), [Total Spend],, DESC, Dense)`               | 0      |

### Channel Performance
| Measure                                | Formula                                                                               | Format  |
| -------------------------------------- | ------------------------------------------------------------------------------------- | ------- |
| Avg Purchases per Customer             | `DIVIDE([Total Purchases], [Total Customers])`                                        | #,##0.0 |
| Channel % of Total                     | `DIVIDE([Total Purchases], CALCULATE([Total Purchases], REMOVEFILTERS(Dim_Channel)))` | 0.0%    |
| Channel Rank                           | `RANKX(ALL(Dim_Channel[Channel]), [Total Purchases],, DESC, Dense)`                   | 0       |
| Total Web/Store/Catalog/Deal Purchases | `CALCULATE([Total Purchases], Dim_Channel[Channel] = "...")`                          | #,##0   |

### Campaign Effectiveness
| Measure                             | Formula                                                                                    | Format   |
| ----------------------------------- | ------------------------------------------------------------------------------------------ | -------- |
| Acceptance Rate                     | `DIVIDE([Total Acceptances], [Total Campaign Offers])`                                     | 0.0%     |
| Campaign % of Total Acceptances     | `DIVIDE([Total Acceptances], CALCULATE([Total Acceptances], REMOVEFILTERS(Dim_Campaign)))` | 0.0%     |
| Unique Accepted Customers           | `CALCULATE(DISTINCTCOUNT(...), ... = 1)`                                                   | #,##0    |
| Campaign Rank                       | `RANKX(ALL(Dim_Campaign[Campaign]), [Total Acceptances],, DESC, Dense)`                    | 0        |
| Avg Campaigns Accepted per Customer | `AVERAGEX(Dim_Customer, Dim_Customer[TotalCampaignsAccepted])`                             | #,##0.00 |

### Web Purchase Drivers
| Measure                        | Formula                                                                | Format  |
| ------------------------------ | ---------------------------------------------------------------------- | ------- |
| Avg Web Purchases per Customer | `DIVIDE([Total Web Purchases], [Total Customers])`                     | #,##0.0 |
| Web Purchase Share             | `DIVIDE([Total Web Purchases], [Total Purchases])`                     | 0.0%    |
| Web Visit to Purchase Ratio    | `DIVIDE([Avg Web Visits per Month], [Avg Web Purchases per Customer])` | #,##0.0 |
