# SQL Easy Problems (7 bài)

## Problem 1️⃣: Select All Customers

**Difficulty**: ⭐ Easy

**Topics**: SELECT, WHERE

**Description**:
Lấy tất cả thông tin của customers có country = 'USA'

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    city VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe', 'john@email.com', 'USA', 'New York'),
(2, 'Jane Smith', 'jane@email.com', 'UK', 'London'),
(3, 'Bob Johnson', 'bob@email.com', 'USA', 'Los Angeles'),
(4, 'Alice Williams', 'alice@email.com', 'Canada', 'Toronto'),
(5, 'Charlie Brown', 'charlie@email.com', 'USA', 'Chicago');
```

**Expected Output**:
```
customer_id | name           | email              | country | city
1           | John Doe       | john@email.com     | USA     | New York
3           | Bob Johnson    | bob@email.com      | USA     | Los Angeles
5           | Charlie Brown  | charlie@email.com  | USA     | Chicago
```

---

## Problem 2️⃣: Order by Price

**Difficulty**: ⭐ Easy

**Topics**: SELECT, ORDER BY, LIMIT

**Description**:
Lấy 5 sản phẩm có giá cao nhất, sắp xếp từ cao xuống thấp

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
(1, 'Laptop', 1200.00, 'Electronics'),
(2, 'Mouse', 25.00, 'Electronics'),
(3, 'Desk', 300.00, 'Furniture'),
(4, 'Chair', 150.00, 'Furniture'),
(5, 'Monitor', 400.00, 'Electronics'),
(6, 'Keyboard', 75.00, 'Electronics'),
(7, 'Desk Lamp', 50.00, 'Furniture');
```

**Expected Output**:
```
product_id | name      | price
1          | Laptop    | 1200.00
5          | Monitor   | 400.00
3          | Desk      | 300.00
4          | Chair     | 150.00
6          | Keyboard  | 75.00
```

---

## Problem 3️⃣: Count Records

**Difficulty**: ⭐ Easy

**Topics**: COUNT, GROUP BY, WHERE

**Description**:
Đếm số lượng khách hàng theo từng country, chỉ hiển thị countries có >= 2 customers

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    country VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe', 'USA'),
(2, 'Jane Smith', 'UK'),
(3, 'Bob Johnson', 'USA'),
(4, 'Alice Williams', 'Canada'),
(5, 'Charlie Brown', 'USA'),
(6, 'David Lee', 'UK');
```

**Expected Output**:
```
country | count
USA     | 3
UK      | 2
```

---

## Problem 4️⃣: Simple INNER JOIN

**Difficulty**: ⭐ Easy

**Topics**: INNER JOIN, SELECT

**Description**:
Liệt kê tên khách hàng và tên sản phẩm mà họ đã mua

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    FOREIGN KEY(customer_id) REFERENCES customers(customer_id)
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

INSERT INTO products VALUES
(1, 'Laptop'),
(2, 'Mouse'),
(3, 'Keyboard');

INSERT INTO orders VALUES
(1, 1, 1),
(2, 1, 2),
(3, 2, 3),
(4, 3, 1);
```

**Expected Output**:
```
customer_name | product_name
John Doe      | Laptop
John Doe      | Mouse
Jane Smith    | Keyboard
Bob Johnson   | Laptop
```

---

## Problem 5️⃣: DISTINCT Values

**Difficulty**: ⭐ Easy

**Topics**: DISTINCT, SELECT

**Description**:
Lấy danh sách các categories sản phẩm (không trùng lặp)

**Schema**:
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50)
);
```

**Sample Data**:
```sql
INSERT INTO products VALUES
(1, 'Laptop', 'Electronics'),
(2, 'Mouse', 'Electronics'),
(3, 'Desk', 'Furniture'),
(4, 'Chair', 'Furniture'),
(5, 'Monitor', 'Electronics');
```

**Expected Output**:
```
category
Electronics
Furniture
```

---

## Problem 6️⃣: SUM with GROUP BY

**Difficulty**: ⭐ Easy

**Topics**: SUM, GROUP BY, JOIN

**Description**:
Tính tổng doanh thu (số lượng × giá) theo từng khách hàng

**Schema**:
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    quantity INT,
    price DECIMAL(10, 2)
);
```

**Sample Data**:
```sql
INSERT INTO customers VALUES
(1, 'John Doe'),
(2, 'Jane Smith'),
(3, 'Bob Johnson');

INSERT INTO orders VALUES
(1, 1, 2, 100.00),
(2, 1, 1, 50.00),
(3, 2, 3, 75.00),
(4, 3, 1, 100.00),
(5, 1, 2, 50.00);
```

**Expected Output**:
```
customer_id | name         | total_revenue
1           | John Doe     | 350.00
2           | Jane Smith   | 225.00
3           | Bob Johnson  | 100.00
```

---

## Problem 7️⃣: WHERE with BETWEEN

**Difficulty**: ⭐ Easy

**Topics**: WHERE, BETWEEN, SELECT

**Description**:
Lấy tất cả sản phẩm có giá từ 50 đến 200

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
(1, 'Laptop', 1200.00),
(2, 'Mouse', 25.00),
(3, 'Desk', 150.00),
(4, 'Chair', 100.00),
(5, 'Monitor', 400.00),
(6, 'Keyboard', 75.00);
```

**Expected Output**:
```
product_id | name      | price
3          | Desk      | 150.00
4          | Chair     | 100.00
6          | Keyboard  | 75.00
```

---

**Next**: Hãy hoàn thành tất cả 7 bài này trước khi chuyển sang Medium level! 🚀
