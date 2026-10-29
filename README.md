# Sales Performance Dashboard (Power BI)

An end-to-end Power BI report that models retail order data into a star schema and analyses **sales vs. targets, profitability, product mix and geography** for FY 2018-19 (Apr 2018 – Mar 2019).

## Dataset at a glance

| Metric                      | Value                            |
| --------------------------- | -------------------------------- |
| Period covered              | 01 Apr 2018 – 31 Mar 2019        |
| Orders                      | 500                              |
| Order line items            | 1,500                            |
| Customers                   | 401 across 19 states / 24 cities |
| Categories / Sub-categories | 3 / 17                           |
| Total sales amount          | 431,502                          |
| Total profit                | 23,955                           |
| Total sales target          | 435,900                          |

---

## Analysis covered

| Business question                                        | Visual(s)                                             |
| -------------------------------------------------------- | ----------------------------------------------------- |
| Are categories hitting their sales targets?              | Clustered column chart (actual vs target by category) |
| Which sub-categories have the best profit margin?        | Donut chart (max profit margin by sub-category)       |
| How do sales trend month over month?                     | Line chart (monthly sales trend)                      |
| How do profit and quantity relate across sub-categories? | Scatter chart (profit vs quantity by sub-category)    |
| How do total sales compare to the target?                | KPI cards (total sales amount, total target)          |
| What is the minimum target for each category?            | Multi-row card                                        |
| How does actual vs target look by category and month?    | Matrix                                                |
| Where are sales concentrated geographically?             | Map (sales amount by city)                            |
| How are sales distributed across sub-categories?         | Treemap                                               |
| How are orders distributed across states?                | Funnel chart (order count by state)                   |
| What are the headline order and profit figures?          | Cards (order count, average profit)                   |

---

## Data model

Star schema with two fact tables sharing conformed dimensions.

| Table                | Type      | Grain                                | Key columns                                                                                     |
| -------------------- | --------- | ------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `fact_order_details` | Fact      | One row per order line               | Order ID, category_key, order_date, month_of_order_date, Sub-Category, Amount, Profit, Quantity |
| `fact_sales_target`  | Fact      | Category × month                     | category_key, Month of Order Date, Target                                                       |
| `dim_orders`         | Dimension | One row per order                    | Order ID, Order Date, customer_key                                                              |
| `dim_customers`      | Dimension | One row per unique customer/location | customer_key, CustomerName, State, City                                                         |
| `dim_category`       | Dimension | One row per category                 | category_key, category_name                                                                     |
| `dim_calender`       | Dimension | One row per order date               | order_date, month_of_order_date                                                                 |
| `dim_month`          | Dimension | One row per month                    | Month of Order Date                                                                             |

---

## Power Query (ETL) highlights

- **`dim_customers`** – deduplicated customer/state/city combinations from _List of Orders_, with a generated `customer_key` surrogate key.
- **`dim_orders`** – merged with `dim_customers` to swap name/state/city for `customer_key`.
- **`dim_category`** – small static lookup (Clothing, Furniture, Electronics).
- **`fact_order_details`** – merged with `dim_category` to replace the text category with `category_key`; added `month_of_order_date` (start of month) for joining to targets.
- **`fact_sales_target`** – category text replaced by `category_key`.
- **`dim_calender` / `dim_month`** – derived distinct dates and months.

---

## DAX

### Measures

```DAX
order_count     = DISTINCTCOUNT(dim_orders[Order ID])
sum_revenue     = SUM(fact_order_details[revenue])
YTD Sales       = TOTALYTD(SUM(fact_order_details[revenue]), dim_calender[order_date])
Profit_margin   = DIVIDE(SUM(fact_order_details[Profit]), SUM(fact_order_details[Amount]))
min_target      = MIN(fact_sales_target[Target])
Average Profit  = CALCULATE(AVERAGE(fact_order_details[Profit]), dim_customers[State] = "Delhi")
```

### Calculated columns

```DAX
revenue        = [Amount] * [Quantity]
Category Type  = fact_order_details[Sub-Category] & " ," & RELATED(dim_category[category_name])
Sales Category = IF(fact_order_details[Amount] >= AVERAGE(fact_order_details[Amount]),
                    "Above Average", "Below Average")
```

## Tech stack

Power BI Desktop · Power Query (M) · DAX · Star-schema modelling
