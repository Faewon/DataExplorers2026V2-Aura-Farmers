# 🚲 Xe Đạp Thống Nhất – Data Explorers Vòng 2
**Team: Aura Farmers**

---

## 📌 Tổng quan dự án

Dự án xây dựng hệ thống dữ liệu toàn trình cho **Xe Đạp Thống Nhất (Thống Nhất Bike)** – doanh nghiệp sản xuất & phân phối xe đạp với hơn 200 SKU, 5 nhóm sản phẩm và mạng lưới **798 đại lý** trên toàn quốc.

Trước dự án, doanh nghiệp không có dashboard quản trị, không có hệ thống dự báo tự động, và toàn bộ đơn hàng phải nhập tay vào ERP. Dự án giải quyết 3 bài toán lớn:

| Hạng mục | Mô tả |
|---|---|
| **A – Vận hành** | Pipeline ETL tự động xử lý đơn hàng từ email/PDF |
| **B – Phân tích** | 6 Dashboard Power BI + 6 Insights kinh doanh |
| **C – Dự báo** | Dự báo nhu cầu Q2/2026 + Ma trận chiến lược |

### Dữ liệu tổng thể (T1/2025 – T3/2026)

| Chỉ số | Giá trị |
|---|---|
| Tổng doanh thu | 109 tỷ đồng |
| Số đơn hàng | 3,000+ |
| Sản phẩm bán | 72,000 |
| Đại lý hoạt động | 807 |
| Email T3/2026 xử lý | 1,132 / 1,132 (100%) |

---

## 🏗️ Kiến trúc hệ thống

Hệ thống theo kiến trúc **Medallion 3 tầng**:

```
Bronze  →  Silver  →  Gold
(raw)      (clean)    (analytics-ready)
```

- **Bronze**: Dữ liệu thô từ PostgreSQL, lưu nguyên trạng
- **Silver**: Chuẩn hóa địa lý, làm sạch tên khách hàng (797/798 khách hàng mapped)
- **Gold**: `gold_fact_sales` – 25,754 dòng, nguồn duy nhất cho 6 dashboard

### Tech Stack

| Thành phần | Công nghệ |
|---|---|
| Orchestration | Apache Airflow 2.8.1 |
| Database | PostgreSQL 13 |
| Ngôn ngữ | Python 3.11 |
| Containerization | Docker Compose (5 services) |
| PDF parsing | pdfplumber |
| BI / Dashboard | Power BI |

---

## 📁 Cấu trúc thư mục

```
.
├── A. Xử lý đơn hàng tự động/
│   ├── airflow/dags/
│   │   └── email_pipeline_dag.py        # DAG chính xử lý email
│   ├── init-scripts/
│   │   ├── 01_create_tables.sql
│   │   ├── 02_import_data.sql
│   │   ├── 03_create_email_log.sql
│   │   ├── 04_create_silver_tables.sql
│   │   └── 05_create_gold_tables.sql
│   ├── src/
│   │   ├── extract.py                   # Đọc MIME, trích xuất PDF
│   │   ├── validators.py                # Kiểm tra hợp lệ đơn hàng
│   │   ├── loaders.py                   # Ghi vào PostgreSQL
│   │   ├── router.py                    # Phân loại file processed/failed
│   │   ├── silver.py                    # Chuẩn hóa Silver layer
│   │   └── warehouse.py                 # Refresh fact tables
│   ├── converter.py
│   ├── docker-compose.yml
│   ├── dockerfile.airflow
│   ├── requirements.txt
│   └── tnbike_database_schema.md        # Schema toàn bộ database
│
├── B. Dashboards và Insights/
│   ├── dashboard/
│   │   ├── Tổng Quan Kinh Doanh.jpg
│   │   ├── Phân Tích Thời Gian.jpg
│   │   ├── Phân Tích Sản Phẩm.jpg
│   │   ├── Phân Tích Đại Lý.jpg
│   │   ├── Phân Tích Địa Lý.jpg
│   │   ├── Trạng Thái Vận Hành.jpg
│   │   └── PowerBIDashboard.pbix        # File Power BI gốc
│   └── docs/
│       ├── BaoCaoKyThuat_HangMucB.md
│       ├── Insights.md
│       ├── DataExplorers2026 – Đề thi Vòng 2.pdf
│       └── README.md
│
├── C. Dự báo nhu cầu và Chiến lược/
│   ├── Part_3.ipynb                     # Notebook phân tích & dự báo
│   ├── Readme.md
│   └── requirements.txt
│
├── Data/                                # Dữ liệu nguồn
│   ├── customer.csv
│   ├── email_log.csv
│   ├── fact_sales.csv
│   ├── gold_fact_sales.csv
│   ├── order_line.csv
│   ├── product.csv / product_group.csv / product_line.csv / product_price.csv
│   ├── province.csv
│   ├── sales_order.csv
│   ├── silver_customer_geo.csv
│   └── silver_province.csv
│
├── .gitignore
├── DataExplorers2026 – Đề thi Vòng 2.pdf
├── requirements.txt
└── README.md                            ← (file này)
```

---

## 🅰️ Hạng mục A – Xử lý đơn hàng tự động

### Kết quả

| Chỉ số | Giá trị |
|---|---|
| Tổng email nhận | 1,132 |
| Xử lý thành công | 1,132 (100%) |
| Lỗi | 0 |
| Tỷ lệ vượt kiểm tra hợp lệ | 100% |

### DAG: `email_order_pipeline`

Xử lý song song tối đa **8 mapped tasks**:

```
list_eml_files
    → process_single_email (×N, parallel)
        → summarize_results
        → run_silver_layer
        → refresh_fact_sales
        → refresh_gold_fact_sales
```

### Luồng ETL từng email

```
.eml → đọc metadata → trích PDF (MIME)
     → parse header PDF → đọc bảng hàng (pdfplumber)
     → validate (qty × price = total, tổng khớp email body)
     → đối chiếu product master → upsert sales_order
     → xóa & ghi lại order_line (idempotent)
     → ghi email_log → route → processed/ hoặc failed/
```

### Các module chính

| Module | Chức năng |
|---|---|
| `extract.py` | Decode MIME headers (RFC 2047), trích PDF, đọc bảng bằng pdfplumber |
| `validators.py` | Kiểm tra trường bắt buộc, công thức dòng, tổng đơn hàng, product master |
| `loaders.py` | Upsert `sales_order`, replace `order_line`, upsert `email_log` |
| `router.py` | Route file → `processed/` hoặc `failed/`, dọn PDF tạm |
| `silver.py` | Chuẩn hóa tên khách hàng (loại bỏ nhãn thừa) |

### Cài đặt & chạy

```bash
# Khởi động toàn bộ stack
docker compose up -d

# Kích hoạt DAG (manual trigger)
airflow dags trigger email_order_pipeline

# Kiểm tra kết quả
psql -U postgres -d tnbike -c "SELECT COUNT(*) FROM email_log WHERE status='success';"
```

**Yêu cầu:** Docker, Docker Compose. File `.eml` đặt vào `maildata/incoming/`.

---

## 🅱️ Hạng mục B – Dashboards & Insights

### 6 Dashboard Power BI (nguồn: `gold_fact_sales`)

| # | Dashboard | Nội dung chính |
|---|---|---|
| 1 | Tổng Quan Kinh Doanh | 4 KPI: 109 tỷ / 3K+ đơn / 72K SP / 807 đại lý |
| 2 | Phân Tích Thời Gian | Xu hướng MoM, YoY theo nhóm SP |
| 3 | Phân Tích Sản Phẩm | Cơ cấu doanh thu, BCG Matrix, màu sắc |
| 4 | Phân Tích Đại Lý | RFM segmentation, Pareto, Cohort Retention |
| 5 | Phân Tích Địa Lý | Heatmap tỉnh/vùng, top tỉnh, tăng trưởng |
| 6 | Trạng Thái Vận Hành | Pipeline status, email log, xử lý theo ngày |

> **Lưu ý kỹ thuật:** Dữ liệu có gián đoạn (Q1/2025 và Q1/2026). Dùng `YearMonth_Label` (text) + `YearMonth_Sort` (int) thay `PREVIOUSMONTH()` để tránh lỗi trục thời gian.

### 6 Insights kinh doanh nổi bật

1. **Mùa vụ tháng 3 có thể dự báo** – T3/2026 đạt 40.8 tỷ (+110% MoM), pattern lặp lại 2 năm → cần chuẩn bị tồn kho từ tháng 1.
2. **Rủi ro tập trung doanh thu** – Top 20% (~159 đại lý) chiếm 68% doanh thu; 280 đại lý At Risk có thể ảnh hưởng 15–20% doanh thu.
3. **Miền Nam bị bỏ ngỏ** – Chỉ 5.5% (~6 tỷ), TP.HCM đứng thứ 7. Cơ hội lớn cần khai thác.
4. **Xe thể thao mất thị phần** – Nhôm -65%, Thép -38% YoY khi toàn thị trường tăng +120%.
5. **Chất lượng đại lý mới 2026 cải thiện** – Cohort 2026-T1 retention T+2 đạt 74.5% vs 45.7% của 2025-T1.
6. **Quảng Ngãi tăng ~11x** – Tăng trưởng cao nhất hệ thống; cần nhân rộng sang Bình Định, Quảng Nam.

---

## 🅲 Hạng mục C – Dự báo & Chiến lược

### Kết quả dự báo Q2/2026

**Tổng doanh thu dự báo: ~65–75 tỷ VND**

| Nhóm SP | Q1/2026 (tỷ) | Dự báo Q2 (tỷ) | Tỷ trọng |
|---|---|---|---|
| Xe phổ thông | 43.06 | ~42–49 | ~54% |
| Xe trẻ em nhóm 1 | 9.21 | ~8–10 | ~11% |
| Xe thể thao thép | 2.97 | ~3–4 | ~4% |
| Xe thể thao nhôm | 1.94 | ~1.5–2.5 | ~2% |
| Xe trẻ em nhóm 2 | 2.20 | ~2–3 | ~3% |

### Ma trận 6 hành động chiến lược ưu tiên

| # | Hành động | KPI mục tiêu |
|---|---|---|
| 1 | Chuẩn bị tồn kho Q3 từ tháng 6 | Không stock-out top 5 dòng trong T3/2027 |
| 2 | Tái kích hoạt 100 đại lý At Risk đầu Q2 | Active dealers ≥ 450 cuối Q2/2026 |
| 3 | Mở rộng Miền Trung: Nghệ An, Hà Tĩnh, Quảng Ninh | +20 đại lý Miền Trung trong Q2 |
| 4 | Đẩy dòng Stars: Xe GN 06-24 2.0, Xe Nữ, Xe Bunny 16 | Stars đạt 35% tổng doanh thu |
| 5 | Tối ưu danh mục màu – rút về 15 màu chủ lực | Giảm 15% chi phí tồn kho |
| 6 | EOL các dòng BCG Dogs giảm >80% | Giải phóng capacity cho Stars |

### Chạy notebook dự báo

```bash
cd "C. Dự báo nhu cầu và Chiến lược"
pip install -r requirements.txt
jupyter notebook Part_3.ipynb
```

---

## 🗄️ Database Schema

Toàn bộ schema xem tại: `A. Xử lý đơn hàng tự động/tnbike_database_schema.md`

**Bảng chính:**

| Bảng | Mô tả |
|---|---|
| `sales_order` | 1,132 đơn hàng T3/2026 |
| `order_line` | Chi tiết từng dòng hàng |
| `email_log` | Log xử lý 1,132 email (100% success) |
| `silver_customer_geo` | 797/798 khách hàng có ánh xạ địa lý |
| `silver_province` | 34 tỉnh/thành đã chuẩn hóa |
| `gold_fact_sales` | 25,754 dòng – nguồn analytics duy nhất |

---

## ✅ Tổng kết thành quả

| Hạng mục | Kết quả |
|---|---|
| A – Vận hành | 1,132/1,132 đơn xử lý thành công, 0 lỗi, Airflow song song 8 task ổn định |
| B – Phân tích | 6 dashboard Power BI, 6 insights có giá trị hành động rõ ràng |
| C – Dự báo | Dự báo Q2/2026 ~65–75 tỷ, ma trận 6 hành động chiến lược |

**KPI mục tiêu Q2/2026:** Doanh thu 120 tỷ · Sản lượng 80K · Tỷ trọng xe phổ thông ≤68% · At Risk ≤250 đại lý.

---

*Team Aura Farmers – Data Explorers Vòng 2, 2026*
