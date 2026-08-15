# Business Intelligence Solution: Global Supply Chain & Logistics
**Course:** DSA 3050A Business Intelligence & Data Visualization  
**Student Name:** Monica Njoki  
**Registration Number:** 670176 

---

## 1. Section A: Dataset Selection & Understanding
- **Source:** DataCo Smart Supply Chain Dataset.
- **Description:** Real-world transactional fulfillment dataset tracking shipments, order profits, and delay risks.
- **Business Problem:** High late-delivery rates and unoptimized carrier routes causing revenue leakage.
- **Key Questions Answered:**
  1. What is the overall On-Time Delivery Rate (OTDR %)?
  2. Which shipping modes produce the highest delay risk[cite: 1]?
  3. How does revenue compare Year-over-Year (YoY)[cite: 1]?
  4. Which regions suffer from severe fulfillment bottlenecks[cite: 1]?
  5. What root causes drive late delivery flags[cite: 1]?

---

## 2. Section B: Power Query Transformations
1.	Customer Name Formatting
• Problem: Customer names were split across `Customer Fname` and `Customer Lname`, leaving trailing spaces and missing values.
   • Transformation: Selected both columns -> Transform -> Merge Columns (Separator: Space) -> Name: `Customer Name`.
   • Reason: Provides a clean single string attribute for customer identification in reports.
   • Result: Single standardized column `Customer Name` without whitespace errors.
 
2.	Date/Time Datatype Correction
• Problem: `order date (DateOrders)` and `shipping date (DateOrders)` were imported as Text fields with time strings.
   • Transformation: Applied `Change Type` to `DateTime`, then extracted `Date` for modeling.
   • Reason: Enables proper date relationship linking and Time Intelligence DAX functions.
   • Result: Fields converted to standard ISO `Date` data types.
 
3.	Calculate Shipping Delay Gap (Custom Column)
• Problem: Raw data tracks real and scheduled shipping days separately without showing explicit delay variance.
   • Transformation: Added Custom Column: `[Days for shipping (real)] - [Days for shipment (scheduled)]` named `Shipping Delay Days`.
   • Reason: Quantifies exact delay duration per order for fulfillment analysis.
   • Result: Numerical column with positive values (delays), zero (on-time), or negative (early).
 
4.	Standardize Delivery Status Categories
• Problem: `Delivery Status` contained inconsistent values such as "SUSPECTED_FRAUD", "LATE_DELIVERY", "CANCELED".
   • Transformation: Applied `Replace Values` and `Capitalize Each Word` to format as "Late Delivery", "Suspected Fraud", etc.
   • Reason: Ensures visually appealing and consistent category labels in slicers and visuals.
   • Result: Clean, human-readable text categories.
 
5.	Filtering Out Fraudulent Transactions
• Problem: Transactions flagged as `Suspected Fraud` skew fulfillment performance calculations.
   • Transformation: Applied row filter on `Order Status` != "SUSPECTED_FRAUD".
   • Reason: Prevents compromised/blocked orders from distorting operational shipping KPIs.
   • Result: Dataset scoped strictly to valid commercial fulfillment records.
 
6.	Text Trimming & Cleaning
• Problem: `Market` and `Order Region` strings contained leading/trailing space whitespace.
   • Transformation: Selected text dimension columns -> `Transform` -> `Text Column` -> `Trim`.
   • Reason: Eliminates duplicate category groupings caused by hidden spaces.
   • Result: Consistent, clean string values across all region dimensions.
 
7.	Dimensional Table Creation (Reference & Deduplicate)
• Problem: Raw dataset is a single flat table containing customer, product, and location attributes mixed with measures.
   • Transformation: Right-clicked source query -> `Reference` -> Kept `Customer Id`, `Customer Name`, `Customer Segment` -> `Remove Duplicates`.
   • Reason: Extracts normalized dimension tables to build an optimized Star Schema.
   • Result: Isolated `DimCustomer` table with unique `Customer Id` keys.
 
8.	Conditional Column for Late Risk Category
• Problem: `Late_delivery_risk` was stored as binary `1` and `0`.
   • Transformation: Added Conditional Column: IF `Late_delivery_risk` = 1 THEN "High Risk" ELSE "Low/No Risk".
   • Reason: Improves dashboard readability for non-technical stakeholders.
   • Result: Descriptive categorical field `Delivery Risk Level`.
 



![Power Query]<img width="975" height="399" alt="image" src="https://github.com/user-attachments/assets/ca156b5c-3c93-47cf-977b-9cee3475d289" />
)

---

## 3. Section C: Data Model & Star Schema
                  ┌──────────────────────┐
                  │       DimDate        │
                  └──────────┬───────────┘
                             │ (1:N)
┌────────────────┐  (1:N)    ▼    (N:1)  ┌────────────────┐
│  DimCustomer   ├────────► Fact ◄───────┤   DimProduct   │
└────────────────┘       Shipments       └────────────────┘
                             ▲
                             │ (N:1)
                  ┌──────────┴───────────┐
                  │     DimLocation      │
                  └──────────────────────┘
DimCustomer[Customer Id] $\rightarrow$ FactShipments[Customer Id]Cardinality: One to Many (1:*)  Cross filter direction: Single  DimProduct[Product Card Id] $\rightarrow$ FactShipments[Product Card Id]Cardinality: One to Many (1:*)  Cross filter direction: Single  DimLocation[LocationKey] (or Region/Country keys) $\rightarrow$ FactShipmentsCardinality: One to Many (1:*)  Cross filter direction: Single  DimDate[Date] $\rightarrow$ FactShipments[Order Date]Cardinality: One to Many (1:*)  Cross filter direction: Single  

![Data Model](<img width="975" height="240" alt="image" src="https://github.com/user-attachments/assets/3c8bcc93-d534-4534-84b5-4cfa3cc46445" />
)

---


## 4. Section D: DAX & Business Calculations

| Measure Name | What It Calculates | Why It Is Useful | Main Functions | Filter Context Effect | Dashboard Location |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `OTDR %`[cite: 1] | Percentage of total orders shipped on or ahead of scheduled date[cite: 1]. | Core fulfillment efficiency metric for supply chains[cite: 1]. | `DIVIDE`, `COUNTROWS`, `CALCULATE`[cite: 1] | Evaluates within active Date, Market, and Carrier slicers[cite: 1]. | Page 1 Executive KPI Ribbon[cite: 1] |
| `Late Delivery Risk %`[cite: 1] | Share of order lines marked with shipping delays[cite: 1]. | Identifies operational failure rates across carriers[cite: 1]. | `DIVIDE`, `CALCULATE`[cite: 1] | Filters `FactShipments` where `Late_delivery_risk = 1`[cite: 1]. | Page 1 & 2 Cards/Tables[cite: 1] |
| `YoY Revenue Growth %`[cite: 1] | Growth in order revenue relative to the prior year[cite: 1]. | Tracks annual commercial expansion[cite: 1]. | `VAR`, `CALCULATE`, `SAMEPERIODLASTYEAR`, `DIVIDE`[cite: 1] | Shifts current `DimDate` filter context back by 1 year[cite: 1]. | Page 1 Trend Analysis[cite: 1] |
| `% Revenue Share by Region`[cite: 1] | Regional revenue contribution relative to global revenue[cite: 1]. | Highlights geographical revenue concentration[cite: 1]. | `DIVIDE`, `CALCULATE`, `ALL`[cite: 1] | Clears row context on `DimLocation[Order Region]`[cite: 1]. | Page 2 Regional Analysis[cite: 1] |
| `Logistics Health Status`[cite: 1] | Dynamic status indicator based on threshold risk rates[cite: 1]. | Gives management instant visual status evaluation[cite: 1]. | `VAR`, `SWITCH`, `TRUE()`[cite: 1] | Re-evaluates status dynamically based on active filters[cite: 1]. | Page 1 Executive Header[cite: 1] |
| `PY Revenue`[cite: 1] | Total Revenue generated in the same historical period last year[cite: 1]. | Provides benchmark for current performance[cite: 1]. | `CALCULATE`, `SAMEPERIODLASTYEAR`[cite: 1] | Modifies filter context on `DimDate[Date]`[cite: 1]. | Page 1 Comparative Visuals[cite: 1] |
---

## 5. Section E: Professional Power BI Dashboards & Storytelling

### Dashboard Architecture Overview

| Page & Theme | Visual Name / Type | Fields & Configuration | Analytical Purpose |
| :--- | :--- | :--- | :--- |
| **Page 1: Executive Overview** <br> *(What Happened?)* | **Header Slicers** | `Year`, `Market`, `Shipping Mode` | Enables global filtering across the entire report page[cite: 1]. |
| | **KPI Ribbon Cards** | `Total Revenue`, `Total Orders`, `OTDR %`, `Late Delivery Risk %`, `Logistics Health Status` | Gives executives immediate top-line performance visibility[cite: 1]. |
| | **Line & Stacked Bar Chart** | **X-Axis:** `Year-Month` <br> **Column:** `Total Revenue` <br> **Line:** `Late Delivery Risk %` | Tracks volume trends against shipping delay rates over time[cite: 1]. |
| | **Filled Geographic Map** | **Location:** `Order Country` <br> **Color Fill:** `Total Revenue` <br> **Tooltips:** `OTDR %` | Displays spatial revenue density alongside delivery efficiency[cite: 1]. |
| | **Donut Chart** | **Legend:** `Customer Segment` <br> **Values:** `Total Orders` | Breaks down order volume by customer classification[cite: 1]. |
| **Page 2: Detailed Analysis** <br> *(Where Did It Happen?)*[cite: 1] | **Matrix Table** | **Rows:** `Market` $\rightarrow$ `Order Region` <br> **Columns:** `Shipping Mode` <br> **Values:** `Total Revenue`, `Late Delivery Risk %`, `Avg Shipping Days` | Provides granular drill-down across logistics channels[cite: 1]. |
| | **Clustered Bar Chart** | **Y-Axis:** `Category Name` (Top 10) <br> **X-Axis:** `Total Delayed Orders` | Highlights product categories causing the most fulfillment friction[cite: 1]. |
| | **Scatter Plot** | **X-Axis:** `Avg Shipping Days` <br> **Y-Axis:** `Profit Margin %` <br> **Size:** `Total Revenue` <br> **Legend:** `Market` | Detects margin leakage relative to extended shipping lead times[cite: 1]. |
| **Page 3: Diagnostic Insights** <br> *(Why Did It Happen?)*[cite: 1] | **Decomposition Tree** | **Analyze:** `Late Delivery Risk %` <br> **Explain By:** `Market` $\rightarrow$ `Shipping Mode` $\rightarrow$ `Category Name` $\rightarrow$ `Customer Segment` | Enables ad-hoc root-cause breakdown of operational delays[cite: 1]. |
| | **Key Influencers Visual** | **Target:** `Late_delivery_risk` = High Risk <br> **Factors:** Shipping mode, Order region, Discount rate | Leverages built-in ML to find key statistical drivers of late shipments[cite: 1]. |
| | **Waterfall Chart** | **Category:** Revenue $\rightarrow$ Discounts $\rightarrow$ Freight Costs $\rightarrow$ Net Profit | Explains profit erosion from gross revenue down to net profitability[cite: 1]. |

### Detailed Logistics Analysis
![Analysis](screenshots/05_dashboard_analysis.png)
┌─────────────────────────────────────────────────────────────────────┐
│  📊 DETAILED LOGISTICS ANALYSIS - Page 2                          │
├───────────────┬─────────────────────────────────────────────────────┤
│               │                                                    │
│  Market       │   MATRIX: Revenue & Delay by Market/Region/       │
│  Filters:     │   Shipping Mode                                   │
│  [All]        │   ┌──────────────────────────────────────────┐    │
│               │   │ Market   │ Region  │ Mode   │ Rev   │Risk│    │
│  Region       │   │──────────┼─────────┼────────┼───────┼─────┤    │
│  [All]        │   │ LATAM    │ Brazil  │ Air    │ $2.4M │ 18%│    │
│               │   │ LATAM    │ Brazil  │ Road   │ $1.1M │ 34%│    │
│  Time Period  │   │ LATAM    │ Mexico  │ Air    │ $1.8M │ 22%│    │
│  [2023-24]    │   │ EU       │ Germany │ Air    │ $4.2M │ 9% │    │
│               │   │ EU       │ France  │ Sea    │ $3.1M │ 14%│    │
│               │   │ APAC     │ Japan   │ Air    │ $5.6M │ 7% │    │
│               │   │ APAC     │ India   │ Road   │ $2.9M │ 28%│    │
│               │   └──────────────────────────────────────────┘    │
│               │                                                    │
├───────────────┴─────────────────────────────────────────────────────┤
│                                                                    │
│  📦 TOP DELAYED PRODUCT CATEGORIES         📈 MARGIN vs LEAD TIME │
│  ┌──────────────────────────────────┐   ┌──────────────────────┐ │
│  │ Electronics         ████████████ │   │ 30% │  • APAC        │ │
│  │ Furniture           ███████████  │   │     │  • EU          │ │
│  │ Apparel             ██████████   │   │ 20% │  • NA          │ │
│  │ Auto Parts          ████████    │   │     │  • LATAM ██    │ │
│  │ Home Goods          ███████     │   │ 10% │        ████    │ │
│  └──────────────────────────────────┘   │  0% │___________██  │ │
│                                          │     │ 5  10  15  20 │ │
│                                          └──────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘

* **Analytical Focus:** Evaluates carrier efficiency, regional shipment performance, and product category bottlenecks[cite: 1].
* **Key Visuals:**
  * **Market Matrix:** Drills down from `Market` to `Order Region` across shipping modes[cite: 1].
  * **Top Delayed Categories:** Identifies the top 10 product lines driving fulfillment delays[cite: 1].
  * **Margin vs. Lead Time Scatter:** Spotlights regional margin erosion caused by extended shipping lead times[cite: 1].

---

### Diagnostic Insights
![Insights](screenshots/06_dashboard_insights.png)
┌─────────────────────────────────────────────────────────────────────┐
│  🔍 DIAGNOSTIC INSIGHTS - Root Cause Analysis                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  🎯 DECOMPOSITION TREE: Late Delivery Risk %                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │   TOTAL: 17.3% Late Risk                                   │   │
│  │         │                                                  │   │
│  │    ┌────┴────┐                                            │   │
│  │    │         │                                            │   │
│  │  APAC      LATAM    EU       NA                           │   │
│  │  12%      28%     13%      16%                           │   │
│  │    │         │                                            │   │
│  │  ┌─┴─┐    ┌──┴──┐                                       │   │
│  │ Air Road  Air  Road Sea                                  │   │
│  │  7% 22%   18%  34% 24%                                  │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                    │
│  🤖 KEY INFLUENCERS: What Drives Late Delivery?                  │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  Shipping Mode = Road             ▲ +22% Risk             │   │
│  │  Order Region = Brazil            ▲ +16% Risk             │   │
│  │  Discount Rate > 15%              ▲ +11% Risk             │   │
│  │  Product Category = Furniture     ▲ +9% Risk              │   │
│  │  Customer Segment = Consumer      ▲ +5% Risk              │   │
│  └────────────────────────────────────────────────────────────┘   │
│                                                                    │
│  💰 WATERFALL: Revenue → Net Profit Erosion                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │                                                           │   │
│  │  $100M                                                    │   │
│  │   │  ████████████████████████████████████                │   │
│  │   │  │ Revenue   $100M                                   │   │
│  │   │  ▼ ─$8M ──▼ ─$5M ──▼ ─$3M ──▼ ─$2M ──▼            │   │
│  │   │  Dscnt  Freight  Returns  Admin   Net               │   │
│  │   │                         Profit: $82M                 │   │
│  │   └─────────────────────────────────────────────────────────  │   │
│  └────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘

* **Analytical Focus:** Root-cause investigation into late shipping drivers and profit leakage[cite: 1].
* **Key Visuals:**
  * **Decomposition Tree:** Deconstructs `Late Delivery Risk %` across regional, operational, and customer tiers[cite: 1].
  * **Key Influencers Visual:** Uses machine learning models to highlight the top statistical drivers of high delivery risks[cite: 1].
  * **Waterfall Chart:** Maps financial flow from gross revenue down to net order profit[cite: 1].
