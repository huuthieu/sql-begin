# SQL Practice Solutions

Tài liệu này chứa các giải pháp cho tất cả 20 bài tập SQL. Hãy cố gắng giải quyết bài toán trước khi xem lời giải! 🚀

---

# EASY Solutions

## Easy 1: Select All Customers

```sql
SELECT *
FROM customers
WHERE country = 'USA';
```

**Giải thích**:
- SELECT * lấy tất cả cột
- WHERE country = 'USA' lọc chỉ khách từ USA

---

## Easy 2: Order by Price

```sql
SELECT *
FROM products
ORDER BY price DESC
LIMIT 5;
```

**Giải thích**:
- ORDER BY price DESC sắp xếp từ cao xuống thấp
- LIMIT 5 lấy 5 bản ghi đầu tiên

---

## Easy 3: Count Records

```sql
SELECT country, COUNT(*) as count
FROM customers
GROUP BY country
HAVING COUNT(*) >= 2
ORDER BY count DESC;
```

**Giải thích**:
- GROUP BY country nhóm theo quốc gia
- COUNT(*) đếm số lượng
- HAVING COUNT(*) >= 2 lọc chỉ nhóm có >= 2
- ORDER BY count DESC sắp xếp theo số lượng giảm dần

---

## Easy 4: Simple INNER JOIN

```sql
SELECT c.name as customer_name, p.name as product_name
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN products p ON o.product_id = p.product_id
ORDER BY c.name;
```

**Giải thích**:
- INNER JOIN kết hợp dữ liệu từ 3 bảng
- ON điều kiện kết hợp
- ORDER BY c.name sắp xếp theo tên khách

---

## Easy 5: DISTINCT Values

```sql
SELECT DISTINCT category
FROM products
ORDER BY category;
```

**Giải thích**:
- DISTINCT loại bỏ các giá trị trùng lặp
- Chỉ hiển thị từng category một lần

---

## Easy 6: SUM with GROUP BY

```sql
SELECT c.customer_id, c.name, SUM(o.quantity * o.price) as total_revenue
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
ORDER BY total_revenue DESC;
```

**Giải thích**:
- JOIN kết hợp customers với orders
- SUM(quantity * price) tính tổng doanh thu
- GROUP BY nhóm theo từng khách
- ORDER BY sắp xếp theo doanh thu giảm dần

---

## Easy 7: WHERE with BETWEEN

```sql
SELECT *
FROM products
WHERE price BETWEEN 50 AND 200
ORDER BY price;
```

**Giải thích**:
- BETWEEN price >= 50 AND price <= 200
- ORDER BY price sắp xếp theo giá tăng dần

---

# MEDIUM Solutions

## Medium 1: Multiple JOINs

```sql
SELECT c.name as customer_name, p.name as product_name, oi.quantity, oi.price
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
ORDER BY c.name;
```

**Giải thích**:
- Kết hợp 4 bảng lại với nhau
- Mỗi JOIN có ON clause riêng
- ORDER BY để sắp xếp kết quả

---

## Medium 2: LEFT JOIN with Aggregation

```sql
SELECT c.customer_id, c.name, COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
ORDER BY order_count DESC, c.name;
```

**Giải thích**:
- LEFT JOIN bao gồm cả khách chưa có đơn hàng
- COUNT(o.order_id) đếm số đơn (NULL được coi là 0)
- GROUP BY nhóm theo khách
- ORDER BY sắp xếp

---

## Medium 3: Subquery in WHERE

```sql
WITH customer_spending AS (
  SELECT customer_id, SUM(total) as total_spending
  FROM orders
  GROUP BY customer_id
),
avg_spending AS (
  SELECT AVG(total_spending) as avg_total
  FROM customer_spending
)
SELECT c.customer_id, c.name, cs.total_spending
FROM customers c
JOIN customer_spending cs ON c.customer_id = cs.customer_id
CROSS JOIN avg_spending
WHERE cs.total_spending > avg_spending.avg_total
ORDER BY cs.total_spending DESC;
```

**Giải thích**:
- Subquery tính tổng tiêu tiền theo khách
- Subquery khác tính giá trị trung bình
- WHERE lọc khách > trung bình
- Cách khác không dùng CTE:

```sql
SELECT c.customer_id, c.name, SUM(o.total) as total_spending
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
HAVING SUM(o.total) > (
  SELECT AVG(total_sum)
  FROM (
    SELECT SUM(total) as total_sum
    FROM orders
    GROUP BY customer_id
  ) subq
)
ORDER BY total_spending DESC;
```

---

## Medium 4: CASE WHEN

```sql
SELECT product_id, name, price,
  CASE
    WHEN price < 100 THEN 'Rẻ'
    WHEN price >= 100 AND price <= 500 THEN 'Trung bình'
    WHEN price > 500 THEN 'Đắt'
  END as price_category
FROM products
ORDER BY price DESC;
```

**Giải thích**:
- CASE WHEN để phân loại theo điều kiện
- ELSE có thể được thêm vào nếu cần
- ORDER BY để sắp xếp

---

## Medium 5: HAVING Clause

```sql
SELECT category, SUM(price) as total_value, COUNT(*) as product_count
FROM products
GROUP BY category
HAVING SUM(price) > 500 AND COUNT(*) >= 2
ORDER BY total_value DESC;
```

**Giải thích**:
- GROUP BY category nhóm theo loại
- HAVING lọc nhóm theo điều kiện (khác WHERE)
- WHERE dùng trước GROUP BY, HAVING dùng sau

---

## Medium 6: Self JOIN

```sql
SELECT e1.name as employee1, e2.name as employee2, e1.department_id
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id
  AND e1.employee_id < e2.employee_id
ORDER BY e1.department_id, e1.name;
```

**Giải thích**:
- Self JOIN kết hợp bảng với chính nó
- e1.employee_id < e2.employee_id tránh lặp lại cặp
- ORDER BY sắp xếp kết quả

---

## Medium 7: UNION

```sql
SELECT name, type FROM customers
UNION
SELECT name, type FROM suppliers
ORDER BY name;
```

**Giải thích**:
- UNION kết hợp kết quả từ 2 queries
- Tự động loại bỏ duplicates
- Dùng UNION ALL nếu muốn giữ duplicates
- Số cột và kiểu dữ liệu phải giống nhau

---

## Medium 8: Correlated Subquery

```sql
SELECT p1.product_id, p1.name, p1.price, p1.category
FROM products p1
WHERE p1.price > (
  SELECT AVG(p2.price)
  FROM products p2
  WHERE p2.category = p1.category
)
ORDER BY p1.category, p1.price DESC;
```

**Giải thích**:
- Correlated subquery tham chiếu dữ liệu từ outer query
- Subquery thực thi cho mỗi hàng của outer query
- Performance có thể yếu, xem xét dùng window function

**Alternative (cách tốt hơn)**:
```sql
SELECT p.product_id, p.name, p.price, p.category
FROM products p
JOIN (
  SELECT category, AVG(price) as avg_price
  FROM products
  GROUP BY category
) cat_avg ON p.category = cat_avg.category
WHERE p.price > cat_avg.avg_price
ORDER BY p.category, p.price DESC;
```

---

# HARD Solutions

## Hard 1: Window Functions - ROW_NUMBER & RANK

```sql
WITH sales_summary AS (
  SELECT
    p.category,
    p.name as product_name,
    SUM(s.quantity) as total_quantity,
    ROW_NUMBER() OVER (PARTITION BY p.category ORDER BY SUM(s.quantity) DESC) as rank
  FROM products p
  JOIN sales s ON p.product_id = s.product_id
  GROUP BY p.category, p.name
)
SELECT category, product_name, total_quantity, rank
FROM sales_summary
WHERE rank <= 2
ORDER BY category, rank;
```

**Giải thích**:
- ROW_NUMBER() OVER (PARTITION BY category ORDER BY ...) tạo xếp hạng
- PARTITION BY chia dữ liệu thành nhóm theo category
- ORDER BY xác định thứ tự xếp hạng
- WHERE rank <= 2 lọc chỉ top 2

---

## Hard 2: LAG & LEAD Functions

```sql
SELECT
  sale_date,
  amount,
  LAG(amount) OVER (ORDER BY sale_date) as prev_day_amount,
  LEAD(amount) OVER (ORDER BY sale_date) as next_day_amount,
  amount - LAG(amount) OVER (ORDER BY sale_date) as change_from_prev,
  LEAD(amount) OVER (ORDER BY sale_date) - amount as change_to_next
FROM sales
ORDER BY sale_date;
```

**Giải thích**:
- LAG(amount) OVER (ORDER BY sale_date) lấy amount của ngày trước
- LEAD(amount) OVER (ORDER BY sale_date) lấy amount của ngày sau
- Tính toán sự thay đổi bằng phép trừ
- NULL cho giá trị không có (ngày đầu/cuối)

---

## Hard 3: Common Table Expression (CTE)

```sql
WITH monthly_revenue AS (
  SELECT
    DATE_TRUNC('month', order_date) as month, -- PostgreSQL
    -- Hoặc: DATE_FORMAT(order_date, '%Y-%m') as month, -- MySQL
    SUM(amount) as total_revenue
  FROM orders
  GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
  month,
  total_revenue,
  RANK() OVER (ORDER BY total_revenue DESC) as rank
FROM monthly_revenue
ORDER BY rank;
```

**Giải thích**:
- WITH ... AS tạo CTE (Common Table Expression)
- DATE_TRUNC trích xuất tháng từ ngày
- CTE được sử dụng như bảng thông thường
- RANK() tạo xếp hạng

**MySQL version**:
```sql
WITH monthly_revenue AS (
  SELECT
    DATE_FORMAT(order_date, '%Y-%m') as month,
    SUM(amount) as total_revenue
  FROM orders
  GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT
  month,
  total_revenue,
  RANK() OVER (ORDER BY total_revenue DESC) as rank
FROM monthly_revenue
ORDER BY rank;
```

---

## Hard 4: Running Total (Cumulative Sum)

```sql
SELECT
  sale_date,
  amount,
  SUM(amount) OVER (PARTITION BY sale_date ORDER BY sale_id) as daily_total,
  SUM(amount) OVER (ORDER BY sale_date, sale_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM sales
ORDER BY sale_date, sale_id;
```

**Giải thích**:
- PARTITION BY sale_date ORDER BY sale_id tính tổng daily
- ORDER BY sale_date, sale_id ROWS BETWEEN ... tính running total
- UNBOUNDED PRECEDING: từ đầu
- CURRENT ROW: đến hàng hiện tại
- Kết quả: mỗi hàng có tổng từ đầu đến hàng đó

---

## Hard 5: Complex Multi-step Query

```sql
WITH customer_orders AS (
  SELECT
    c.customer_id,
    c.name,
    SUM(o.total) as total_order_value,
    COUNT(DISTINCT o.order_id) as total_orders,
    COUNT(DISTINCT oi.product_id) as unique_products
  FROM customers c
  JOIN orders o ON c.customer_id = o.customer_id
  JOIN order_items oi ON o.order_id = oi.order_id
  GROUP BY c.customer_id, c.name
)
SELECT
  RANK() OVER (ORDER BY total_order_value DESC) as rank,
  customer_id,
  name as customer_name,
  total_order_value,
  total_orders,
  unique_products
FROM customer_orders
LIMIT 3;
```

**Giải thích**:
- CTE tính tổng giá trị, số đơn, và số sản phẩm khác nhau
- COUNT(DISTINCT oi.product_id) đếm số sản phẩm khác nhau
- RANK() OVER (ORDER BY ...) tạo xếp hạng
- LIMIT 3 lấy top 3

---

# Benchmark Performance 🚀

| Problem | Expected Execution Time |
|---------|------------------------|
| Easy | < 10ms |
| Medium | 10-100ms |
| Hard | 100ms - 1s |

---

# Common SQL Patterns 📝

## 1. Top N per Group
```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rn
  FROM products
) subq
WHERE rn <= N;
```

## 2. Running Total
```sql
SELECT *, SUM(amount) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM sales;
```

## 3. Percentage of Total
```sql
SELECT category, SUM(amount) * 100.0 / SUM(SUM(amount)) OVER () as percentage
FROM sales
GROUP BY category;
```

## 4. Year-over-Year Comparison
```sql
SELECT
  EXTRACT(MONTH FROM sale_date) as month,
  EXTRACT(YEAR FROM sale_date) as year,
  SUM(amount) as total
FROM sales
GROUP BY EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date)
ORDER BY month, year;
```

---

**Good luck! 🎯 Practise makes perfect!**
