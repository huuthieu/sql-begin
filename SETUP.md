# SQL Setup Guide

Hướng dẫn chuẩn bị môi trường và setup database cho các bài tập.

---

## 🛠️ Cài Đặt Môi Trường

### Option 1: MySQL (Recommended for Interviews)

**Cài đặt MySQL trên Mac**:
```bash
brew install mysql
brew services start mysql
mysql -u root
```

**Cài đặt MySQL trên Ubuntu/Linux**:
```bash
sudo apt-get install mysql-server
sudo mysql_secure_installation
mysql -u root -p
```

**Cài đặt MySQL trên Windows**:
- Download từ: https://dev.mysql.com/downloads/mysql/
- Chạy installer và follow hướng dẫn

---

### Option 2: PostgreSQL

**Cài đặt PostgreSQL trên Mac**:
```bash
brew install postgresql@15
brew services start postgresql@15
```

**Cài đặt PostgreSQL trên Ubuntu/Linux**:
```bash
sudo apt-get install postgresql postgresql-contrib
sudo systemctl start postgresql
```

---

### Option 3: SQLite (Đơn giản nhất, không cần setup)

**Cài đặt SQLite trên Mac**:
```bash
brew install sqlite
```

**Tạo database**:
```bash
sqlite3 practice.db
```

---

## 📊 GUI Tools (Optional)

Các công cụ giúp dễ dàng quản lý và chạy SQL queries:

- **DBeaver**: https://dbeaver.io/ (Free, cross-platform)
- **DataGrip**: https://www.jetbrains.com/datagrip/ (Paid, professional)
- **MySQL Workbench**: https://www.mysql.com/products/workbench/ (Free, MySQL only)
- **pgAdmin**: https://www.pgadmin.org/ (Free, PostgreSQL only)
- **VS Code + SQLTools Extension**: (Lightweight, free)

---

## 🚀 Quick Start (MySQL)

### 1. Login vào MySQL
```bash
mysql -u root -p
# Nhập password khi được yêu cầu
```

### 2. Tạo Database cho Practice
```sql
CREATE DATABASE sql_practice;
USE sql_practice;
```

### 3. Tạo bảng cho Easy Problem 1
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    city VARCHAR(50)
);
```

### 4. Insert Sample Data
```sql
INSERT INTO customers VALUES
(1, 'John Doe', 'john@email.com', 'USA', 'New York'),
(2, 'Jane Smith', 'jane@email.com', 'UK', 'London'),
(3, 'Bob Johnson', 'bob@email.com', 'USA', 'Los Angeles'),
(4, 'Alice Williams', 'alice@email.com', 'Canada', 'Toronto'),
(5, 'Charlie Brown', 'charlie@email.com', 'USA', 'Chicago');
```

### 5. Kiểm tra dữ liệu
```sql
SELECT * FROM customers;
```

### 6. Giải quyết Problem 1
```sql
SELECT * FROM customers WHERE country = 'USA';
```

---

## 📝 Creating Multiple Tables (Efficient Way)

Bạn có thể tạo tệp SQL và chạy tất cả cùng lúc:

**Tạo file `setup_easy.sql`**:
```sql
-- Easy Problems Setup

-- Problem 1
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    city VARCHAR(50)
);

INSERT INTO customers VALUES
(1, 'John Doe', 'john@email.com', 'USA', 'New York'),
(2, 'Jane Smith', 'jane@email.com', 'UK', 'London'),
(3, 'Bob Johnson', 'bob@email.com', 'USA', 'Los Angeles'),
(4, 'Alice Williams', 'alice@email.com', 'Canada', 'Toronto'),
(5, 'Charlie Brown', 'charlie@email.com', 'USA', 'Chicago');

-- Problem 2
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2),
    category VARCHAR(50)
);

INSERT INTO products VALUES
(1, 'Laptop', 1200.00, 'Electronics'),
(2, 'Mouse', 25.00, 'Electronics'),
(3, 'Desk', 300.00, 'Furniture'),
(4, 'Chair', 150.00, 'Furniture'),
(5, 'Monitor', 400.00, 'Electronics'),
(6, 'Keyboard', 75.00, 'Electronics'),
(7, 'Desk Lamp', 50.00, 'Furniture');

-- ... more tables
```

**Chạy file**:
```bash
mysql -u root -p sql_practice < setup_easy.sql
```

---

## 🔧 Useful Commands

### Show all databases
```sql
SHOW DATABASES;
```

### Select a database
```sql
USE database_name;
```

### Show all tables
```sql
SHOW TABLES;
```

### Show table structure
```sql
DESCRIBE table_name;
```

### Show row count
```sql
SELECT COUNT(*) FROM table_name;
```

### Delete all data (not table structure)
```sql
TRUNCATE TABLE table_name;
```

### Delete table
```sql
DROP TABLE table_name;
```

### Export data to CSV
```sql
SELECT * FROM table_name
INTO OUTFILE '/path/to/file.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

---

## 💡 Tips for Practice

1. **Start Fresh Each Time**:
```sql
DROP TABLE IF EXISTS table_name;
CREATE TABLE table_name (...);
INSERT INTO table_name VALUES (...);
```

2. **Test Query with LIMIT**:
```sql
SELECT * FROM large_table LIMIT 10;
```

3. **Check Execution Time**:
```sql
-- MySQL
SELECT COUNT(*) FROM table_name;

-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM table_name;
```

4. **Use Comments in SQL**:
```sql
-- Single line comment
SELECT * FROM customers; -- Get all customers

/* Multi-line
   comment */
SELECT * FROM orders;
```

---

## 🐛 Common Issues

### Issue: "Access denied for user 'root'@'localhost'"
**Solution**:
```bash
mysql -u root
# Hoặc
mysql -u root -p
```

### Issue: "Unknown database 'sql_practice'"
**Solution**:
```sql
CREATE DATABASE sql_practice;
USE sql_practice;
```

### Issue: "Column count doesn't match value count"
**Solution**: Kiểm tra số cột trong CREATE TABLE và INSERT VALUES khớp nhau

### Issue: "Table already exists"
**Solution**:
```sql
DROP TABLE table_name;
-- Hoặc
DROP TABLE IF EXISTS table_name;
```

---

## 📚 Learning Resources

- **W3Schools SQL Tutorial**: https://www.w3schools.com/sql/
- **SQL Documentation**:
  - MySQL: https://dev.mysql.com/doc/
  - PostgreSQL: https://www.postgresql.org/docs/
- **LeetCode Database**: https://leetcode.com/problemset/database/
- **HackerRank SQL**: https://www.hackerrank.com/domains/sql

---

**Ready to start practicing? 🚀**
