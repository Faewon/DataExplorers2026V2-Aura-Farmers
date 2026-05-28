# 📂 Data – Mô tả dữ liệu

Thư mục chứa toàn bộ dữ liệu xuất ra từ hệ thống pipeline của dự án **Xe Đạp Thống Nhất – Data Explorers Vòng 2**.  
Phạm vi thời gian: **T1/2025 – T3/2026**.

---

## Sơ đồ quan hệ tổng quan

```
province ◄──────────────────────── customer
                                       │
                              sales_order (BH25.xxxx / BH26.xxxx)
                                       │
                              order_line ──► product ──► product_line ──► product_group
                                       │              └──► product_price
                              email_log (T3/2026 only)

                    ── Silver layer ──
silver_province ◄── silver_customer_geo (ánh xạ customer → tỉnh chuẩn hóa)

                    ── Analytics layer ──
fact_sales          (schema legacy – join với province gốc)
gold_fact_sales     (schema mới – join với silver_province, nguồn cho Power BI)
```

---

## Danh sách file

| File | Số dòng | Mô tả |
|---|---|---|
| `customer.csv` | 798 | Danh sách đại lý |
| `sales_order.csv` | 2,759 | Đơn hàng tổng hợp |
| `order_line.csv` | 25,754 | Chi tiết từng dòng hàng |
| `product.csv` | 265 | Danh mục sản phẩm (SKU) |
| `product_line.csv` | 77 | Dòng xe |
| `product_group.csv` | 5 | Nhóm sản phẩm |
| `product_price.csv` | 1,016 | Lịch sử giá theo sản phẩm |
| `province.csv` | 75 | Bảng tỉnh/thành (legacy) |
| `email_log.csv` | 1,132 | Log xử lý email T3/2026 |
| `silver_province.csv` | 34 | Bảng tỉnh/thành chuẩn hóa (Silver) |
| `silver_customer_geo.csv` | 797 | Ánh xạ địa lý đại lý (Silver) |
| `fact_sales.csv` | 25,754 | Bảng fact tổng hợp (schema legacy) |
| `gold_fact_sales.csv` | 25,754 | Bảng fact Gold – nguồn chính cho dashboard |

---

## Chi tiết từng file

### `customer.csv` – Danh sách đại lý
**798 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `customer_code` | string | Mã đại lý, định dạng `KH-XXXXX` (PK) |
| `customer_name` | string | Tên đại lý / hộ kinh doanh / công ty |
| `tax_code` | int | Mã số thuế |
| `address` | string | Địa chỉ đầy đủ *(1 dòng NULL)* |
| `province_id` | int | FK → `province.province_id` *(97 dòng NULL – đại lý chưa map tỉnh)* |
| `customer_tier` | string | Phân hạng khách hàng (`STANDARD`, ...) |
| `is_active` | bool | Trạng thái hoạt động |
| `created_at` | datetime | Thời điểm tạo |
| `updated_at` | datetime | Thời điểm cập nhật |

> **Lưu ý:** 97 dòng `province_id = NULL` là các đại lý chưa được ánh xạ địa lý trong bảng legacy. Bảng `silver_customer_geo` xử lý vấn đề này với tỷ lệ map đạt 797/798 (99.9%).

---

### `sales_order.csv` – Đơn hàng
**2,759 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `order_id` | int | PK tự tăng |
| `so_number` | string | Số chứng từ, định dạng `BH25.XXXX` / `BH26.XXXX` |
| `invoice_symbol` | float | Ký hiệu hóa đơn *(NULL cho 1,132 đơn T3/2026 – chưa xuất hóa đơn)* |
| `invoice_number` | float | Số hóa đơn *(NULL tương tự)* |
| `order_date` | date | Ngày đặt hàng |
| `customer_code` | string | FK → `customer.customer_code` |
| `total_amount` | float | Tổng tiền đơn hàng (VND) |
| `total_quantity` | int | Tổng số lượng sản phẩm |
| `line_count` | int | Số dòng hàng trong đơn |
| `fiscal_year` | int | Năm tài chính |
| `fiscal_month` | int | Tháng tài chính (1–12) |
| `fiscal_quarter` | int | Quý tài chính (1–4) |
| `created_at` | datetime | Thời điểm ghi vào DB |

> **Lưu ý:** `invoice_symbol` và `invoice_number` đều NULL cho toàn bộ 1,132 đơn T3/2026 do pipeline tự động chưa tích hợp xuất hóa đơn.

---

### `order_line.csv` – Chi tiết dòng hàng
**25,754 dòng** – không có giá trị NULL

| Cột | Kiểu | Mô tả |
|---|---|---|
| `line_id` | int | PK tự tăng |
| `order_id` | int | FK → `sales_order.order_id` |
| `so_number` | string | Số chứng từ (denormalized, tiện join) |
| `product_code` | int | FK → `product.product_code` |
| `quantity` | float | Số lượng |
| `unit_price` | float | Đơn giá (VND) |
| `line_total` | float | Thành tiền = `quantity × unit_price` |
| `created_at` | datetime | Thời điểm ghi |

---

### `product.csv` – Danh mục sản phẩm (SKU)
**265 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `product_code` | int | PK (mã 12 chữ số) |
| `product_name` | string | Tên đầy đủ sản phẩm |
| `line_id` | int | FK → `product_line.line_id` *(90 dòng NULL – SKU chưa phân dòng)* |
| `color` | string | Màu sắc *(18 dòng NULL)* |
| `unit` | string | Đơn vị tính (`Chiếc`) |
| `is_active` | bool | Còn kinh doanh hay không |
| `created_at` | datetime | Thời điểm tạo |

---

### `product_line.csv` – Dòng xe
**77 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `line_id` | int | PK |
| `line_name` | string | Tên dòng xe (vd: `Xe New 26`, `Xe GN 05-26`) |
| `group_code` | string | FK → `product_group.group_code` |
| `created_at` | datetime | Thời điểm tạo |

---

### `product_group.csv` – Nhóm sản phẩm
**5 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `group_code` | string | PK |
| `group_name` | string | Tên nhóm |
| `description` | string | Mô tả chi tiết |
| `created_at` | datetime | Thời điểm tạo |

**5 nhóm sản phẩm:**

| group_code | group_name |
|---|---|
| `CITYBIKE_P` | Xe phổ thông |
| `KIDBIKE_1` | Xe trẻ em nhóm 1 |
| `KIDBIKE_2` | Xe trẻ em nhóm 2 |
| `SPORTBIKE_S` | Xe thể thao thép |
| `SPORTBIKE_A` | Xe thể thao nhôm |

---

### `product_price.csv` – Lịch sử giá
**1,016 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `price_id` | int | PK |
| `product_code` | int | FK → `product.product_code` |
| `unit_price` | float | Đơn giá (VND) |
| `effective_from` | date | Ngày giá có hiệu lực |
| `effective_to` | float | Ngày hết hiệu lực *(toàn bộ 1,016 dòng NULL = giá đang hiện hành)* |
| `created_at` | datetime | Thời điểm tạo |

> **Lưu ý:** `effective_to = NULL` trên toàn bộ bảng nghĩa là chưa có lịch sử thay đổi giá – mỗi sản phẩm chỉ có một mức giá hiện tại.

---

### `province.csv` – Bảng tỉnh/thành (legacy)
**75 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `province_id` | int | PK |
| `province_name` | string | Tên tỉnh/thành |
| `region` | string | Vùng: `Miền Bắc`, `Miền Trung`, `Miền Nam` |
| `created_at` | datetime | Thời điểm tạo |

> **Lưu ý:** Bảng legacy gốc, có 75 bản ghi (bao gồm tên trùng lặp hoặc không chuẩn hóa). Dùng `silver_province` cho phân tích địa lý chính xác.

---

### `email_log.csv` – Log xử lý email
**1,132 dòng** – không có giá trị NULL

| Cột | Kiểu | Mô tả |
|---|---|---|
| `id` | int | PK |
| `message_id` | string | Message-ID email (RFC 5322) |
| `from_address` | string | Địa chỉ email người gửi (kèm tên hiển thị) |
| `received_at` | datetime | Thời điểm email được nhận |
| `attachment_name` | string | Tên file PDF đính kèm |
| `processing_status` | string | Trạng thái xử lý (`Success` / `Failed`) |
| `created_at` | datetime | Thời điểm ghi log |

> **Phạm vi:** Chỉ chứa email T3/2026 (1,132 email, tỷ lệ `Success` = 100%).

---

### `silver_province.csv` – Bảng tỉnh/thành chuẩn hóa
**34 dòng**

| Cột | Kiểu | Mô tả |
|---|---|---|
| `province_code` | int | PK (mã chuẩn hóa) |
| `province_name` | string | Tên tỉnh/thành chuẩn hóa |
| `province_type` | string | Loại: `Thành phố`, `Tỉnh` |
| `region` | string | Vùng: `Miền Bắc`, `Miền Trung`, `Miền Nam` |
| `source` | string | Nguồn danh mục |
| `is_active` | bool | Đang hoạt động |
| `created_at` | datetime | Thời điểm tạo |

> **Khác biệt so với `province.csv`:** Đã hợp nhất về 34 tỉnh/thành chuẩn (loại bỏ trùng lặp, chuẩn hóa tên). Đây là bảng địa lý dùng cho toàn bộ dashboard và phân tích.

---

### `silver_customer_geo.csv` – Ánh xạ địa lý đại lý
**797 dòng** (797/798 đại lý – 1 đại lý không map được)

| Cột | Kiểu | Mô tả |
|---|---|---|
| `customer_code` | string | FK → `customer.customer_code` (PK của bảng này) |
| `province_code` | int | FK → `silver_province.province_code` |
| `source_address` | string | Địa chỉ nguồn dùng để map |
| `legacy_province_id` | int | Province ID trong bảng legacy *(96 dòng NULL – đại lý không có province_id gốc)* |
| `legacy_province_name` | string | Tên tỉnh legacy *(96 dòng NULL tương tự)* |
| `matched_text` | string | Chuỗi text được nhận diện khi map |
| `match_method` | string | Phương pháp map (`address`) |
| `confidence` | string | Độ tin cậy (`HIGH`) |
| `created_at` | datetime | Thời điểm tạo |
| `updated_at` | datetime | Thời điểm cập nhật |

> **Lưu ý:** Toàn bộ 797 bản ghi hiện tại đều có `confidence = HIGH` và `match_method = address`.

---

### `fact_sales.csv` – Bảng fact (schema legacy)
**25,754 dòng**

Bảng wide tổng hợp, kết hợp thông tin từ `sales_order`, `order_line`, `product`, `product_line`, `product_group`, và `province` (legacy).

| Cột | Kiểu | Mô tả |
|---|---|---|
| `fact_id` | int | PK |
| `order_date` | date | Ngày đặt hàng |
| `fiscal_year/quarter/month` | int | Thông tin kỳ tài chính |
| `week_of_year` | int | Tuần trong năm |
| `so_number` | string | Số chứng từ |
| `order_id` | int | FK → `sales_order` |
| `line_id` | int | FK → `order_line` |
| `customer_code` | string | Mã đại lý |
| `customer_name` | string | Tên đại lý |
| `province_id` | int | Tỉnh (legacy) *(1,275 NULL)* |
| `province_name` | string | Tên tỉnh (legacy) |
| `region` | string | Vùng (legacy) |
| `product_code` | int | Mã sản phẩm |
| `product_name` | string | Tên sản phẩm |
| `color` | string | Màu sắc *(81 NULL)* |
| `line_id_fk` | int | FK → `product_line` *(5,355 NULL)* |
| `line_name` | string | Tên dòng xe *(5,355 NULL)* |
| `group_code` | string | Mã nhóm SP *(5,355 NULL)* |
| `group_name` | string | Tên nhóm SP *(5,355 NULL)* |
| `quantity` | float | Số lượng |
| `unit_price` | float | Đơn giá |
| `line_total` | float | Thành tiền |

> **Không khuyến nghị dùng cho phân tích mới.** Dùng `gold_fact_sales` thay thế.

---

### `gold_fact_sales.csv` – Bảng fact Gold ⭐
**25,754 dòng** – nguồn chính cho toàn bộ 6 dashboard Power BI

Kế thừa `fact_sales` và bổ sung cột địa lý Silver (chuẩn hóa cao hơn). Có 2 bộ cột địa lý song song để so sánh:

**Địa lý Legacy (giữ nguyên):**

| Cột | Mô tả |
|---|---|
| `legacy_province_id` | Province ID gốc *(1,275 NULL)* |
| `legacy_province_name` | Tên tỉnh gốc |
| `legacy_region` | Vùng gốc |

**Địa lý Silver (chuẩn hóa):**

| Cột | Mô tả |
|---|---|
| `silver_province_code` | FK → `silver_province.province_code` *(112 NULL)* |
| `silver_province_name` | Tên tỉnh chuẩn hóa |
| `silver_province_type` | `Thành phố` / `Tỉnh` |
| `silver_region` | Vùng chuẩn hóa |
| `geo_match_method` | Phương pháp match |
| `geo_confidence` | Độ tin cậy |

**Cột sản phẩm & doanh thu:** giống `fact_sales`.

> **Khuyến nghị:** Luôn dùng cột `silver_*` cho phân tích địa lý. Cột `legacy_*` chỉ dùng để kiểm tra/debug.

---

## Lưu ý chung về chất lượng dữ liệu

| Vấn đề | Bảng | Số dòng | Ghi chú |
|---|---|---|---|
| `province_id = NULL` | `customer` | 97 | Đại lý không có tỉnh legacy → đã xử lý qua `silver_customer_geo` |
| `line_id/group_code = NULL` | `fact_sales`, `gold_fact_sales` | 5,355 | SKU chưa phân dòng xe → không ảnh hưởng tổng doanh thu |
| `silver_province = NULL` | `gold_fact_sales` | 112 | Đại lý không map được tỉnh Silver |
| `invoice_symbol/number = NULL` | `sales_order` | 1,132 | Đơn T3/2026 chưa xuất hóa đơn |
| `effective_to = NULL` | `product_price` | 1,016 (100%) | Chưa có lịch sử thay đổi giá |
| Đại lý thiếu địa lý | `silver_customer_geo` | 1 | 1/798 đại lý không map được |

---

*Data Explorers Vòng 2 – Team Aura Farmers, 2026*
