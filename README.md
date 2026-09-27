# DataCo Supply Chain Analysis: Why Are 57% of Orders Late?

An end-to-end supply chain analysis of the DataCo dataset (~66K orders, 2015–2018) using **Python** for data cleaning and **Power BI** for visualization. The project investigates why 57% of orders are delivered late by examining late rates across time, shipping mode, region, market, and customer segment. The analysis finds that **shipping mode is the only factor that meaningfully explains lateness**: premium modes consistently miss their delivery promises (First Class is late on 100% of orders, and Second Class takes roughly twice its scheduled time), while regional and customer-level differences are minimal or driven by small sample sizes.



---

## Key Takeaways

- **57.3% of orders are delivered late**, and the rate has stayed flat at ~55–59% for three years, so the problem is structural rather than seasonal.
- **Shipping mode drives lateness**: First Class 100%, Second Class 79.9%, Same Day 48.4%, Standard Class 39.9%.
- **Premium modes over-promise**: First Class promises 1 day and takes 2; Second Class promises 2 days and takes ~4.
- **Region, market, and customer segment make little difference.** The most extreme regions are also the smallest ones.

**Top recommendations**
1. Revise First and Second Class delivery promises to match actual performance (~2 and ~4 days), or invest to meet the current promise.
2. Investigate and reduce delivery-time variability in Standard Class.
3. Track on-time rate per shipping mode as a core KPI, always shown alongside order volume.

---

## Dataset

The [DataCo Smart Supply Chain dataset](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis), published on Kaggle, contains around 180,000 order-item records from a global retailer, covering orders placed between 2015 and early 2018. Each row represents a single item within an order and includes information about the customer (segment and location), the product (name, category, and department), the order (date, status, destination market, region, and country), financial details (price, quantity, discount, sales, and profit), and shipping details (shipping mode, scheduled vs actual shipping days, and delivery status). The dataset spans five markets: Africa, Europe, LATAM, Pacific Asia, and USCA.

> The raw dataset is not included in this repository because of its size. Download it from the Kaggle link above.

---

## Business Questions

1. How often are orders delivered late, and is this changing over time?
2. Which shipping modes are most likely to be late?
3. Are late deliveries caused by slow execution or by unrealistic delivery promises?
4. Is lateness concentrated in specific regions, markets, or customer segments?

---

## Tools

| Stage | Tools |
|---|---|
| Data cleaning & feature engineering | Python (pandas), Jupyter Notebook |
| Data modeling & dashboard | Power BI (DAX, Power Query) |

---

## Process

### 1. Data Cleaning (Python)
- **Removed personal data (PII):** customer names, email, password, and street address.
- **Removed duplicate columns** after verifying they were 100% identical (e.g. `Order Customer Id` vs `Customer Id`, `Benefit per order` vs `Order Profit Per Order`).
- **Removed irrelevant or derivable columns:** mostly-empty zip codes, store coordinates, customer location fields (almost all US/Puerto Rico), ID columns with a matching name column, and ratios that can be recalculated from other fields.
- **Converted** order and shipping dates to datetime and standardized column names to `snake_case`.
- **Validated** the data: no missing values and no duplicate rows.
- Reduced the dataset from 53 columns to 25 relevant columns.

### 2. Feature Engineering
- `shipping_delay_days` = actual shipping days − scheduled shipping days
- `is_late` = 1 if `shipping_delay_days` > 0, otherwise 0

### 3. Data Modeling (Power BI)
- Built a dedicated **date table** and related it to a date-only order date column.
- Created DAX measures instead of aggregating raw columns. Orders are counted with `DISTINCTCOUNT` because each row is an order *item*, not an order.
- Fixed a **locale issue** where decimal values were misread on import (Indonesian regional settings treat `.` as a thousands separator), which initially inflated revenue and profit.

```DAX
Total Orders   = DISTINCTCOUNT(dataco_clean[order_id])
Late Orders    = CALCULATE([Total Orders], dataco_clean[is_late] = 1)
Late Rate %    = DIVIDE([Late Orders], [Total Orders])
Avg Delay Days = AVERAGE(dataco_clean[shipping_delay_days])
Revenue        = SUM(dataco_clean[order_item_total])
Profit         = SUM(dataco_clean[order_profit_per_order])
```

---

## Findings & Recommendations

### Overall Performance (KPI Cards)
- ~66K orders generated **$33M revenue** and **$4M profit** (~12% margin).
- **57.3% of orders are late**, with an average delay of 0.57 days. The problem is how *often* orders are late, not how *long* they are delayed.

**Recommendation:** Make on-time delivery rate a primary operational KPI alongside revenue and profit.

### Late Rate Over Time
- The late rate stays within **~55–59% from 2015 to early 2018**, with no trend and no repeating seasonal pattern.
- Filtered to First Class, the rate is **100% in every month**.

**Recommendation:** Seasonal fixes (e.g. extra peak-season capacity) won't help. The cause lies in how delivery times are set per shipping mode.

### Late Rate by Shipping Mode

| Shipping mode | Late rate |
|---|---|
| First Class | **100%** |
| Second Class | **79.9%** |
| Same Day | **48.4%** |
| Standard Class | **39.9%** |

- The more premium the mode, the **worse** its reliability.
- The same ranking holds across **all markets** and **all customer segments** (each ~57%).

**Recommendation:** Prioritize First and Second Class, and review how premium shipping is priced and marketed until reliability improves.

### Scheduled vs Actual Delivery Days

| Shipping mode | Scheduled | Actual (avg) | Pattern |
|---|---|---|---|
| First Class | 1 day | 2 days | Always late by ~1 day |
| Second Class | 2 days | ~4 days | Takes about twice the promised time |
| Standard Class | 4 days | 4.00 days | On target on average, yet ~40% late |
| Same Day | 0 days | ~0.5 days | About half ship the next day |

Different modes fail for different reasons:
- **Premium modes: over-promising.** The average delivery time itself exceeds the promise.
- **Standard Class: inconsistency.** Early and late deliveries average out to exactly 4 days, but individual orders vary too much.

**Recommendations**
- **First & Second Class:** adjust scheduled delivery times to actual performance (~2 and ~4 days), or invest to meet the current promise.
- **Standard Class:** investigate what causes the slowest deliveries and reduce variability.
- **Same Day:** review order cut-off times or relabel the service.

### Shipping Mode by Region
- Within each mode, late rates are **nearly identical across regions**: First Class is 100% everywhere, Second Class ~75–86%, and Standard Class ~33–45%.
- Variation between regions (~10 pp) is far smaller than variation between modes (~60 pp).
- The most extreme regions are also the smallest: **Central Africa** (60.6% late, 556 orders) and **Canada** (52.8% late, 309 orders), each under 1% of all orders.

**Recommendations**
- Don't allocate resources region by region; apply the shipping-mode fixes company-wide.
- Always show order volume next to late rate when comparing regions.

---

## Limitations

- The data shows **what** is late but not **why**: there are no carrier, warehouse, or inventory fields.
- There is no customer satisfaction or repeat-purchase data, so the **business cost of lateness can't be measured directly**.
- Some regions (e.g. Canada, Central Africa) only have orders between Aug 2016 and Jan 2017, so regional comparisons cover different time periods and sample sizes.
