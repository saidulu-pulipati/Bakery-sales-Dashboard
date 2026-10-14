# Bakery-sales-Dashboard

# 🥐 Bakery Sales Dashboard (Power BI)

An interactive Power BI dashboard that analyzes two years of bakery sales (Jan 2023 to Dec 2024) across revenue, profit, waste, promotions, customer segments, and peak trading hours.

## 🎯 Problem Statement

Despite generating 3.44M in revenue, the bakery reported a loss of 230K and inventory waste of 1.39M. The goal was to identify unprofitable products, ineffective promotions, and opportunities to improve profitability.

This dashboard turns raw sales data into KPIs and visuals that answer:
- Is the bakery profitable, and how does it compare with last year?
- Which categories and products make or lose money?
- How much stock is wasted, and where?
- Do promotions and loyalty customers actually help?
- Which days and hours bring the most revenue?

## 🛠️ Tools & Skills

| Tool | Usage |
|---|---|
| Power BI Desktop | Dashboard design and visuals |
| DAX | KPI measures, YoY comparison, trend arrows, conditional colors |
| Power Query | Data cleaning and transformation |
| Excel / CSV | Source data |

## 📂 Dataset

- **Period:** 01-01-2023 to 31-12-2024
- **Transactions:** 16,569
- **Key fields:** Date, Day_of_Week, Hour, Store_ID, Season, Category, Product, Customer_Segment, Loyalty, Promotion_Applied, Revenue, Profit, Waste, Sell-through
- **Source:** [https://1drv.ms/x/c/7c4c85ed95ff2758/IQAXDDVPQclbTpvCh6XzYZgaAVwWchXmGIL0cSg3-twApfc?e=rO6MFZ]

## 🧹 Data Preparation

- Removed duplicates and null values
- Fixed data types (dates, currency, whole numbers)
- Created a **dim_Date,dim_products,dim_customers,fact_sales,fact-inventory,dim_store** 
- Built relationship: `Date[Date]` → `Sales[Date]` (one-to-many)
- `products[product-id]` → `Sales[product_id]` (one-to-many)
- `customers[customer_id]` → `Sales[customer_id]` (one-to-many)
- `store[store_id]` → `Sales[store_id]` (one-to-many)

---

## 📊 Dashboard Overview

**Slicers:** Season, Promotion_Applied, Customer_Segment, Store_ID, Category, Date range

**KPI cards (with YoY trend arrows):**

| KPI | Value | vs Last Year |
|---|---|---|
| total_revenue | 3.44M | ▼ -0.20% |
| Total_Profit | -230.34K | ▼ -0.01% |
| Profit_margin | -6.69% | ▼ -0.21% |
| Total_Waste | 1.39M | ▼ -0.24% (good) |
| Waste_margin | 40.32% | n/a |
| Transactions | 16,569 | n/a |

**Visuals:**
1. Revenue and profit trend by month (line chart)
2. Total profit by category (bar chart)
3. Top 5 and Bottom 5 products (column charts)
4. Total waste and sell-through by category (combo chart)
5. Profit by promotion type (column chart)
6. Revenue by loyalty vs non-loyalty (donut chart)
7. Revenue by customer segment (treemap)
8. Revenue by day of week and hour (matrix heat table)

---

## 📐 DAX Measures

> Adjust table and column names to match your model.

### Core KPIs
```DAX
total_revenue = SUM ( Sales[Revenue] )

Total_Profit = SUM ( Sales[Profit] )

Profit_margin = DIVIDE ( [Total_Profit], [total_revenue], 0 )

Total_Waste = SUM ( Sales[Waste_Value] )

Waste_margin = DIVIDE ( [Total_Waste], [total_revenue], 0 )

Transactions = DISTINCTCOUNT ( Sales[Transaction_ID] )

Sell Through % = DIVIDE ( SUM ( Sales[Units_Sold] ), SUM ( Sales[Units_Stocked] ), 0 )
```

### Last Year and YoY
```DAX
Revenue LY = CALCULATE ( [total_revenue], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Profit LY  = CALCULATE ( [Total_Profit],  SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Margin LY  = CALCULATE ( [Profit_margin], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
Waste LY   = CALCULATE ( [Total_Waste],   SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

Revenue YoY % = DIVIDE ( [total_revenue] - [Revenue LY], [Revenue LY], 0 )
Profit YoY %  = DIVIDE ( [Total_Profit] - [Profit LY], ABS ( [Profit LY] ), 0 )
Margin YoY %  = [Profit_margin] - [Margin LY]
Waste YoY %   = DIVIDE ( [Total_Waste] - [Waste LY], [Waste LY], 0 )
```

### Share and Segment Measures
```DAX
Loyalty Revenue % =
DIVIDE (
    CALCULATE ( [total_revenue], Sales[Loyalty] = "Yes" ),
    [total_revenue],
    0
)

Segment Share % =
DIVIDE ( [total_revenue], CALCULATE ( [total_revenue], ALL ( Sales[Customer_Segment] ) ), 0 )
```

---

## 🔺 Trend Arrows (KPI cards)

```DAX
Revenue Trend =
VAR _g = [Revenue YoY %]
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( _g ), "▬",
        _g > 0, "▲ " & FORMAT ( _g, "0.00%" ) & " Vs last year",
        _g < 0, "▼ " & FORMAT ( ABS ( _g ), "0.00%" ) & " Vs last year",
        "▬ 0.00% Vs last year"
    )
```

Repeat the same pattern for **Profit**, **Profit margin**, and **Waste** using their YoY measures. Place each text measure under its KPI card as a small card or in the card's subtitle.

---

## 🎨 Conditional Formatting

### 1. Trend arrow color
**Revenue, Profit, Margin:** higher is better.
**Waste:** lower is better, so the logic is reversed (a ▼ in waste shows green).

```DAX
Revenue Color =
SWITCH ( TRUE (), [Revenue YoY %] > 0, "#008000", [Revenue YoY %] < 0, "#FF0000", "#F9A825" )

Waste Color =
SWITCH ( TRUE (), [Waste YoY %] < 0, "#008000", [Waste YoY %] > 0, "#FF0000", "#F9A825" )
```

**Apply:** select the card → **Format** → **Font color** → **fx** → Format style: **Field value** → choose the color measure.

### 2. Profit bars (green positive, red negative)
Used on: **Total Profit by Category**, **Top 5 / Bottom 5 Products**, **Profit by Promotion Type**.

| Condition | Color | Hex |
|---|---|---|
| Profit ≥ 0 | 🟢 Green | `#008000` |
| Profit < 0 | 🔴 Red | `#FF0000` |

**Apply:** visual → **Format** → **Bars/Columns** → **Color** → **fx** → Format style: **Rules** → If value ≥ 0 then green, else red.

### 3. Heat map matrix (Day of Week × Hour)
- Blue and gold alternating banding for readability
- Optional: **Cell elements → Background color → Gradient** (light to dark) to highlight peak hours

---

## 💡 Key Insights

1. The bakery earns **3.44M revenue but a loss of -230.34K** (profit margin **-6.69%**).
2. **Waste is 1.39M, or 40.32% of revenue**, the biggest driver of the loss.
3. **Beverages** are the only strong profit category (**75K**). **Cakes (-108K)**, **Dairy (-106K)**, and **Spreads (-74K)** lose the most.
4. **Top products:** Coffee (35K), Cold Coffee (22K), Tea (14K). **Worst products:** Black Forest Cake (-55K), Chocolate Cake (-40K), Honey (-39K).
5. **Every promotion type loses money.** Weekend promotions are the worst at **-145K**.
6. **Loyalty customers bring 1.24M (35.88%)** of revenue, while non-loyalty customers bring 2.21M (64.12%).
7. **Family (1.08M)** and **Young Professional (1.05M)** are the top segments. Senior Citizen is the smallest (0.43M).
8. **Saturday (807K)** and **Sunday (734.58K)** are the busiest days.

## ✅ Recommendations

- Reduce baking quantities for **cakes and dairy** to cut waste and losses
- Review or stop **Weekend** and **Flash Sale** promotions
- Push high-margin **beverages** (coffee, tea) as combos with bakery items
- Grow the **loyalty program** to convert more non-loyalty customers
- Increase staffing and fresh stock on **Saturday and Sunday**
- Discount unsold items near closing time instead of discarding them


## 👤 Author

**Saidulu**
B.Tech CSE (AI & ML) | Aspiring Data Analyst
📍 Hyderabad, India
🔗 [LinkedIn](#) | [GitHub](#) | 📧 [Email](#)
