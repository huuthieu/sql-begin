# Giải thích Expert 3: Cohort Analysis & Retention

## 📚 Cohort Analysis là gì?

**Cohort Analysis** (Phân tích nhóm người dùng) là một kỹ thuật phân tích dữ liệu giúp theo dõi hành vi của các nhóm người dùng theo thời gian. Mỗi **cohort** là một nhóm người dùng có chung một đặc điểm nhất định (thường là thời gian đăng ký).

**Retention Rate** (Tỷ lệ giữ chân khách hàng) đo lường bao nhiêu % khách hàng trong một cohort tiếp tục sử dụng dịch vụ sau X tháng.

### Tại sao Cohort Analysis quan trọng?

- **Đánh giá sức khỏe của business**: Retention cao = khách hàng trung thành
- **So sánh các chiến dịch marketing**: Cohort tháng nào có retention tốt hơn?
- **Dự đoán revenue**: Biết được bao nhiêu % khách hàng sẽ quay lại
- **Tối ưu hóa sản phẩm**: Tìm ra tháng nào khách hàng rời bỏ nhiều

---

## 🎯 Mô tả bài toán

Bạn có 2 bảng:
- `customers`: Thông tin khách hàng và ngày đăng ký
- `orders`: Các đơn hàng của khách hàng

**Yêu cầu**: Tạo một bảng retention hiển thị % khách hàng quay lại mua hàng trong tháng 0, 1, 2, 3... sau khi đăng ký.

### Dữ liệu mẫu

```sql
-- Khách hàng
(1, '2024-01-15')  -- Customer 1 đăng ký ngày 15/01
(2, '2024-01-20')  -- Customer 2 đăng ký ngày 20/01
(3, '2024-01-25')  -- Customer 3 đăng ký ngày 25/01
(4, '2024-02-10')  -- Customer 4 đăng ký ngày 10/02
(5, '2024-02-15')  -- Customer 5 đăng ký ngày 15/02
(6, '2024-03-05')  -- Customer 6 đăng ký ngày 05/03

-- Đơn hàng
Customer 1: có đơn vào 15/01, 10/02, 05/03
Customer 2: có đơn vào 25/01, 15/02
Customer 3: có đơn vào 01/02
...
```

### Kết quả mong đợi

```
cohort_month | cohort_size | month_0 | month_1 | month_2 | month_3
2024-01      | 3           | 100%    | 67%     | 33%     | 0%
2024-02      | 2           | 100%    | 100%    | 50%     | NULL
2024-03      | 1           | 100%    | 33%     | NULL    | NULL
```

**Giải thích kết quả**:
- **Cohort 2024-01** (3 khách hàng: 1, 2, 3):
  - **Month 0**: 100% (cả 3 đều có đơn trong tháng đăng ký)
  - **Month 1**: 67% (2/3 khách quay lại tháng sau: khách 1 và 2)
  - **Month 2**: 33% (1/3 khách quay lại: chỉ khách 1)
  - **Month 3**: 0% (không ai quay lại)

---

## 💡 Cách giải quyết - Từng bước chi tiết

### Bước 1: Xác định Cohort của mỗi khách hàng

Mỗi khách hàng thuộc về một cohort dựa trên **tháng đăng ký**.

```sql
WITH customer_cohort AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', signup_date) as cohort_month
  FROM customers
)
```

**Kết quả Bước 1**:
```
customer_id | cohort_month
1           | 2024-01-01
2           | 2024-01-01
3           | 2024-01-01
4           | 2024-02-01
5           | 2024-02-01
6           | 2024-03-01
```

**Giải thích**:
- `DATE_TRUNC('month', signup_date)` chuyển ngày đăng ký thành ngày đầu tháng
- Ví dụ: '2024-01-15', '2024-01-20', '2024-01-25' → đều thành '2024-01-01'

---

### Bước 2: Tính Month Index cho mỗi đơn hàng

Với mỗi đơn hàng, tính xem nó được đặt sau bao nhiêu tháng kể từ khi khách hàng đăng ký.

```sql
user_activity AS (
  SELECT
    cc.customer_id,
    cc.cohort_month,
    DATE_TRUNC('month', o.order_date) as order_month,
    (DATE_PART('year', DATE_TRUNC('month', o.order_date)) - DATE_PART('year', cc.cohort_month)) * 12 +
    (DATE_PART('month', DATE_TRUNC('month', o.order_date)) - DATE_PART('month', cc.cohort_month)) as month_number
  FROM customer_cohort cc
  LEFT JOIN orders o ON cc.customer_id = o.customer_id
)
```

**Kết quả Bước 2** (một vài ví dụ):
```
customer_id | cohort_month | order_month | month_number
1           | 2024-01-01   | 2024-01-01  | 0    (đơn trong tháng đăng ký)
1           | 2024-01-01   | 2024-02-01  | 1    (đơn sau 1 tháng)
1           | 2024-01-01   | 2024-03-01  | 2    (đơn sau 2 tháng)
2           | 2024-01-01   | 2024-01-01  | 0
2           | 2024-01-01   | 2024-02-01  | 1
3           | 2024-01-01   | 2024-02-01  | 1
```

**Giải thích công thức month_number**:
```sql
(year_order - year_cohort) * 12 + (month_order - month_cohort)
```
Ví dụ: Cohort = 2024-01, Order = 2024-03
- (2024 - 2024) * 12 + (3 - 1) = 0 + 2 = 2 tháng

---

### Bước 3: Tính kích thước của mỗi Cohort

Đếm số lượng khách hàng trong mỗi cohort.

```sql
cohort_size AS (
  SELECT
    cohort_month,
    COUNT(DISTINCT customer_id) as cohort_size
  FROM customer_cohort
  GROUP BY cohort_month
)
```

**Kết quả Bước 3**:
```
cohort_month | cohort_size
2024-01-01   | 3
2024-02-01   | 2
2024-03-01   | 1
```

---

### Bước 4: Tính số khách hàng active trong mỗi tháng

Với mỗi cohort và mỗi month_number, đếm có bao nhiêu khách hàng DISTINCT có đơn hàng.

```sql
retention_table AS (
  SELECT
    ua.cohort_month,
    ua.month_number,
    COUNT(DISTINCT ua.customer_id) as retained_customers
  FROM user_activity ua
  WHERE ua.month_number >= 0
  GROUP BY ua.cohort_month, ua.month_number
)
```

**Kết quả Bước 4**:
```
cohort_month | month_number | retained_customers
2024-01-01   | 0            | 3    (cả 3 khách đều có đơn tháng 0)
2024-01-01   | 1            | 2    (khách 1, 2 có đơn tháng 1)
2024-01-01   | 2            | 1    (chỉ khách 1 có đơn tháng 2)
2024-02-01   | 0            | 2
2024-02-01   | 1            | 2
2024-02-01   | 2            | 1
2024-03-01   | 0            | 1
```

---

### Bước 5: Tính Retention Rate (%)

Chia số khách hàng retained cho cohort_size, rồi nhân 100%.

```sql
SELECT
  rt.cohort_month,
  cs.cohort_size,
  rt.month_number,
  ROUND(100.0 * rt.retained_customers / cs.cohort_size, 1) as retention_pct
FROM retention_table rt
JOIN cohort_size cs ON rt.cohort_month = cs.cohort_month
ORDER BY rt.cohort_month, rt.month_number;
```

**Kết quả Bước 5**:
```
cohort_month | cohort_size | month_number | retention_pct
2024-01-01   | 3           | 0            | 100.0%   (3/3 * 100)
2024-01-01   | 3           | 1            | 66.7%    (2/3 * 100)
2024-01-01   | 3           | 2            | 33.3%    (1/3 * 100)
2024-02-01   | 2           | 0            | 100.0%
2024-02-01   | 2           | 1            | 100.0%
2024-02-01   | 2           | 2            | 50.0%
```

---

### Bước 6: PIVOT để hiển thị dạng bảng ngang

Chuyển các dòng `month_number` thành các cột `month_0`, `month_1`, `month_2`...

```sql
SELECT
  cohort_month,
  cohort_size,
  ROUND(100.0 * MAX(CASE WHEN month_number = 0 THEN retained_customers END) / cohort_size, 1) as month_0,
  ROUND(100.0 * MAX(CASE WHEN month_number = 1 THEN retained_customers END) / cohort_size, 1) as month_1,
  ROUND(100.0 * MAX(CASE WHEN month_number = 2 THEN retained_customers END) / cohort_size, 1) as month_2,
  ROUND(100.0 * MAX(CASE WHEN month_number = 3 THEN retained_customers END) / cohort_size, 1) as month_3
FROM retention_table rt
JOIN cohort_size cs ON rt.cohort_month = cs.cohort_month
GROUP BY cohort_month, cohort_size
ORDER BY cohort_month;
```

**Kết quả cuối cùng**:
```
cohort_month | cohort_size | month_0 | month_1 | month_2 | month_3
2024-01      | 3           | 100.0   | 66.7    | 33.3    | 0
2024-02      | 2           | 100.0   | 100.0   | 50.0    | NULL
2024-03      | 1           | 100.0   | NULL    | NULL    | NULL
```

---

## 🔑 Các khái niệm SQL quan trọng

### 1. DATE_TRUNC()
```sql
DATE_TRUNC('month', '2024-01-15') → '2024-01-01'
```
Làm tròn ngày về đầu tháng (hoặc năm, quý...)

### 2. DATE_PART()
```sql
DATE_PART('year', '2024-03-15') → 2024
DATE_PART('month', '2024-03-15') → 3
```
Trích xuất phần năm, tháng, ngày từ một date

### 3. Multiple CTEs
```sql
WITH
  cte1 AS (...),
  cte2 AS (...),
  cte3 AS (...)
SELECT ...
```
Tạo nhiều bảng tạm (Common Table Expressions) để chia nhỏ logic

### 4. CASE WHEN với MAX() để PIVOT
```sql
MAX(CASE WHEN month_number = 0 THEN retained_customers END) as month_0
```
- `CASE WHEN`: Nếu month_number = 0 thì lấy giá trị retained_customers, nếu không thì NULL
- `MAX()`: Vì đang GROUP BY cohort, nên cần aggregate function. MAX sẽ lấy giá trị duy nhất (hoặc NULL)

### 5. COUNT(DISTINCT)
```sql
COUNT(DISTINCT customer_id)
```
Đếm số lượng customer_id **duy nhất** (không trùng lặp)

---

## 📊 Phân tích kết quả

### Cohort tháng 1 (2024-01)
- **Cohort size**: 3 khách hàng
- **Month 0**: 100% → Cả 3 khách đều mua hàng ngay trong tháng đăng ký
- **Month 1**: 67% → 2/3 khách quay lại tháng sau (retention tốt)
- **Month 2**: 33% → Chỉ còn 1/3 khách (retention giảm mạnh)
- **Month 3**: 0% → Không còn ai quay lại

**Nhận xét**: Retention drop từ 67% → 33% ở tháng 2 là dấu hiệu cần quan tâm.

### Cohort tháng 2 (2024-02)
- **Cohort size**: 2 khách hàng
- **Month 0-1**: 100% → Retention hoàn hảo trong 2 tháng đầu
- **Month 2**: 50% → Giảm xuống nhưng vẫn cao hơn cohort tháng 1

**Nhận xét**: Cohort này có retention tốt hơn cohort tháng 1.

---

## 🎓 Ứng dụng thực tế

### 1. E-commerce
- Theo dõi khách hàng mua lại sau bao lâu
- So sánh hiệu quả các đợt flash sale

### 2. SaaS (Software as a Service)
- Đo lường subscription retention
- Tìm ra tháng nào người dùng dễ churn (rời bỏ) nhất

### 3. Social Media / Gaming
- Đo DAU/MAU (Daily/Monthly Active Users)
- Tìm ra cohort nào "sticky" (dính chặt) nhất

### 4. Marketing
- So sánh retention giữa các channels (Google Ads, Facebook, Organic)
- Tính LTV (Lifetime Value) của từng cohort

---

## 💪 Câu hỏi mở rộng

1. **Làm thế nào để tính retention theo tuần thay vì tháng?**
   - Thay `DATE_TRUNC('month', ...)` bằng `DATE_TRUNC('week', ...)`

2. **Làm sao để tính revenue retention thay vì user retention?**
   - Thay `COUNT(DISTINCT customer_id)` bằng `SUM(total)`

3. **Nếu muốn xem cohort theo nguồn traffic (source) thì sao?**
   - Thêm cột `source` vào `customer_cohort`
   - GROUP BY thêm `source`

4. **Làm thế nào để highlight cohort có retention tốt nhất?**
   - Thêm `RANK() OVER (ORDER BY month_1 DESC)` để xếp hạng

---

## 📝 Tóm tắt

**Cohort Analysis** giúp bạn:
1. Nhóm khách hàng theo thời gian đăng ký
2. Theo dõi % khách quay lại theo từng tháng
3. So sánh retention giữa các cohort
4. Tìm ra vấn đề trong customer journey

**Kỹ thuật SQL cần biết**:
- DATE_TRUNC, DATE_PART
- Multiple CTEs
- LEFT JOIN
- COUNT(DISTINCT)
- CASE WHEN + MAX() để PIVOT
- Window functions (nâng cao)

**Các công ty sử dụng**: Meta, Uber, Netflix, Spotify, Amazon...

---

Chúc bạn học tốt! 🚀
