# SQL Cheatsheet - Quick Reference

Tài liệu này chứa tất cả những điều bạn cần biết về SQL syntax.

---

## 📌 Table of Contents

1. [Basic SELECT](#basic-select)
2. [WHERE Clause](#where-clause)
3. [JOIN Operations](#join-operations)
4. [Aggregation](#aggregation)
5. [Window Functions](#window-functions)
6. [CTEs](#ctes)
7. [Advanced Topics](#advanced-topics)

---

## Basic SELECT

### Simple SELECT
```sql
SELECT column1, column2
FROM table_name;

-- Select all columns
SELECT * FROM table_name;

-- Select with alias
SELECT name AS customer_name, age AS years_old
FROM customers;
```

### SELECT DISTINCT
```sql
SELECT DISTINCT country
FROM customers;
```

### LIMIT & OFFSET
```sql
SELECT * FROM products
LIMIT 10;               -- First 10 rows

SELECT * FROM products
LIMIT 10 OFFSET 5;      -- 10 rows starting from 6th row

-- Alternative (MySQL)
SELECT * FROM products
LIMIT 5, 10;            -- 10 rows starting from 6th row
```

---

## WHERE Clause

### Comparison Operators
```sql
SELECT * FROM products WHERE price > 100;
SELECT * FROM products WHERE price >= 100;
SELECT * FROM products WHERE price < 100;
SELECT * FROM products WHERE price <= 100;
SELECT * FROM products WHERE price = 100;
SELECT * FROM products WHERE price != 100;     -- or <>
```

### Logical Operators
```sql
-- AND
SELECT * FROM products
WHERE price > 100 AND category = 'Electronics';

-- OR
SELECT * FROM products
WHERE price > 100 OR category = 'Electronics';

-- NOT
SELECT * FROM products
WHERE NOT category = 'Electronics';
```

### String Matching
```sql
-- LIKE
SELECT * FROM customers WHERE name LIKE 'John%';      -- Starts with John
SELECT * FROM customers WHERE name LIKE '%Smith';     -- Ends with Smith
SELECT * FROM customers WHERE name LIKE '%John%';     -- Contains John
SELECT * FROM customers WHERE name LIKE 'J_hn';       -- _ = single character

-- IN
SELECT * FROM customers
WHERE country IN ('USA', 'UK', 'Canada');

-- NOT IN
SELECT * FROM customers
WHERE country NOT IN ('USA', 'UK', 'Canada');
```

### Range Checking
```sql
-- BETWEEN
SELECT * FROM products
WHERE price BETWEEN 50 AND 200;

-- NOT BETWEEN
SELECT * FROM products
WHERE price NOT BETWEEN 50 AND 200;

-- NULL checking
SELECT * FROM customers WHERE phone IS NULL;
SELECT * FROM customers WHERE phone IS NOT NULL;
```

---

## ORDER BY

### Basic Sorting
```sql
-- Ascending (default)
SELECT * FROM products
ORDER BY price;

-- Descending
SELECT * FROM products
ORDER BY price DESC;

-- Multiple columns
SELECT * FROM orders
ORDER BY customer_id ASC, order_date DESC;

-- By column number (avoid in production)
SELECT name, price FROM products
ORDER BY 2 DESC;  -- Order by 2nd column (price)
```

---

## JOIN Operations

### INNER JOIN
```sql
SELECT c.name, o.order_id, o.total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- Shorthand: can omit INNER
SELECT c.name, o.order_id, o.total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

### LEFT JOIN (LEFT OUTER JOIN)
```sql
-- Include all from left table, matching from right
SELECT c.name, COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- Include all from right table, matching from left
SELECT c.name, o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

### FULL OUTER JOIN (Not in MySQL)
```sql
-- PostgreSQL, SQLite
SELECT c.name, o.order_id
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- MySQL alternative using UNION
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
UNION
SELECT c.name, o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

### CROSS JOIN
```sql
-- Cartesian product: every row from left × every row from right
SELECT c.name, p.name
FROM customers c
CROSS JOIN products p;
```

### Self JOIN
```sql
-- Join table with itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id;
```

---

## Aggregation

### Aggregate Functions
```sql
SELECT
  COUNT(*) as total_rows,        -- Count all rows
  COUNT(email) as emails,         -- Count non-NULL values
  SUM(amount) as total_amount,    -- Sum of values
  AVG(price) as average_price,    -- Average value
  MIN(price) as lowest_price,     -- Minimum value
  MAX(price) as highest_price     -- Maximum value
FROM orders;
```

### GROUP BY
```sql
SELECT category, COUNT(*) as product_count, AVG(price) as avg_price
FROM products
GROUP BY category;

-- Multiple columns
SELECT country, city, COUNT(*) as customer_count
FROM customers
GROUP BY country, city;
```

### HAVING
```sql
-- HAVING is like WHERE but for grouped results
SELECT category, COUNT(*) as product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 5;            -- Filter groups

-- HAVING with multiple conditions
SELECT category, SUM(price) as total_price
FROM products
GROUP BY category
HAVING SUM(price) > 1000 AND COUNT(*) >= 5;
```

### WHERE vs HAVING
```sql
-- WHERE: filters rows before grouping
SELECT category, COUNT(*) as product_count
FROM products
WHERE price > 100              -- Applied before GROUP BY
GROUP BY category;

-- HAVING: filters groups after grouping
SELECT category, COUNT(*) as product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 5;           -- Applied after GROUP BY
```

### DISTINCT with Aggregation
```sql
SELECT COUNT(DISTINCT country) as country_count
FROM customers;

SELECT category, COUNT(DISTINCT product_id)
FROM products
GROUP BY category;
```

---

## Window Functions

### ROW_NUMBER
```sql
-- Assigns unique number to each row
SELECT
  product_id, name, price,
  ROW_NUMBER() OVER (ORDER BY price DESC) as rank
FROM products;

-- With PARTITION BY
SELECT
  category, name, price,
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rank_in_category
FROM products;
```

### RANK & DENSE_RANK
```sql
-- RANK: skips numbers for ties
SELECT
  name, score,
  RANK() OVER (ORDER BY score DESC) as rank
FROM results;
-- Scores: 100, 90, 90, 80 → Ranks: 1, 2, 2, 4

-- DENSE_RANK: doesn't skip numbers
SELECT
  name, score,
  DENSE_RANK() OVER (ORDER BY score DESC) as rank
FROM results;
-- Scores: 100, 90, 90, 80 → Ranks: 1, 2, 2, 3
```

### LAG & LEAD
```sql
-- LAG: get previous row value
SELECT
  sale_date, amount,
  LAG(amount) OVER (ORDER BY sale_date) as previous_amount
FROM sales;

-- LEAD: get next row value
SELECT
  sale_date, amount,
  LEAD(amount) OVER (ORDER BY sale_date) as next_amount
FROM sales;

-- With offset and default
SELECT
  sale_date, amount,
  LAG(amount, 1, 0) OVER (ORDER BY sale_date) as prev_1_day,
  LAG(amount, 7, 0) OVER (ORDER BY sale_date) as prev_7_days
FROM sales;
```

### SUM/AVG OVER (Running Total)
```sql
-- Running total
SELECT
  sale_date, amount,
  SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM sales;

-- Running average (last 7 days)
SELECT
  sale_date, amount,
  AVG(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as avg_7days
FROM sales;
```

### Window Frame Options
```sql
-- ROWS BETWEEN clauses:
ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW     -- All rows up to current
ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- All rows
ORDER BY date ROWS BETWEEN 1 PRECEDING AND CURRENT ROW              -- Previous and current
ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING              -- Previous, current, next

-- RANGE (for numeric/date values):
ORDER BY date RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW
```

---

## CTEs (Common Table Expression)

### Basic CTE
```sql
WITH top_products AS (
  SELECT product_id, name, price
  FROM products
  WHERE price > 100
)
SELECT * FROM top_products;
```

### Multiple CTEs
```sql
WITH high_value_orders AS (
  SELECT order_id, customer_id, total
  FROM orders
  WHERE total > 1000
),
customer_summary AS (
  SELECT customer_id, COUNT(*) as order_count
  FROM high_value_orders
  GROUP BY customer_id
)
SELECT * FROM customer_summary;
```

### Recursive CTE (PostgreSQL, SQLite)
```sql
WITH RECURSIVE numbers AS (
  SELECT 1 as n
  UNION ALL
  SELECT n + 1 FROM numbers WHERE n < 10
)
SELECT * FROM numbers;

-- Hierarchical data
WITH RECURSIVE employee_hierarchy AS (
  SELECT employee_id, name, manager_id, 1 as level
  FROM employees
  WHERE manager_id IS NULL        -- Start with top managers
  UNION ALL
  SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
  FROM employees e
  JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy;
```

---

## Advanced Topics

### CASE WHEN
```sql
-- Simple CASE
SELECT
  product_id, name, price,
  CASE
    WHEN price < 50 THEN 'Cheap'
    WHEN price BETWEEN 50 AND 200 THEN 'Medium'
    WHEN price > 200 THEN 'Expensive'
    ELSE 'Unknown'
  END as price_category
FROM products;

-- Searched CASE
SELECT
  customer_id,
  CASE
    WHEN total_orders > 100 THEN 'VIP'
    WHEN total_orders > 50 THEN 'Gold'
    WHEN total_orders > 10 THEN 'Silver'
    ELSE 'Regular'
  END as customer_type
FROM customer_summary;
```

### Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM customers
WHERE customer_id IN (
  SELECT customer_id FROM orders
  GROUP BY customer_id
  HAVING COUNT(*) > 5
);

-- Subquery in FROM
SELECT avg_price FROM (
  SELECT AVG(price) as avg_price
  FROM products
  GROUP BY category
) category_avg;

-- Correlated subquery
SELECT name, price
FROM products p1
WHERE price > (
  SELECT AVG(price)
  FROM products p2
  WHERE p2.category = p1.category
);
```

### UNION / EXCEPT / INTERSECT
```sql
-- UNION: combine results, remove duplicates
SELECT name FROM customers
UNION
SELECT name FROM suppliers;

-- UNION ALL: combine results, keep duplicates
SELECT name FROM customers
UNION ALL
SELECT name FROM suppliers;

-- EXCEPT: in first query but not second
SELECT product_id FROM inventory
EXCEPT
SELECT product_id FROM sales;

-- INTERSECT: in both queries
SELECT customer_id FROM customers_2023
INTERSECT
SELECT customer_id FROM customers_2024;
```

### UPDATE / DELETE / INSERT

```sql
-- INSERT
INSERT INTO customers (name, email, country)
VALUES ('John Doe', 'john@email.com', 'USA');

-- INSERT from SELECT
INSERT INTO customers_backup
SELECT * FROM customers WHERE country = 'USA';

-- UPDATE
UPDATE customers
SET country = 'United States'
WHERE country = 'USA';

-- UPDATE from JOIN
UPDATE orders o
SET o.status = 'completed'
WHERE o.order_date < DATE_SUB(NOW(), INTERVAL 30 DAY);

-- DELETE
DELETE FROM customers
WHERE country = 'Unknown';

-- DELETE with JOIN
DELETE c
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;  -- Delete customers with no orders
```

---

## 🎯 Performance Tips

### Check Execution Plan
```sql
-- MySQL
EXPLAIN SELECT * FROM large_table WHERE id > 1000;

-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM large_table WHERE id > 1000;
```

### Common Optimization Techniques
```sql
-- 1. Use WHERE before GROUP BY
-- ✗ Bad
SELECT category, COUNT(*) FROM products
WHERE price > 100
GROUP BY category;

-- ✓ Good (same result, but sometimes faster with WHERE first)

-- 2. Use EXISTS instead of IN with subqueries
-- ✗ Slower
SELECT * FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);

-- ✓ Faster
SELECT * FROM customers
WHERE EXISTS (SELECT 1 FROM orders WHERE orders.customer_id = customers.customer_id);

-- 3. Avoid SELECT *
-- ✗ Bad
SELECT * FROM large_table;

-- ✓ Good
SELECT column1, column2 FROM large_table;

-- 4. Use proper JOINs
-- ✗ Slower (Cartesian product)
SELECT * FROM orders, order_items
WHERE orders.order_id = order_items.order_id;

-- ✓ Faster
SELECT * FROM orders
INNER JOIN order_items ON orders.order_id = order_items.order_id;
```

---

## Common Functions

### String Functions
```sql
UPPER(string)              -- Convert to uppercase
LOWER(string)              -- Convert to lowercase
LENGTH(string)             -- Length of string
SUBSTRING(string, 1, 3)    -- Extract substring
TRIM(string)               -- Remove leading/trailing spaces
REPLACE(string, 'old', 'new')  -- Replace text
CONCAT(str1, str2)         -- Concatenate strings
```

### Date Functions
```sql
NOW() / CURRENT_TIMESTAMP  -- Current date/time
DATE_FORMAT(date, '%Y-%m-%d')  -- Format date (MySQL)
DATE_TRUNC('month', date)      -- Truncate to month (PostgreSQL)
DATE_ADD(date, INTERVAL 1 DAY) -- Add days
DATEDIFF(date1, date2)         -- Days between
EXTRACT(MONTH FROM date)       -- Extract part
```

### Numeric Functions
```sql
ROUND(number, decimals)    -- Round to decimals
FLOOR(number)              -- Round down
CEIL(number)               -- Round up
ABS(number)                -- Absolute value
```

---

## Quick Syntax Checklist

```sql
-- SELECT structure
SELECT [DISTINCT] column1, column2
FROM table_name
[WHERE conditions]
[GROUP BY columns]
[HAVING conditions]
[ORDER BY column]
[LIMIT count]
[OFFSET offset];

-- JOIN structure
FROM table1
[INNER/LEFT/RIGHT/FULL] JOIN table2 ON table1.id = table2.id
[INNER/LEFT/RIGHT/FULL] JOIN table3 ON ...

-- Window function structure
[window_function(column)] OVER (
  [PARTITION BY column]
  [ORDER BY column]
  [ROWS/RANGE BETWEEN ...]
)

-- CTE structure
WITH cte_name AS (
  SELECT ...
)
SELECT ... FROM cte_name;
```

---

**Happy querying! 🚀**

*Keep this cheatsheet handy during practice!*
