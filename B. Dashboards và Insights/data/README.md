# Data Dictionary

Toàn bộ file CSV trong thư mục này tuân theo kiến trúc **Medallion 3 tầng** (Bronze → Silver → Gold).

---

## Sơ đồ luồng dữ liệu

```
[PostgreSQL]
     │
     ▼
  BRONZE (thô)
  sales_order, order_line, customer,
  product, product_line, product_group,
  product_price, province, fact_sales
     │
     ▼
  SILVER (chuẩn hóa địa lý)
  silver_province, silver_customer_geo
     │
     ▼
  GOLD (tích hợp, denormalized)
  gold_fact_sales  ← nguồn duy nhất cho 6 dashboard
     │
  email_log        ← nguồn riêng cho Dashboard 6
```

---

## GOLD Layer

### `gold_fact_sales.csv`
Bảng tích hợp chính – **25,754 dòng giao dịch**, denormalized toàn bộ chiều thời gian, sản phẩm, khách hàng và địa lý. Đây là nguồn dữ liệu duy nhất cho Dashboard 1–5, giúp Power BI truy vấn nhanh mà không cần JOIN.

| Cột quan trọng | Mô tả |
|---|---|
| `order_id` | Mã đơn hàng |
| `order_date` | Ngày đặt hàng |
| `YearMonth_Label` | Nhãn tháng dạng text (vd: "2025-T1") – dùng cho trục X |
| `YearMonth_Sort` | Số nguyên sort thứ tự tháng (vd: 202501) |
| `customer_id` | Mã đại lý |
| `province_name` | Tên tỉnh/thành đã chuẩn hóa |
| `region` | Vùng miền (Bắc / Trung / Nam) |
| `product_id` | Mã sản phẩm |
| `product_line` | Dòng xe |
| `product_group` | Nhóm xe (vd: Xe phổ thông, Xe thể thao nhôm) |
| `color` | Màu sắc |
| `quantity` | Số lượng bán |
| `unit_price` | Đơn giá |
| `revenue` | Doanh thu = quantity × unit_price |

---

## SILVER Layer

### `silver_province.csv`
Bảng tỉnh/thành đã chuẩn hóa tên và phân vùng miền. Dùng để join địa lý trong quá trình tạo Gold layer.

| Cột | Mô tả |
|---|---|
| `province_code` | Mã tỉnh chuẩn |
| `province_name` | Tên tỉnh chuẩn hóa |
| `region` | Miền Bắc / Miền Trung / Miền Nam |

### `silver_customer_geo.csv`
Bảng khách hàng đã được map tọa độ địa lý tỉnh/thành. Kết quả sau bước chuẩn hóa từ `customer.csv` + `silver_province.csv`.

| Cột | Mô tả |
|---|---|
| `customer_id` | Mã đại lý |
| `customer_name` | Tên đại lý |
| `province_name` | Tỉnh/thành đã chuẩn hóa |
| `region` | Vùng miền |
| `latitude` / `longitude` | Tọa độ địa lý tỉnh |

---

## BRONZE Layer

### `sales_order.csv`
Đơn hàng thô từ PostgreSQL.

| Cột | Mô tả |
|---|---|
| `order_id` | Mã đơn hàng (PK) |
| `customer_id` | Mã khách hàng/đại lý (FK) |
| `order_date` | Ngày đặt hàng |
| `status` | Trạng thái đơn |

### `order_line.csv`
Chi tiết từng dòng sản phẩm trong đơn hàng.

| Cột | Mô tả |
|---|---|
| `line_id` | Mã dòng đơn (PK) |
| `order_id` | Mã đơn hàng (FK) |
| `product_id` | Mã sản phẩm (FK) |
| `quantity` | Số lượng |
| `unit_price` | Đơn giá tại thời điểm đặt hàng |

### `fact_sales.csv`
Bảng fact giao dịch thô – chưa join địa lý và sản phẩm. Tiền thân của `gold_fact_sales`.

### `customer.csv`
Thông tin đại lý/khách hàng thô.

| Cột | Mô tả |
|---|---|
| `customer_id` | Mã đại lý (PK) |
| `customer_name` | Tên đại lý |
| `province` | Tỉnh/thành (raw, chưa chuẩn hóa) |
| `address` | Địa chỉ |

### `product.csv`
Danh mục sản phẩm chi tiết.

| Cột | Mô tả |
|---|---|
| `product_id` | Mã sản phẩm (PK) |
| `product_name` | Tên sản phẩm |
| `color` | Màu sắc |
| `product_line_id` | FK → product_line |

### `product_line.csv`
Dòng xe (vd: Xe New 26, Xe New 24).

| Cột | Mô tả |
|---|---|
| `product_line_id` | Mã dòng xe (PK) |
| `line_name` | Tên dòng xe |
| `product_group_id` | FK → product_group |

### `product_group.csv`
Nhóm xe cấp cao nhất (vd: Xe phổ thông, Xe thể thao nhôm).

| Cột | Mô tả |
|---|---|
| `product_group_id` | Mã nhóm (PK) |
| `group_name` | Tên nhóm sản phẩm |

### `product_price.csv`
Bảng giá theo thời gian (price history).

| Cột | Mô tả |
|---|---|
| `product_id` | FK → product |
| `price` | Giá bán |
| `effective_date` | Ngày áp dụng giá |

### `province.csv`
Bảng tỉnh/thành thô từ PostgreSQL – chưa chuẩn hóa tên và vùng miền.

---

## Log vận hành

### `email_log.csv`
Log xử lý email đặt hàng T3/2026 từ pipeline Hạng mục A. Nguồn dữ liệu cho **Dashboard 6 – Trạng thái vận hành**.

| Cột | Mô tả |
|---|---|
| `email_id` | Mã email |
| `received_at` | Thời điểm nhận |
| `status` | `success` / `error` / `pending` |
| `order_id` | Mã đơn hàng được tạo (nếu thành công) |
| `error_message` | Lý do lỗi (nếu có) |

**Kết quả T3/2026:** 1,132/1,132 emails xử lý thành công (100%), 0 lỗi.
