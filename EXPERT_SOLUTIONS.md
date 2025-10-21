# Expert Level Solutions

Lời giải chi tiết cho 7 bài tập Expert level SQL.

---

## Expert 1: Median & Percentiles

```sql
-- PostgreSQL Solution
WITH product_stats AS (
  SELECT
    category,
    product_id,
    name,
    price,
    COUNT(*) OVER (PARTITION BY category) as category_count,
    AVG(price) OVER (PARTITION BY category) as avg_price,
    STDDEV_POP(price) OVER (PARTITION BY category) as stddev_price,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) OVER (PARTITION BY category) as median_price,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY price) OVER (PARTITION BY category) as p25,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY price) OVER (PARTITION BY category) as p75
  FROM products
)
SELECT
  category,
  name,
  price,
  ROUND(median_price::numeric, 2) as median_price,
  ROUND(p25::numeric, 2) as p25,
  ROUND(p75::numeric, 2) as p75,
  ROUND(((price - avg_price) / NULLIF(stddev_price, 0))::numeric, 2) as z_score
FROM product_stats
ORDER BY category, price DESC;
```

**MySQL Alternative** (không có PERCENTILE_CONT):
```sql
WITH product_stats AS (
  SELECT
    category,
    product_id,
    name,
    price,
    AVG(price) OVER (PARTITION BY category) as avg_price,
    STDDEV_POP(price) OVER (PARTITION BY category) as stddev_price
  FROM products
),
percentiles AS (
  SELECT
    category,
    @rownum:=@rownum+1 as rownum,
    price
  FROM (SELECT DISTINCT category, price FROM products) p,
       (SELECT @rownum:=0) init
  ORDER BY category, price
)
SELECT ps.*, ROUND((ps.price - ps.avg_price) / NULLIF(ps.stddev_price, 0), 2) as z_score
FROM product_stats ps
ORDER BY ps.category, ps.price DESC;
```

**Key Concepts**:
- PERCENTILE_CONT: Giá trị interpolated tại percentile
- STDDEV_POP: Population standard deviation
- Z-score formula: (value - mean) / stddev
- PARTITION BY: Tính independent stats per category

---

## Expert 2: Customer Lifetime Value & Churn Analysis

```sql
-- Complete RFM + Churn Analysis
WITH customer_metrics AS (
  SELECT
    c.customer_id,
    c.name,
    c.signup_date,
    SUM(o.total) as clv,
    COUNT(DISTINCT o.order_id) as frequency,
    DATEDIFF(DAY, MAX(o.order_date), '2024-10-20') as recency_days,
    MAX(o.order_date) as last_order_date
  FROM customers c
  LEFT JOIN orders o ON c.customer_id = o.customer_id
  GROUP BY c.customer_id, c.name, c.signup_date
),
rfm_ranking AS (
  SELECT
    *,
    NTILE(5) OVER (ORDER BY recency_days DESC) as r_rank,      -- Higher = better (lower recency)
    NTILE(5) OVER (ORDER BY frequency ASC) as f_rank,           -- Lower = better (higher frequency)
    NTILE(5) OVER (ORDER BY clv ASC) as m_rank                  -- Lower = better (higher monetary)
  FROM customer_metrics
),
churn_analysis AS (
  SELECT
    *,
    CASE
      WHEN recency_days > 90 THEN 'Yes'
      ELSE 'No'
    END as churn_risk,
    CONCAT(r_rank, f_rank, m_rank) as rfm_score
  FROM rfm_ranking
)
SELECT
  customer_id,
  name,
  COALESCE(clv, 0) as CLV,
  COALESCE(frequency, 0) as frequency,
  recency_days,
  churn_risk,
  rfm_score
FROM churn_analysis
ORDER BY clv DESC;
```

**Key Concepts**:
- CLV = Lifetime monetary value
- Frequency = Purchase count
- Recency = Days since last purchase
- NTILE(5): Rank into 5 buckets (quintiles)
- RFM Score: Concatenate ranks (higher = better customer)

---

## Expert 3: Cohort Analysis & Retention

```sql
-- Cohort Analysis Solution
WITH customer_cohort AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', signup_date) as cohort_month
  FROM customers
),
user_activity AS (
  SELECT
    cc.customer_id,
    cc.cohort_month,
    DATE_TRUNC('month', o.order_date) as order_month,
    (DATE_PART('year', DATE_TRUNC('month', o.order_date)) - DATE_PART('year', cc.cohort_month)) * 12 +
    (DATE_PART('month', DATE_TRUNC('month', o.order_date)) - DATE_PART('month', cc.cohort_month)) as month_number
  FROM customer_cohort cc
  LEFT JOIN orders o ON cc.customer_id = o.customer_id
),
cohort_size AS (
  SELECT
    cohort_month,
    COUNT(DISTINCT customer_id) as cohort_size
  FROM customer_cohort
  GROUP BY cohort_month
),
retention_table AS (
  SELECT
    ua.cohort_month,
    ua.month_number,
    COUNT(DISTINCT ua.customer_id) as retained_customers
  FROM user_activity ua
  WHERE ua.month_number >= 0
  GROUP BY ua.cohort_month, ua.month_number
)
SELECT
  rt.cohort_month,
  cs.cohort_size,
  CASE WHEN rt.month_number = 0 THEN CONCAT(ROUND(100.0 * rt.retained_customers / cs.cohort_size, 1), '%')
       WHEN rt.month_number = 1 THEN CONCAT(ROUND(100.0 * rt.retained_customers / cs.cohort_size, 1), '%')
       -- Add more columns for month 2, 3, etc.
  END as retention
FROM retention_table rt
JOIN cohort_size cs ON rt.cohort_month = cs.cohort_month
ORDER BY rt.cohort_month, rt.month_number;
```

**Pivot Version** (PostgreSQL):
```sql
-- Cross-tabulation for better readability
SELECT
  cohort_month,
  ROUND(100.0 * MAX(CASE WHEN month_number = 0 THEN retained_customers END) / cohort_size, 1) as m0,
  ROUND(100.0 * MAX(CASE WHEN month_number = 1 THEN retained_customers END) / cohort_size, 1) as m1,
  ROUND(100.0 * MAX(CASE WHEN month_number = 2 THEN retained_customers END) / cohort_size, 1) as m2
FROM (
  -- CTE queries above
);
```

**Key Concepts**:
- Cohort = GROUP based on signup period
- Month index = DATEDIFF in months
- Retention % = Active users in month X / Cohort size

---

## Expert 4: Moving Averages & Trend Analysis

```sql
-- Moving Averages & Anomaly Detection
WITH sales_with_ma AS (
  SELECT
    date,
    sales,
    ROUND(AVG(sales) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) as ma7,
    ROUND(AVG(sales) OVER (ORDER BY date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW), 2) as ma30,
    LAG(sales) OVER (ORDER BY date) as prev_sales,
    ROUND(100.0 * (sales - LAG(sales) OVER (ORDER BY date)) / LAG(sales) OVER (ORDER BY date), 1) as growth_rate,
    STDDEV_POP(sales) OVER (ORDER BY date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) as stddev_30
  FROM daily_sales
),
with_trend AS (
  SELECT
    date,
    sales,
    ma7,
    ma30,
    prev_sales,
    growth_rate,
    CASE
      WHEN growth_rate > 2 THEN 'Up'
      WHEN growth_rate < -2 THEN 'Down'
      ELSE 'Flat'
    END as trend,
    CASE
      WHEN ma7 IS NOT NULL AND stddev_30 IS NOT NULL
        AND ABS(sales - ma7) > 2 * stddev_30 THEN 'Yes'
      ELSE 'No'
    END as is_anomaly
  FROM sales_with_ma
)
SELECT * FROM with_trend
ORDER BY date;
```

**Key Concepts**:
- MA7: 7-day moving average
- MA30: 30-day moving average
- Growth rate: % change from previous day
- Anomaly: Z-score > 2 (beyond 2 standard deviations)
- ROWS BETWEEN: Define window frame

---

## Expert 5: Hierarchical Data & Path Finding

```sql
-- Recursive CTE for Manager Chain
WITH RECURSIVE employee_hierarchy AS (
  -- Base case: top level employees (no manager)
  SELECT
    employee_id,
    name,
    manager_id,
    salary,
    1 as level,
    CAST(name AS VARCHAR(1000)) as manager_chain
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive case: join to find subordinates
  SELECT
    e.employee_id,
    e.name,
    e.manager_id,
    e.salary,
    eh.level + 1,
    CONCAT(eh.manager_chain, ' > ', e.name)
  FROM employees e
  INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
),
subordinate_count AS (
  SELECT
    manager_id,
    COUNT(*) as subordinates
  FROM employees
  WHERE manager_id IS NOT NULL
  GROUP BY manager_id

  UNION ALL

  SELECT
    employee_id,
    0
  FROM employees
  WHERE employee_id NOT IN (SELECT DISTINCT manager_id FROM employees WHERE manager_id IS NOT NULL)
)
SELECT
  eh.employee_id,
  eh.name,
  eh.manager_chain,
  eh.level,
  COALESCE(sc.subordinates, 0) as subordinates,
  eh.salary
FROM employee_hierarchy eh
LEFT JOIN subordinate_count sc ON eh.employee_id = sc.manager_id
ORDER BY eh.level, eh.employee_id;
```

**Key Concepts**:
- Base case: Start with top-level (no manager)
- Recursive case: Find children
- Level tracking: Increment per recursion
- Path reconstruction: CONCAT strings
- Subordinate count: COUNT from base query

---

## Expert 6: Market Basket Analysis

```sql
-- Frequently Bought Together Analysis
WITH product_pairs AS (
  SELECT
    oi1.product_id as product_a_id,
    oi2.product_id as product_b_id,
    COUNT(DISTINCT oi1.order_id) as transactions_both
  FROM order_items oi1
  JOIN order_items oi2 ON oi1.order_id = oi2.order_id
    AND oi1.product_id < oi2.product_id  -- Avoid duplicates and self-pairs
  GROUP BY oi1.product_id, oi2.product_id
),
product_counts AS (
  SELECT
    product_id,
    COUNT(DISTINCT order_id) as product_orders
  FROM order_items
  GROUP BY product_id
),
total_orders AS (
  SELECT COUNT(DISTINCT order_id) as total FROM orders
)
SELECT
  p1.name as product_a,
  p2.name as product_b,
  pp.transactions_both,
  CONCAT(ROUND(100.0 * pp.transactions_both / to.total, 1), '%') as support,
  CONCAT(ROUND(100.0 * pp.transactions_both / pc_a.product_orders, 1), '%') as confidence_a_to_b,
  ROUND((pp.transactions_both / to.total) / ((pc_a.product_orders / to.total) * (pc_b.product_orders / to.total)), 2) as lift
FROM product_pairs pp
JOIN products p1 ON pp.product_a_id = p1.product_id
JOIN products p2 ON pp.product_b_id = p2.product_id
JOIN product_counts pc_a ON pp.product_a_id = pc_a.product_id
JOIN product_counts pc_b ON pp.product_b_id = pc_b.product_id
CROSS JOIN total_orders to
WHERE pp.transactions_both >= 1  -- Filter for significance
ORDER BY lift DESC, support DESC;
```

**Formulas**:
- Support(A,B) = Count(A ∩ B) / Total Orders
- Confidence(A→B) = Count(A ∩ B) / Count(A)
- Lift(A→B) = Confidence(A→B) / Support(B)
- Lift > 1: Products are positively correlated

**Key Concepts**:
- Self-join: Compare products within same order
- oi1.product_id < oi2.product_id: Avoid duplicates
- CROSS JOIN: Get total for denominator

---

## Expert 7: Time-series Decomposition

```sql
-- Seasonal Decomposition
WITH ranked_sales AS (
  SELECT
    date,
    sales_amount as actual,
    ROW_NUMBER() OVER (ORDER BY date) as rn
  FROM sales
),
trend_component AS (
  SELECT
    date,
    actual,
    ROUND(AVG(actual) OVER (
      ORDER BY rn ROWS BETWEEN 11 PRECEDING AND 12 FOLLOWING
    ), 2) as trend  -- 12-month centered MA
  FROM ranked_sales
),
seasonal_component AS (
  SELECT
    date,
    actual,
    trend,
    CASE
      WHEN trend IS NOT NULL THEN ROUND(actual::decimal / trend, 3)
      ELSE NULL
    END as seasonal_factor,
    ROUND(actual - COALESCE(trend, 0), 2) as residual
  FROM trend_component
),
seasonal_indices AS (
  SELECT
    EXTRACT(MONTH FROM date) as month,
    AVG(seasonal_factor) as avg_seasonal_factor
  FROM seasonal_component
  WHERE seasonal_factor IS NOT NULL
  GROUP BY EXTRACT(MONTH FROM date)
)
SELECT
  s.date,
  s.actual,
  s.trend,
  si.avg_seasonal_factor as seasonal_factor,
  s.residual,
  ROUND((s.trend * si.avg_seasonal_factor), 2) as forecast_next_month
FROM seasonal_component s
LEFT JOIN seasonal_indices si ON EXTRACT(MONTH FROM s.date) = si.month
ORDER BY s.date;
```

**Time Series Components**:
- **Trend**: Long-term direction (12-month MA)
- **Seasonal**: Repeating patterns per period
- **Residual**: Random variation
- **Decomposition**: Actual = Trend × Seasonal + Residual

**Key Concepts**:
- Centered moving average: ROWS BETWEEN 11 PRECEDING AND 12 FOLLOWING
- Seasonal factor: Actual / Trend
- Monthly indices: AVG seasonal factor per month
- Forecast: Trend × Average seasonal factor

---

## 🎓 Performance Optimization Tips

### 1. Indexing Strategy
```sql
-- Create indexes for frequently joined/filtered columns
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- Composite index for common query patterns
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```

### 2. Query Optimization Checklist
```sql
-- Use EXPLAIN to analyze query performance
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_date > '2024-01-01';

-- Look for:
-- - Full table scan (Seq Scan) → need index
-- - Index Scan → good
-- - Nested Loop Join vs Hash Join
-- - Execution time and planning time
```

### 3. CTE vs Subquery
```sql
-- CTE: More readable, may or may not be optimized
WITH user_stats AS (
  SELECT customer_id, COUNT(*) as orders
  FROM orders
  GROUP BY customer_id
)
SELECT * FROM user_stats WHERE orders > 5;

-- Subquery: Can be optimized by query planner
SELECT * FROM (
  SELECT customer_id, COUNT(*) as orders
  FROM orders
  GROUP BY customer_id
) stats
WHERE orders > 5;
```

### 4. Window Function Optimization
```sql
-- Use PARTITION BY to limit window scope
-- Instead of: SUM(amount) OVER (ORDER BY date)
-- Use: SUM(amount) OVER (PARTITION BY customer_id ORDER BY date)

-- This reduces computation and may enable better indexing
```

---

## 📈 Business Metrics Dictionary

- **CLV (Customer Lifetime Value)**: Total revenue per customer
- **RFM**: Recency, Frequency, Monetary value
- **Churn Rate**: % customers stopped buying
- **Retention Rate**: % customers continue buying
- **Cohort**: Group of customers with common characteristics
- **Support**: % of transactions with both products (Market Basket)
- **Confidence**: Probability of B given A (Market Basket)
- **Lift**: Strength of association between A and B
- **Moving Average**: Trend smoothing
- **Anomaly**: Outlier detection

---

**Master these topics and you're ready for Senior/Staff engineer interviews! 🚀**
