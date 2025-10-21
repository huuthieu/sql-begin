# SQL Expert Problems (7 bài) - Advanced Interview Level

Những bài tập này tương đương với các kỳ phỏng vấn tại Google, Amazon, Facebook, Microsoft.

---

## Problem 1️⃣: Median & Percentiles

**Difficulty**: ⭐⭐⭐⭐ Expert

**Topics**: WINDOW FUNCTIONS, PERCENTILE_CONT, PERCENTILE_DISC, MEDIAN

**Description**:
Tính median, 25th percentile, 75th percentile của giá sản phẩm theo từng category. Đồng thời tính Z-score (số độ lệch chuẩn từ mean)

**Schema**:
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2),
    category VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO products VALUES
(1, 'Mouse', 25.00, 'Electronics'),
(2, 'Keyboard', 75.00, 'Electronics'),
(3, 'Monitor', 300.00, 'Electronics'),
(4, 'Laptop', 1200.00, 'Electronics'),
(5, 'USB Cable', 10.00, 'Electronics'),
(6, 'Desk', 150.00, 'Furniture'),
(7, 'Chair', 100.00, 'Furniture'),
(8, 'Table', 200.00, 'Furniture'),
(9, 'Lamp', 50.00, 'Furniture'),
(10, 'Shelf', 75.00, 'Furniture');
```

**Expected Output** (PostgreSQL):
```
category     | product_name | price | median_price | p25 | p75 | z_score
Electronics  | Mouse        | 25.00 | 75.00        | 25  | 300 | -1.23
Electronics  | Keyboard     | 75.00 | 75.00        | 25  | 300 | -0.45
Electronics  | Monitor      | 300.00| 75.00        | 25  | 300 | 1.12
Electronics  | Laptop       | 1200.00| 75.00       | 25  | 300 | 3.89
Electronics  | USB Cable    | 10.00 | 75.00        | 25  | 300 | -1.56
Furniture    | Desk         | 150.00| 87.50        | 50  | 150 | 1.02
Furniture    | Chair        | 100.00| 87.50        | 50  | 150 | 0.41
Furniture    | Table        | 200.00| 87.50        | 50  | 150 | 1.63
Furniture    | Lamp         | 50.00 | 87.50        | 50  | 150 | -0.85
Furniture    | Shelf        | 75.00 | 87.50        | 50  | 150 | -0.41
```

**Hints**:
- Z-score = (value - mean) / stddev
- Dùng PERCENTILE_CONT (PostgreSQL) hoặc tính manually
- Dùng STDDEV hoặc STDDEV_POP

---

## Problem 2️⃣: Customer Lifetime Value (CLV) & Churn Analysis

**Difficulty**: ⭐⭐⭐⭐ Expert

**Topics**: MULTIPLE CTEs, WINDOW FUNCTIONS, DATE CALCULATIONS, COMPLEX JOINS

**Description**:
Tính Customer Lifetime Value (CLV), frequency (bao nhiêu lần mua), recency (bao lâu kể từ lần mua cuối), RFM score, và xác định những customers có nguy cơ churn (không mua trong 90 ngày)

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    signup_date DATE
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total DECIMAL(10, 2),
    FOREIGN KEY(customer_id) REFERENCES customers(customer_id)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe', '2023-01-01'),
(2, 'Jane Smith', '2023-02-01'),
(3, 'Bob Johnson', '2023-03-01'),
(4, 'Alice Williams', '2023-04-01'),
(5, 'Charlie Brown', '2023-05-01');

INSERT INTO orders VALUES
(1, 1, '2024-01-05', 500.00),
(2, 1, '2024-01-15', 750.00),
(3, 1, '2024-08-01', 300.00),
(4, 2, '2024-02-10', 1000.00),
(5, 2, '2024-02-20', 800.00),
(6, 2, '2024-03-05', 600.00),
(7, 3, '2024-06-15', 500.00),
(8, 3, '2024-07-20', 400.00),
(9, 4, '2024-03-10', 200.00),
(10, 5, '2024-01-01', 100.00);

-- Assume today is 2024-10-20
```

**Expected Output**:
```
customer_id | name           | CLV    | frequency | recency_days | churn_risk | rfm_score
1           | John Doe       | 1550.00| 3         | 80           | No         | 533
2           | Jane Smith     | 2400.00| 3         | 235          | No         | 542
3           | Bob Johnson    | 900.00 | 2         | 92           | Yes        | 322
4           | Alice Williams | 200.00 | 1         | 225          | Yes        | 111
5           | Charlie Brown  | 100.00 | 1         | 293          | Yes        | 111
```

**Hints**:
- CLV = SUM(total) per customer
- Frequency = COUNT(distinct orders)
- Recency = DATEDIFF(TODAY, MAX(order_date))
- RFM Score: Rank customers 1-5 pada Recency, Frequency, Monetary
- Churn Risk: recency_days > 90

---

## Problem 3️⃣: Cohort Analysis & Retention Rate

**Difficulty**: ⭐⭐⭐⭐ Expert

**Topics**: MULTIPLE CTEs, WINDOW FUNCTIONS, DATE ARITHMETIC, PIVOT-like Logic

**Description**:
Tạo cohort analysis table để xem retention rate của customers theo từng tháng đăng ký. Hiển thị % customers quay lại vào tháng thứ 1, 2, 3... sau khi đăng ký

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    signup_date DATE
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, '2024-01-15'),
(2, '2024-01-20'),
(3, '2024-01-25'),
(4, '2024-02-10'),
(5, '2024-02-15'),
(6, '2024-03-05');

INSERT INTO orders VALUES
(1, 1, '2024-01-15', 100),
(2, 1, '2024-02-10', 150),
(3, 1, '2024-03-05', 200),
(4, 2, '2024-01-25', 100),
(5, 2, '2024-02-15', 150),
(6, 3, '2024-02-01', 100),
(7, 4, '2024-02-15', 200),
(8, 4, '2024-03-10', 250),
(9, 5, '2024-02-20', 100),
(10, 6, '2024-03-10', 150);
```

**Expected Output**:
```
cohort_month | cohort_size | month_0 | month_1 | month_2 | month_3
2024-01      | 3           | 100%    | 67%     | 33%     | 0%
2024-02      | 2           | 100%    | 100%    | 50%     | NULL
2024-03      | 1           | 100%    | 33%     | NULL    | NULL
```

**Hints**:
- Cohort = DATE_TRUNC('month', signup_date)
- Month index = DATEDIFF(month, signup_date, order_date)
- Group by cohort_month, month_index
- Tính % = COUNT(distinct customer) in month X / cohort_size

---

## Problem 4️⃣: Moving Average & Trend Analysis

**Difficulty**: ⭐⭐⭐⭐ Expert

**Topics**: WINDOW FUNCTIONS, ROWS BETWEEN, LAG/LEAD, MOVING AVERAGES

**Description**:
Tính 7-day moving average, 30-day moving average, trend (up/down/flat) so với trước, growth rate, và anomaly detection (nếu giá trị lệch > 2 standard deviations từ moving average)

**Schema**:
```sql
CREATE TABLE daily_sales (
    date DATE PRIMARY KEY,
    sales DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO daily_sales VALUES
('2024-01-01', 1000),
('2024-01-02', 1100),
('2024-01-03', 1050),
('2024-01-04', 1200),
('2024-01-05', 1150),
('2024-01-06', 1300),
('2024-01-07', 1250),
('2024-01-08', 1400),
('2024-01-09', 1350),
('2024-01-10', 5000),  -- Anomaly
('2024-01-11', 1450),
('2024-01-12', 1500);
```

**Expected Output**:
```
date       | sales | ma7   | ma30  | prev_sales | growth_rate | trend     | is_anomaly
2024-01-01 | 1000  | NULL  | NULL  | NULL       | NULL        | NULL      | No
2024-01-02 | 1100  | NULL  | NULL  | 1000       | 10.0%       | Up        | No
2024-01-03 | 1050  | NULL  | NULL  | 1100       | -4.5%       | Down      | No
2024-01-04 | 1200  | NULL  | NULL  | 1050       | 14.3%       | Up        | No
2024-01-05 | 1150  | NULL  | NULL  | 1200       | -4.2%       | Down      | No
2024-01-06 | 1300  | NULL  | NULL  | 1150       | 13.0%       | Up        | No
2024-01-07 | 1250  | 1150  | NULL  | 1300       | -3.8%       | Down      | No
2024-01-08 | 1400  | 1221  | NULL  | 1250       | 12.0%       | Up        | No
2024-01-09 | 1350  | 1271  | NULL  | 1400       | -3.6%       | Down      | No
2024-01-10 | 5000  | 1960  | NULL  | 1350       | 270.4%      | Up        | Yes
2024-01-11 | 1450  | 2207  | NULL  | 5000       | -71.0%      | Down      | Yes
2024-01-12 | 1500  | 2243  | NULL  | 1450       | 3.4%        | Up        | No
```

**Hints**:
- MA7 = AVG(sales) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
- Growth rate = (sales - prev_sales) / prev_sales * 100
- Trend = "Up" if growth_rate > 2%, "Down" if < -2%, "Flat" otherwise
- Anomaly: ABS(sales - ma7) > 2 * STDDEV(sales)

---

## Problem 5️⃣: SQL Anti-pattern: Hierarchical Data & Path Finding

**Difficulty**: ⭐⭐⭐⭐⭐ Expert

**Topics**: RECURSIVE CTE, TREE TRAVERSAL, PATH RECONSTRUCTION

**Description**:
Tìm manager chain (path từ employee đến top manager), tính depth level, tính số lượng subordinates, tìm sibling relationships

**Schema**:
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    salary DECIMAL(10, 2),
    department VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO employees VALUES
(1, 'CEO', NULL, 500000, 'Executive'),
(2, 'VP Sales', 1, 200000, 'Sales'),
(3, 'VP Engineering', 1, 220000, 'Engineering'),
(4, 'Sales Manager', 2, 100000, 'Sales'),
(5, 'Sales Rep 1', 4, 60000, 'Sales'),
(6, 'Sales Rep 2', 4, 60000, 'Sales'),
(7, 'Eng Manager', 3, 110000, 'Engineering'),
(8, 'Senior Engineer', 7, 90000, 'Engineering'),
(9, 'Junior Engineer', 7, 70000, 'Engineering');
```

**Expected Output**:
```
employee_id | name            | manager_chain        | level | subordinates | salary
1           | CEO             | CEO                  | 0     | 8            | 500000
2           | VP Sales       | CEO > VP Sales       | 1     | 3            | 200000
3           | VP Engineering | CEO > VP Engineering | 1     | 3            | 220000
4           | Sales Manager  | CEO > VP Sales > ... | 2     | 2            | 100000
5           | Sales Rep 1    | CEO > VP Sales > ... | 3     | 0            | 60000
6           | Sales Rep 2    | CEO > VP Sales > ... | 3     | 0            | 60000
7           | Eng Manager    | CEO > VP Eng > ...   | 2     | 2            | 110000
8           | Senior Engineer| CEO > VP Eng > ...   | 3     | 0            | 90000
9           | Junior Engineer| CEO > VP Eng > ...   | 3     | 0            | 70000
```

**Hints**:
- Dùng recursive CTE để build manager chain
- Tính depth từ recursion level
- Dùng LATERAL hoặc subquery để count subordinates

---

## Problem 6️⃣: E-commerce: Product Recommendations & Basket Analysis

**Difficulty**: ⭐⭐⭐⭐⭐ Expert

**Topics**: COMPLEX JOINS, WINDOW FUNCTIONS, STATISTICAL ANALYSIS

**Description**:
Tìm frequently bought together products (market basket analysis). Tính support (% đơn hàng có cả 2 sản phẩm), confidence (nếu mua A, khả năng mua B), lift (độ liên kết so với random)

**Schema**:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO orders VALUES
(1, 1, '2024-01-01'),
(2, 2, '2024-01-02'),
(3, 3, '2024-01-03'),
(4, 1, '2024-01-04'),
(5, 2, '2024-01-05'),
(6, 4, '2024-01-06');

INSERT INTO products VALUES
(1, 'Laptop', 'Electronics'),
(2, 'Mouse', 'Electronics'),
(3, 'Keyboard', 'Electronics'),
(4, 'Monitor', 'Electronics'),
(5, 'Desk', 'Furniture');

INSERT INTO order_items VALUES
(1, 1, 1, 1), -- Order 1: Laptop
(2, 1, 2, 1), -- Order 1: Mouse
(3, 2, 1, 1), -- Order 2: Laptop
(4, 2, 4, 1), -- Order 2: Monitor
(5, 3, 2, 1), -- Order 3: Mouse
(6, 3, 3, 1), -- Order 3: Keyboard
(7, 4, 1, 1), -- Order 4: Laptop
(8, 4, 2, 1), -- Order 4: Mouse
(9, 5, 2, 1), -- Order 5: Mouse
(10, 5, 3, 1), -- Order 5: Keyboard
(11, 6, 3, 1); -- Order 6: Keyboard
```

**Expected Output**:
```
product_a  | product_b  | transactions_both | support | confidence_a_to_b | lift
Laptop     | Mouse      | 2                 | 33.3%   | 66.7%             | 2.0
Mouse      | Keyboard   | 1                 | 16.7%   | 33.3%             | 1.25
Laptop     | Monitor    | 1                 | 16.7%   | 33.3%             | 2.0
Mouse      | Laptop     | 2                 | 33.3%   | 50.0%             | 1.67
Keyboard   | Mouse      | 1                 | 16.7%   | 25.0%             | 0.8
```

**Hints**:
- Support(A,B) = Count(orders with both A and B) / Total orders
- Confidence(A→B) = Count(orders with both) / Count(orders with A)
- Lift(A→B) = Confidence(A→B) / Support(B)
- Self-join order_items với chính nó

---

## Problem 7️⃣: Time-series Forecasting: Season Decomposition

**Difficulty**: ⭐⭐⭐⭐⭐ Expert

**Topics**: WINDOW FUNCTIONS, COMPLEX CALCULATIONS, MOVING AVERAGES

**Description**:
Tách time series data thành trend, seasonality, residual components. Tính seasonal indices theo tháng/quarter, forecast ngày mai, tính MAPE error

**Schema**:
```sql
CREATE TABLE sales (
    date DATE PRIMARY KEY,
    sales_amount DECIMAL(10, 2)
);
```

**Sample Data** (ít nhất 24 điểm dữ liệu cho 2 năm):
```sql
INSERT INTO sales VALUES
('2023-01-01', 1000), ('2023-02-01', 900),  ('2023-03-01', 1200),
('2023-04-01', 1100), ('2023-05-01', 1300), ('2023-06-01', 1200),
('2023-07-01', 1500), ('2023-08-01', 1400), ('2023-09-01', 1600),
('2023-10-01', 1400), ('2023-11-01', 1800), ('2023-12-01', 2000),
('2024-01-01', 1100), ('2024-02-01', 1000), ('2024-03-01', 1300),
('2024-04-01', 1200), ('2024-05-01', 1400), ('2024-06-01', 1300),
('2024-07-01', 1600), ('2024-08-01', 1500), ('2024-09-01', 1700),
('2024-10-01', 1500), ('2024-11-01', 1900), ('2024-12-01', 2100);
```

**Expected Output** (simplified):
```
date       | actual | trend | seasonal_factor | residual | forecast_next_month
2023-01-01 | 1000   | 950   | 1.05            | 50       | NULL
2023-02-01 | 900    | 1000  | 0.90            | -100     | 1050
...
2024-12-01 | 2100   | 1950  | 1.08            | 150      | 2150
```

**Hints**:
- Trend = 12-month (hoặc appropriate period) moving average
- Seasonal factor = Actual / Trend
- Residual = Actual - Trend
- Forecast = Trend * avg(seasonal_factor) per month
- Cần CASE WHEN để group theo month

---

## 🏆 Expert Level Tips

1. **Recursive CTE**: Sử dụng cho hierarchical data, tree traversal
2. **Statistical Calculations**: Mean, Median, Percentiles, Stddev, Variance
3. **Window Function Optimization**: PARTITION BY, ORDER BY, ROWS BETWEEN
4. **Performance Tuning**: Subqueries vs JOINs, Index optimization
5. **Complex Business Logic**: RFM, Cohort, Market Basket Analysis
6. **Time-series Analysis**: Moving averages, seasonal decomposition
7. **Data Science Concepts**: Lift, Support, Confidence, Anomaly Detection

---

## 📊 Interview Tips cho Expert Level

- **Clarity**: Giải thích approach trước khi code
- **Edge Cases**: Xử lý NULL, empty results, edge dates
- **Performance**: Đề cập đến indexing, query optimization
- **Real-world**: Liên kết với business metrics
- **Scalability**: Cách query hoạt động với millions of rows

---

**You are now at the professional SQL level! 🚀**

*Hoàn thành những bài này, bạn đã sẵn sàng cho senior positions!*
