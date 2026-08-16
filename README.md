# AI Supply Chain Risk Intelligence Platform
## Part 2 — Statistical EDA, Hypothesis Testing & Visualization


## Overview

This analysis was performed as part of the **AI Supply Chain Risk Intelligence Platform**. The objective of this analysis is to understand supply-chain risk patterns, their financial impact, and the operational areas that should be prioritized for risk monitoring.

The analysis includes descriptive statistics, NumPy analysis, grouped analysis, risk-exposure segmentation, correlation analysis, hypothesis testing, and visualization.

---

## 1. Project Overview

The **AI Supply Chain Risk Intelligence Platform** is designed to analyze supply-chain operations and identify patterns related to operational risk, financial loss, product categories, suppliers, warehouses, and shipping performance.

This part of the project focuses on:

- Statistical Exploratory Data Analysis (EDA)
- NumPy numerical analysis
- Descriptive statistics
- Feature engineering
- Grouped analysis
- Risk-exposure segmentation
- Correlation analysis
- Hypothesis testing
- Data visualization
- Data storytelling
- Business recommendations

The objective is to move from basic descriptive findings to deeper statistical analysis and finally convert the findings into actionable supply-chain recommendations.

---

# 1: Initial Inspection

The cleaned supply-chain dataset was loaded into Pandas and inspected before performing statistical analysis.

### Dataset Dimensions

The dataset contains:

- **44,965 rows**
- **49 columns**

The dataset contains order-item level supply-chain information related to:

- Orders
- Products
- Customers
- Suppliers
- Warehouses
- Shipments
- Pricing
- Sales
- Profit
- Delivery performance
- Risk events
- Estimated financial loss

### Data Structure

"df.info()" was used to inspect column names, data types, and non-null values.

The dataset contains numerical and categorical variables.

Most columns contain **44,965 non-null values**.

Two columns contain missing values:

- "actual_delivery_date" — **11,896 missing values**
- "delivery_days" — **11,896 missing values**

These missing values are related to delivery information.

### Duplicate Check

The dataset was checked for duplicate records.

**Duplicate rows = 0**

Therefore, no duplicate rows were identified.

### Descriptive Overview

"df.describe(include='all')" was used to obtain descriptive statistics for numerical and categorical columns.

Important observations include:

- There are **15,000 unique orders** represented across **44,965 order-item records**.
- "quantity" ranges from **1 to 5**, with a mean of approximately **2.997**.
- "risk_event_count" ranges from **0 to 4**.
- "total_estimated_loss" has a mean of approximately **11,226.33**.
- "net_sales" has a mean of approximately **48,280.78**.
- "profit" has a mean of approximately **10,362.74**.

The initial inspection confirms that the dataset is suitable for statistical exploratory analysis.

---

# 2: NumPy Fundamentals

NumPy was used to perform numerical calculations and Boolean filtering.

### NumPy Array Conversion

The "shipping_cost" and "quantity" columns were converted into NumPy arrays:


s = np.array(df["shipping_cost"])
q = np.array(df["quantity"])


This allowed the numerical calculations to be performed using NumPy operations.

### Vectorized Operation

A vectorized calculation was performed to calculate shipping cost per unit:

shipping_cost_per_unit = s / q

The first ten values were:

[103.76, 90.166, 90.166, 75.21, 259.42,
 129.71, 373.24, 93.31, 124.41333333, 186.62]

This demonstrates vectorized numerical calculation without using an explicit loop.

### Boolean Filtering

A two-condition Boolean filter was used to identify shipping-cost-per-unit values between 50 and 100:

```python
f = scpu[(scpu >= 50) & (scpu <= 100)]
```

The filter returned:

**6,973 values**

The first ten filtered values were:

[90.166, 90.166, 75.21, 93.31, 58.04,
 58.04, 58.915, 97.182, 59.92, 55.175]

This demonstrates NumPy Boolean indexing using two conditions.


# 3: Descriptive Statistics

Descriptive statistics were calculated for two important financial variables "net_sales"
and "profit".

The following statistics were calculated:

- Mean
- Median
- Standard deviation
- Variance
- 90th percentile

## Net Sales

| Statistic | Value |
|---|---:|
| Mean | 48,280.78 |
| Median | 11,350.35 |
| Standard deviation | 98,655.38 |
| Variance | 9,732,883,181.92 |
| 90th percentile | 138,664.93 |

The mean is considerably higher than the median, indicating substantial variation in transaction-level net sales.

### Profit

| Statistic | Value |
|---|---:|
| Mean | 10,362.74 |
| Median | 2,484.69 |
| Standard deviation | 21,739.30 |
| Variance | 472,597,238.80 |
| 90th percentile | 28,628.68 |

The mean profit is also substantially higher than the median, indicating variation and a right-skewed financial distribution.

### Business Relevance

These descriptive statistics provide a baseline understanding of the financial characteristics of the supply-chain transactions before moving into deeper analysis.

---

# 4: Feature Engineering

Two additional analytical features were created to support the supply-chain risk analysis.

### Gross Amount

A new gross_amount column was created:

df["gross_amount"] = df["quantity"] * df["unit_price"]

This represents the gross transaction value based on quantity and unit price.

### Risk Exposure Percentage

A new risk_exposure_percentage column was calculated:

df["risk_exposure_percentage"] = (
    df["total_estimated_loss"] / df["gross_amount"]
) * 100

This represents estimated financial loss relative to the gross transaction amount.

The engineered feature provides a relative measure of financial exposure and is later used for risk segmentation.

---

# 5: Grouped Analysis

Grouped analysis was performed using pivot tables and multi-aggregation groupby().agg() operations.

### Pivot Table 1 — Risk by Product Category

The first pivot table summarizes:

- Total risk events
- Total estimated loss

by product category.

category_risk = df.pivot_table(
    index="category_name",
    values=["risk_event_count", "total_estimated_loss"],
    aggfunc="sum"
)

### Highest Risk-Event Categories

| Product Category | Risk Events |
|---|---:|
| Bags & Luggage | 849 |
| Footwear | 816 |
| Health & Wellness | 803 |
| Stationery | 790 |
| Electronics | 788 |

Bags & Luggage recorded the highest number of risk events with 849 events.

### Highest Estimated Loss Categories

The highest total estimated losses included:

| Product Category | Total Estimated Loss |
|---|---:|
| Footwear | 29,574,707 |
| Office Supplies | 29,508,787 |
| Fashion | 28,913,769 |
| Appliances | 27,298,588 |
| Grocery | 27,262,361 |

This analysis helps identify product categories with high supply-chain risk and financial impact.

### Pivot Table 2 — Risk Exposure by Warehouse

The second pivot table examines the maximum observed risk exposure percentage for each warehouse.

warehouse_risk = df.pivot_table(
    index="warehouse_name",
    values="risk_exposure_percentage",
    aggfunc="max"
)

The highest observed values were:

| Warehouse | Maximum Risk Exposure % |
|---|---:|
| Bhopal Warehouse | 223,910.15 |
| Raipur Warehouse | 203,882.34 |
| Pune Warehouse | 183,665.88 |
| Gurugram Warehouse | 173,909.99 |
| Delhi Warehouse | 146,842.76 |

Bhopal Warehouse recorded the highest observed risk exposure percentage at 223,910.15%.

These extreme values identify warehouse-order combinations that may require additional investigation.

### Pivot Table 3 — Supplier Rating and Operational Performance

A third pivot table compared:

- Average delivery days
- Average lead time
- Average shipping cost

across supplier ratings.

For example:

- Supplier rating 4.3 had an average lead time of 17.97 days.
- Supplier rating 4.4 had an average lead time of 10.37 days.
- Supplier rating 4.8 had an average lead time of 16.74 days.
- Supplier rating 5.0 had an average lead time of 11.79 days.

This shows that supplier rating alone does not provide a complete picture of operational performance.

### Multi-Aggregation GroupBy — Supplier Type

A multi-aggregation groupby().agg() operation was performed by supplier type.

supplier_risk = df.groupby("supplier_type").agg(
{
    "risk_event_count": ["sum", "mean"],
    "total_estimated_loss": ["sum", "mean"]
}
)

The results were:

| Supplier Type | Total Risk Events | Avg Risk Events | Total Estimated Loss | Avg Estimated Loss |
|---|---:|---:|---:|---:|
| Distributor | 1,613 | 0.328 | 56,999,059 | 11,596.96 |
| Manufacturer | 13,458 | 0.336 | 447,793,073 | 11,180.85 |

Manufacturers account for the majority of recorded risk events and total estimated loss.

### Multi-Aggregation GroupBy — Shipping Partner

Shipping partners were also analyzed using:

shipping_performance = df.groupby("shipping_partner").agg(
{
    "delivery_days": ["min", "max"],
    "shipping_cost": ["mean", "sum"]
}
)

This analysis compares delivery-day ranges and shipping costs across shipping partners.

For example:

- Blue Dart — average shipping cost 527.95
- DTDC — 531.08
- Delhivery — 521.73
- Ecom Express — 528.87
- Ekart Logistics — 541.18
- India Post — 534.78
- Shadowfax — 520.85
- XpressBees — 518.98

This provides a comparison of shipping-partner operational and cost performance.

---

## 6: Bucket Segmentation

A custom function was created to divide risk_exposure_percentage into three categories.

The segmentation rules were:

- Low Risk Exposure: risk_exposure_percentage <= 25
- Medium Risk Exposure: 25 < risk_exposure_percentage <= 50
- High Risk Exposure: risk_exposure_percentage > 50

The function was applied using .apply():

df["risk_exposure_bucket"] = (
    df["risk_exposure_percentage"]
    .apply(risk_exposure_bucket)
)

The resulting distribution was:

| Risk Exposure Level | Number of Orders |
|---|---:|
| Low Risk Exposure | 35,809 |
| Medium Risk Exposure | 1,335 |
| High Risk Exposure | 7,821 |

The three categories account for all 44,965 records.

The new risk_exposure_bucket column was displayed using df.head() to verify the result.

### Business Interpretation

The largest group is Low Risk Exposure with 35,809 records.

However, 7,821 records are classified as High Risk Exposure, making this an important group for targeted monitoring.

The segmentation provides a practical framework:

- Low Risk Exposure → routine monitoring
- Medium Risk Exposure → enhanced monitoring

---


## 7:Correlation Analysis

Pearson correlation was calculated across the numerical columns of the dataset.

The correlation matrix was analyzed after excluding the diagonal values because each variable has a correlation of `1.0` with itself. These diagonal values do not represent a meaningful relationship between two different variables.

### Highest Absolute Correlation

The highest absolute correlation was found between:

- "unit_price"
- "selling_price"

**Absolute correlation = 1.0**

This indicates a perfect positive linear relationship between "unit_price" and "selling_price" in the dataset.

### Lowest Absolute Correlation

The lowest absolute correlation was found between:

- "quantity"
- "reserved_stock"

**Absolute correlation = 4.4403002039725265e-06/0.0000044403**

This value is extremely close to zero, indicating almost no linear relationship between "quantity" and "reserved_stock".

### Tie Handling

The diagonal values were excluded before identifying the highest and lowest correlation pairs.

No tied highest or lowest correlation pairs were identified, so no additional tie-breaking rule was required.

### NaN Handling

The correlation analysis produced:

**Number of NaN correlation values = 0**

Therefore, no NaN correlation values were present, and no removal or imputation of correlation values was required.

---

# 8:Hypothesis Test

## Business Claim

The business question tested was:

> **Does the average "total_estimated_loss" differ between orders with recorded risk events and orders without recorded risk events?**

This hypothesis is directly related to the supply-chain risk objective because it examines whether recorded risk events are associated with a difference in financial loss.

## Null Hypothesis (H₀)

There is **no difference** in the average "total_estimated_loss" between orders with recorded risk events and orders without recorded risk events.

## Alternative Hypothesis (H₁)

There **is a difference** in the average "total_estimated_loss" between orders with recorded risk events and orders without recorded risk events.

## Significance Level

The significance level was set to:

**α = 0.05**

## Test Used

A **two-sample t-test** was used because the analysis compares the average "total_estimated_loss" between two independent groups:

1. Orders with recorded risk events
2. Orders without recorded risk events

The unequal-variance version was used because the two groups have different variance characteristics.

### Assumption Check

The approximate normality of the relevant distribution was checked visually using a histogram of `total_estimated_loss` for orders with recorded risk events.

The distribution appears strongly right-skewed, with most observations concentrated toward lower estimated-loss values and a long tail toward higher values.

No formal normality test was performed because the task specifically states that a visual histogram/skewness check is sufficient.

The two groups were treated as independent because each row represents a separate order-item record.

---

### Hypothesis Test Results

A two-sample t-test was performed to compare the average "total_estimated_loss" between orders with risk events and orders without risk events.

* **Risk-event orders:** 12,697
* **No-risk orders:** 32,268
* **Mean loss for risk-event orders:** 39,756.80
* **Mean loss for no-risk orders:** 0.00
* **Test statistic (t):** 56.616
* **P-value:** < 0.001
* **Significance level (α):** 0.05
* **Decision:** Reject H₀

Since the p-value is less than the significance level of 0.05, the null hypothesis is rejected. There is statistically significant evidence that the average total estimated loss differs between orders with risk events and orders without risk events.

## Assumption Check

The **approximate normality assumption** was checked visually using a histogram of "total_estimated_loss" for risk-event orders.

The distribution was observed to be **strongly right-skewed**, with most observations concentrated toward lower loss values and a long tail toward higher loss values.

No formal normality test was performed because a visual histogram/skewness check was sufficient for this requirement.

The two groups were treated as independent because each row represents a separate order-item record.

The no-risk group has "total_estimated_loss = 0" for all observations, resulting in zero variance. Therefore, its distribution cannot be meaningfully assessed for normality.

---

# 9:Visualization Summary

The analysis includes the required visualizations:

1. **Correlation Heatmap**
   - Pearson correlation matrix
   - "annot=True"
   - Title and labelled axes

2. **Scatter Plot**
   - Shipping cost vs net sales
   - Risk exposure used as the categorical "hue"
   - Title and labelled axes

3. **Bar Plot**
   - Total estimated loss by product category
   - Title and labelled axes

4. **Distribution Plot**
   - Distribution of risk exposure percentage
   - Title and labelled axes

Additional project-focused visualizations include:

5. **Donut Chart**
   - Order distribution by risk exposure

6. **Horizontal Bar Chart**
   - Total supply-chain risk events by product category

All required visualizations were saved as ".png" files.

## 10:Data Storytelling

The EDA findings were connected into a multi-layer supply-chain risk story. Each layer builds on the previous finding to move from a broad descriptive observation to a deeper diagnostic finding and finally to actionable recommendations.

### Layer 1 — Descriptive: Where are supply-chain risk events concentrated?

The product-category analysis shows that **Bags & Luggage recorded the highest number of risk events with 849 events**, followed by Footwear with **816**, Health & Wellness with **803**, Stationery with **790**, and Electronics with **788**.

This indicates that risk events are not evenly distributed across product categories. Bags & Luggage is the category with the highest observed risk-event volume and therefore becomes an important area for further investigation.

**Recommendation:**  
Prioritize supply-chain risk monitoring and preventive controls for **Bags & Luggage**, followed by Footwear and Health & Wellness.

---

### Layer 2 — Diagnostic: Are risk events associated with financial loss?

After identifying where risk events are concentrated, the analysis examined whether recorded risk events are associated with financial impact.

There were **12,697 orders with recorded risk events** and **32,268 orders without recorded risk events**. The average "total_estimated_loss" for risk-event orders was **39,756.80**, compared with **0.00** for no-risk orders.

A two-sample t-test produced a **t-statistic of 56.6157** and a **p-value below 0.001**. Since the p-value is below the significance level of **0.05**, the null hypothesis was rejected.

This provides statistically significant evidence that the average estimated loss differs between risk-event and no-risk orders.

**Recommendation:**  
Orders with recorded risk events should be flagged for **priority financial-impact assessment, investigation, and risk mitigation**.

---

### Layer 3 — Deeper Diagnostic: Does the number of risk events relate to financial loss?

The analysis was then extended to examine whether the frequency of risk events is related to estimated financial loss.

The Pearson correlation between "risk_event_count" and "total_estimated_loss" was approximately **0.435**.

This represents a **moderate positive linear relationship**, meaning that higher numbers of recorded risk events tend to be associated with higher estimated financial loss. This correlation indicates an association and does not by itself establish causation.

**Recommendation:**  
Use "risk_event_count" as a **risk-prioritization indicator**. Orders with multiple recorded risk events should receive greater attention for investigation and corrective action.

---

### Layer 4 — Risk Prioritization: How can orders be segmented by exposure?

To make the risk findings easier to operationalize, orders were segmented using "risk_exposure_percentage".

The segmentation produced:

- **35,809 orders** as Low Risk Exposure
- **1,335 orders** as Medium Risk Exposure
- **7,821 orders** as High Risk Exposure

The largest segment is Low Risk Exposure with **35,809 orders**, while **7,821 orders** fall into the High Risk Exposure category.

This provides a practical way to prioritize risk-management resources instead of treating every order equally.

**Recommendation:**  
Adopt a tiered monitoring strategy:

- **Low Risk Exposure:** routine monitoring
- **Medium Risk Exposure:** enhanced monitoring
- **High Risk Exposure:** priority investigation and intervention

---

### Layer 5 — Deeper Business Action: Where should resources be focused?

Combining the category-level risk analysis, financial-loss analysis, risk-event correlation, and exposure segmentation provides a clearer supply-chain risk strategy.

The analysis shows that **Bags & Luggage has the highest observed risk-event volume at 849 events**, risk-event orders have an average estimated loss of **39,756.80**, and "risk_event_count" has a **0.435 positive correlation** with estimated loss. At the order level, **7,821 orders are classified as High Risk Exposure**.

These findings indicate that the supply-chain risk system should not only identify whether an order has a risk event, but should also consider the **frequency of risk events, financial impact, risk exposure, and product category** when prioritizing action.

**Recommendation:**  
Develop a prioritized risk-monitoring workflow that:

1. Flags orders with recorded risk events.
2. Gives higher priority to orders with multiple risk events.
3. Escalates High Risk Exposure orders.
4. Prioritizes categories with high risk-event volumes, particularly **Bags & Luggage**.
5. Uses estimated financial loss to prioritize cases with greater potential business impact.

---

### Overall Story

The analysis moves from **where risk occurs → whether risk has financial impact → whether risk frequency relates to loss → how orders can be segmented → where the business should focus its resources**.

---
