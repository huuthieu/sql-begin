# SQL Hard Problems (5 bài)

## Problem 1️⃣: Window Functions - ROW_NUMBER & RANK

**Difficulty**: ⭐⭐⭐ Hard

**Topics**: WINDOW FUNCTIONS, ROW_NUMBER, RANK, PARTITION BY

**Description**:
Lấy top 2 sản phẩm bán chạy nhất theo từng category (dựa trên tổng số lượng bán)

**Schema**:
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50)
);

CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    sale_date DATE
);
```

**Sample Data**:
```sql
INSERT INTO products VALUES
(1, 'Mouse', 'Electronics'),
(2, 'Keyboard', 'Electronics'),
(3, 'Monitor', 'Electronics'),
(4, 'Desk', 'Furniture'),
(5, 'Chair', 'Furniture'),
(6, 'Lamp', 'Furniture');

INSERT INTO sales VALUES
(1, 1, 50, '2024-01-01'),
(2, 1, 30, '2024-01-15'),
(3, 2, 40, '2024-01-02'),
(4, 2, 20, '2024-01-16'),
(5, 3, 10, '2024-01-03'),
(6, 4, 60, '2024-01-04'),
(7, 4, 25, '2024-01-17'),
(8, 5, 45, '2024-01-05'),
(9, 6, 15, '2024-01-06');
```

**Expected Output**:
```
category    | product_name | total_quantity | rank
Electronics | Mouse        | 80             | 1
Electronics | Keyboard     | 60             | 2
Furniture   | Desk         | 85             | 1
Furniture   | Chair        | 45             | 2
```

---

## Problem 2️⃣: LAG & LEAD Functions

**Difficulty**: ⭐⭐⭐ Hard

**Topics**: WINDOW FUNCTIONS, LAG, LEAD, ORDER BY

**Description**:
Lấy bán hàng hàng ngày cùng với bán hàng ngày trước và ngày sau (so sánh các ngày liên tiếp)

**Schema**:
```sql
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    sale_date DATE,
    amount DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO sales VALUES
(1, '2024-01-01', 1000.00),
(2, '2024-01-02', 1500.00),
(3, '2024-01-03', 1200.00),
(4, '2024-01-04', 1800.00),
(5, '2024-01-05', 1600.00);
```

**Expected Output**:
```
sale_date  | amount    | prev_day_amount | next_day_amount | change_from_prev | change_to_next
2024-01-01 | 1000.00   | NULL            | 1500.00         | NULL             | 500.00
2024-01-02 | 1500.00   | 1000.00         | 1200.00         | 500.00           | -300.00
2024-01-03 | 1200.00   | 1500.00         | 1800.00         | -300.00          | 600.00
2024-01-04 | 1800.00   | 1200.00         | 1600.00         | 600.00           | -200.00
2024-01-05 | 1600.00   | 1800.00         | NULL            | -200.00          | NULL
```

---

## Problem 3️⃣: Common Table Expression (CTE)

**Difficulty**: ⭐⭐⭐ Hard

**Topics**: CTE, WITH, RECURSIVE, MULTIPLE CTEs

**Description**:
Tính tổng doanh thu theo tháng và xếp hạng tháng theo doanh thu (từ cao xuống thấp)

**Schema**:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    amount DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO orders VALUES
(1, '2024-01-05', 500.00),
(2, '2024-01-15', 750.00),
(3, '2024-02-10', 1000.00),
(4, '2024-02-20', 800.00),
(5, '2024-03-05', 600.00),
(6, '2024-01-25', 300.00),
(7, '2024-02-28', 200.00);
```

**Expected Output**:
```
month       | total_revenue | rank
2024-02     | 2000.00       | 1
2024-01     | 1550.00       | 2
2024-03     | 600.00        | 3
```

---

## Problem 4️⃣: Running Total (Cumulative Sum)

**Difficulty**: ⭐⭐⭐ Hard

**Topics**: WINDOW FUNCTIONS, SUM OVER, ROWS BETWEEN

**Description**:
Tính tổng hợp doanh thu (running total) theo từng ngày

**Schema**:
```sql
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    sale_date DATE,
    amount DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO sales VALUES
(1, '2024-01-01', 1000.00),
(2, '2024-01-01', 500.00),
(3, '2024-01-02', 1200.00),
(4, '2024-01-03', 800.00),
(5, '2024-01-03', 600.00);
```

**Expected Output**:
```
sale_date  | amount    | daily_total | running_total
2024-01-01 | 1000.00   | 1500.00     | 1500.00
2024-01-01 | 500.00    | 1500.00     | 1500.00
2024-01-02 | 1200.00   | 1200.00     | 2700.00
2024-01-03 | 800.00    | 1400.00     | 4100.00
2024-01-03 | 600.00    | 1400.00     | 4100.00
```

---

## Problem 5️⃣: Complex Multi-step Query

**Difficulty**: ⭐⭐⭐ Hard

**Topics**: CTE, WINDOW FUNCTIONS, MULTIPLE JOINS, SUBQUERIES

**Description**:
Tìm top 3 khách hàng có giá trị đơn hàng cao nhất, kèm theo tổng số đơn hàng và số sản phẩm khác nhau họ mua

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    total DECIMAL(10, 2),
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
    name VARCHAR(100)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe'),
(2, 'Jane Smith'),
(3, 'Bob Johnson'),
(4, 'Alice Williams');

INSERT INTO orders VALUES
(1, 1, 500.00, '2024-01-05'),
(2, 1, 300.00, '2024-01-15'),
(3, 2, 800.00, '2024-01-10'),
(4, 2, 200.00, '2024-01-20'),
(5, 3, 1200.00, '2024-02-05'),
(6, 4, 100.00, '2024-02-10');

INSERT INTO products VALUES
(1, 'Laptop'),
(2, 'Mouse'),
(3, 'Keyboard'),
(4, 'Monitor');

INSERT INTO order_items VALUES
(1, 1, 1, 1),
(2, 1, 2, 2),
(3, 2, 3, 1),
(4, 3, 1, 1),
(5, 3, 4, 1),
(6, 4, 2, 3),
(7, 5, 1, 2),
(8, 5, 3, 1),
(9, 6, 2, 1);
```

**Expected Output**:
```
rank | customer_id | customer_name  | total_order_value | total_orders | unique_products
1    | 3           | Bob Johnson    | 1200.00           | 1            | 2
2    | 2           | Jane Smith     | 1000.00           | 2            | 2
3    | 1           | John Doe       | 800.00            | 2            | 2
```

---

## Tips for Hard Problems 💡

1. **ROW_NUMBER vs RANK vs DENSE_RANK**:
   - ROW_NUMBER: Không có xếp hạng giống nhau
   - RANK: Có xếp hạng giống nhau, bỏ số hạng tiếp theo
   - DENSE_RANK: Có xếp hạng giống nhau, không bỏ số hạng

2. **LAG & LEAD**:
   - LAG: Lấy dữ liệu từ hàng trước
   - LEAD: Lấy dữ liệu từ hàng sau
   - Có thể chỉ định offset và default value

3. **CTE (WITH clause)**:
   - Giúp code dễ đọc hơn
   - Có thể dùng nhiều CTE
   - Một số DB hỗ trợ recursive CTE

4. **Window Functions**:
   - PARTITION BY: Chia dữ liệu thành nhóm
   - ORDER BY: Sắp xếp trong mỗi nhóm
   - ROWS BETWEEN: Định nghĩa window (current, unbounded, etc.)

5. **Performance**:
   - Chỉ mục (Index) là quan trọng
   - Tránh subquery lồng sâu
   - Sử dụng CTE thay vì subquery lặp

---

**Congratulations!** Hoàn thành tất cả Hard problems = Bạn đã sẵn sàng cho phỏng vấn! 🎉
