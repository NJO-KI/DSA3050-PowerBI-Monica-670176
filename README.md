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
Measure NameWhat It CalculatesWhy It Is UsefulMain FunctionsFilter Context EffectDashboard LocationOTDR %  Percentage of total orders shipped on or ahead of scheduled date.  Core fulfillment efficiency metric for supply chains.  DIVIDE, COUNTROWS, CALCULATE  Evaluates within active Date, Market, and Carrier slicers.  Page 1 Executive KPI Ribbon  Late Delivery Risk %  Share of order lines marked with shipping delays.  Identifies operational failure rates across carriers.  DIVIDE, CALCULATE  Filters FactShipments where Late_delivery_risk = 1.  Page 1 & 2 Cards/Tables  YoY Revenue Growth %  Growth in order revenue relative to the prior year.  Tracks annual commercial expansion.  VAR, CALCULATE, SAMEPERIODLASTYEAR, DIVIDE  Shifts current DimDate filter context back by 1 year.  Page 1 Trend Analysis  % Revenue Share by Region  Regional revenue contribution relative to global revenue.  Highlights geographical revenue concentration.  DIVIDE, CALCULATE, ALL  Clears row context on DimLocation[Order Region].  Page 2 Regional Analysis  Logistics Health Status  Dynamic status indicator based on threshold risk rates.  Gives management instant visual status evaluation.  VAR, SWITCH, TRUE()  Re-evaluates status dynamically based on active filters.  Page 1 Executive Header  PY Revenue  Total Revenue generated in the same historical period last year.  Provides benchmark for current performance.  CALCULATE, SAMEPERIODLASTYEAR  Modifies filter context on DimDate[Date].  Page 1 Comparative Visuals  
---

## 5. Section E: Dashboards & Storytelling
### Executive Overview
![Overview](screenshots/04_dashboard_overview.png)

### Detailed Logistics Analysis
![Analysis](screenshots/05_dashboard_analysis.png)

### Diagnostic Insights
![Insights](screenshots/06_dashboard_insights.png)
