# SQL Medium Problems (8 bài)

## Problem 1️⃣: Multiple JOINs

**Difficulty**: ⭐⭐ Medium

**Topics**: MULTIPLE JOINs, SELECT, WHERE

**Description**:
Lấy tên khách hàng, sản phẩm họ mua, số lượng và giá, sắp xếp theo tên khách hàng

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY(customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    FOREIGN KEY(order_id) REFERENCES orders(order_id)
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
(3, 'Bob Johnson');

INSERT INTO orders VALUES
(1, 1, '2024-01-10'),
(2, 1, '2024-02-15'),
(3, 2, '2024-01-20'),
(4, 3, '2024-03-05');

INSERT INTO products VALUES
(1, 'Laptop'),
(2, 'Mouse'),
(3, 'Keyboard');

INSERT INTO order_items VALUES
(1, 1, 1, 1, 1200.00),
(2, 1, 2, 2, 25.00),
(3, 2, 3, 1, 75.00),
(4, 3, 2, 3, 25.00),
(5, 4, 1, 1, 1200.00);
```

**Expected Output**:
```
customer_name | product_name | quantity | price
Bob Johnson   | Laptop       | 1        | 1200.00
Jane Smith    | Mouse        | 3        | 25.00
John Doe      | Keyboard     | 1        | 75.00
John Doe      | Laptop       | 1        | 1200.00
John Doe      | Mouse        | 2        | 25.00
```

---

## Problem 2️⃣: LEFT JOIN with Aggregation

**Difficulty**: ⭐⭐ Medium

**Topics**: LEFT JOIN, GROUP BY, COUNT, HAVING

**Description**:
Liệt kê tất cả khách hàng và số đơn hàng họ đã đặt (bao gồm những khách chưa đặt đơn)

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
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
(1, 1, '2024-01-10'),
(2, 1, '2024-02-15'),
(3, 2, '2024-01-20'),
(4, 1, '2024-03-05');
```

**Expected Output**:
```
customer_id | name           | order_count
1           | John Doe       | 3
2           | Jane Smith     | 1
3           | Bob Johnson    | 0
4           | Alice Williams | 0
```

---

## Problem 3️⃣: Subquery in WHERE

**Difficulty**: ⭐⭐ Medium

**Topics**: SUBQUERY, WHERE, SELECT

**Description**:
Lấy tất cả khách hàng có tổng tiêu tiền > trung bình tiêu tiền của tất cả khách

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    total DECIMAL(10, 2)
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
(1, 1, 100.00),
(2, 1, 150.00),
(3, 2, 50.00),
(4, 3, 500.00),
(5, 4, 75.00),
(6, 1, 200.00);
```

**Expected Output**:
```
customer_id | name       | total_spending
1           | John Doe   | 450.00
3           | Bob Johnson| 500.00
```

---

## Problem 4️⃣: CASE WHEN

**Difficulty**: ⭐⭐ Medium

**Topics**: CASE WHEN, SELECT, ORDER BY

**Description**:
Liệt kê sản phẩm với phân loại giá: 'Rẻ' (< 100), 'Trung bình' (100-500), 'Đắt' (> 500)

**Schema**:
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO products VALUES
(1, 'Mouse', 25.00),
(2, 'Keyboard', 75.00),
(3, 'Monitor', 300.00),
(4, 'Laptop', 1200.00),
(5, 'Desk', 150.00),
(6, 'Chair', 100.00);
```

**Expected Output**:
```
product_id | name      | price     | price_category
1          | Mouse     | 25.00     | Rẻ
2          | Keyboard  | 75.00     | Rẻ
3          | Monitor   | 300.00    | Trung bình
4          | Laptop    | 1200.00   | Đắt
5          | Desk      | 150.00    | Trung bình
6          | Chair     | 100.00    | Trung bình
```

---

## Problem 5️⃣: HAVING Clause

**Difficulty**: ⭐⭐ Medium

**Topics**: GROUP BY, HAVING, COUNT, SUM

**Description**:
Lấy tất cả categories có tổng giá trị sản phẩm > 500 và có >= 2 sản phẩm

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
(3, 'Monitor', 400.00, 'Electronics'),
(4, 'Desk', 150.00, 'Furniture'),
(5, 'Chair', 200.00, 'Furniture'),
(6, 'Lamp', 30.00, 'Furniture');
```

**Expected Output**:
```
category    | total_value | product_count
Electronics | 500.00      | 3
Furniture   | 380.00      | 3
```

---

## Problem 6️⃣: Self JOIN

**Difficulty**: ⭐⭐ Medium

**Topics**: SELF JOIN, SELECT

**Description**:
Tìm các cặp nhân viên làm trong cùng phòng

**Schema**:
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT
);
```

**Sample Data**:
```sql
INSERT INTO employees VALUES
(1, 'John', 1),
(2, 'Jane', 1),
(3, 'Bob', 2),
(4, 'Alice', 1),
(5, 'Charlie', 2);
```

**Expected Output**:
```
employee1 | employee2 | department_id
John      | Jane      | 1
John      | Alice     | 1
Jane      | Alice     | 1
Bob       | Charlie   | 2
```

---

## Problem 7️⃣: UNION

**Difficulty**: ⭐⭐ Medium

**Topics**: UNION, SELECT

**Description**:
Kết hợp danh sách tất cả khách hàng và nhà cung cấp (làm một bảng duy nhất)

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    type VARCHAR(20)
);

CREATE TABLE suppliers (
    supplier_id INT PRIMARY KEY,
    name VARCHAR(100),
    type VARCHAR(20)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe', 'Customer'),
(2, 'Jane Smith', 'Customer');

INSERT INTO suppliers VALUES
(1, 'Tech Supply Co', 'Supplier'),
(2, 'Office Furniture Ltd', 'Supplier');
```

**Expected Output**:
```
name                   | type
John Doe               | Customer
Jane Smith             | Customer
Tech Supply Co         | Supplier
Office Furniture Ltd   | Supplier
```

---

## Problem 8️⃣: Correlated Subquery

**Difficulty**: ⭐⭐ Medium

**Topics**: CORRELATED SUBQUERY, SELECT, WHERE

**Description**:
Lấy sản phẩm có giá cao hơn giá trung bình của category của nó

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
(3, 'Monitor', 400.00, 'Electronics'),
(4, 'Desk', 150.00, 'Furniture'),
(5, 'Chair', 200.00, 'Furniture'),
(6, 'Lamp', 50.00, 'Furniture');
```

**Expected Output**:
```
product_id | name      | price     | category
2          | Keyboard  | 75.00     | Electronics
3          | Monitor   | 400.00    | Electronics
5          | Chair     | 200.00    | Furniture
```

---

**Next**: Khi hoàn thành Medium level, hãy tiếp tục với Hard problems! 💪
