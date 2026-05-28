# DataExplorers 2026 – Hạng mục B: Business Intelligence Dashboard

Hệ thống phân tích kinh doanh đa chiều trên **Power BI** cho **Thống Nhất Bike** — sản xuất & phân phối xe đạp B2B, 200+ SKU, 798 đại lý toàn quốc.

> Tích hợp trực tiếp dữ liệu T3/2026 từ pipeline Hạng mục A · 6 dashboard · RFM & Cohort Analysis thuần DAX

---

## Số liệu tổng quan

| Chỉ số | Giá trị |
|---|---|
| Thời gian dữ liệu | T1/2025 – T3/2026 |
| Tổng doanh thu | 109 tỷ đồng |
| Số đơn hàng | 3,000+ |
| Sản phẩm bán | 72,000 |
| Đại lý hoạt động | 807 |
| Đơn T3/2026 từ HM-A | 1,132 · xử lý thành công 100% |

---

## Cấu trúc repo

```
├── dashboard/
│   ├── PowerBIDashboard.pbix              # File Power BI chính
│   ├── Tổng Quan Kinh Doanh.jpg           # Dashboard 1
│   ├── Phân Tích Thời Gian.jpg            # Dashboard 2
│   ├── Phân Tích Sản Phẩm.jpg             # Dashboard 3
│   ├── Phân Tích Đại Lý.jpg               # Dashboard 4
│   ├── Phân Tích Địa Lý.jpg               # Dashboard 5
│   └── Trạng Thái Vận Hành.jpg            # Dashboard 6
│
├── data/
│   ├── README.md                          # Data dictionary – schema từng file CSV
│   ├── gold_fact_sales.csv                # [GOLD]   Bảng tích hợp chính · 25,754 dòng
│   ├── silver_customer_geo.csv            # [SILVER] Khách hàng + tọa độ địa lý
│   ├── silver_province.csv                # [SILVER] Tỉnh/thành chuẩn hóa
│   ├── fact_sales.csv                     # [BRONZE] Giao dịch thô
│   ├── sales_order.csv                    # [BRONZE] Đơn hàng
│   ├── order_line.csv                     # [BRONZE] Chi tiết dòng đơn
│   ├── customer.csv                       # [BRONZE] Đại lý/khách hàng
│   ├── product.csv                        # [BRONZE] Danh mục sản phẩm
│   ├── product_line.csv                   # [BRONZE] Dòng xe
│   ├── product_group.csv                  # [BRONZE] Nhóm xe
│   ├── product_price.csv                  # [BRONZE] Bảng giá
│   ├── province.csv                       # [BRONZE] Tỉnh/thành thô
│   └── email_log.csv                      # Log xử lý email T3/2026 · nguồn Dashboard 6
│
├── docs/
│   ├── BaoCaoKyThuat_HangMucB.md          # Báo cáo kỹ thuật đầy đủ
│   └── Insights.md                        # 6 insights · Phát hiện → Ý nghĩa → Khuyến nghị
│
├── DataExplorers2026 – Đề thi Vòng...     # Đề bài gốc
└── README.md
```

---

## Kiến trúc dữ liệu – Medallion 3 tầng

```
[PostgreSQL]
     │
     ▼
  BRONZE ── sales_order, order_line, customer,
             product, product_line, product_group,
             product_price, province, fact_sales
     │
     ▼
  SILVER ── silver_province, silver_customer_geo
     │
     ▼
  GOLD   ── gold_fact_sales  (25,754 dòng · nguồn duy nhất cho 6 dashboard)
             email_log       (nguồn riêng cho Dashboard 6)
```

---

## 6 Dashboard

| # | Tên | Câu hỏi kinh doanh |
|---|---|---|
| 1 | Tổng Quan Kinh Doanh | Tình hình tổng thể? Trạng thái pipeline T3/2026? |
| 2 | Phân Tích Thời Gian | Xu hướng doanh số? Mùa cao điểm? YoY Q1? |
| 3 | Phân Tích Sản Phẩm | Nhóm nào dẫn đầu? BCG Stars/Dogs là ai? Màu nào thống trị? |
| 4 | Phân Tích Đại Lý | RFM 5 segment? Pareto 20/80? Cohort retention? |
| 5 | Phân Tích Địa Lý | Thị trường nào lớn nhất? Tỉnh nào tăng/giảm bất thường? |
| 6 | Trạng Thái Vận Hành | Pipeline email T3/2026: thành công / lỗi / chờ? |

---

## Insights nổi bật

Chi tiết đầy đủ tại [`docs/Insights.md`](./docs/Insights.md).

| # | Insight | Số liệu chính |
|---|---|---|
| 1 | Mùa vụ tháng 3 có thể dự báo được | T3/2026 đạt 40.8 tỷ · +110% MoM · đỉnh tuần 10–13 |
| 2 | Rủi ro tập trung doanh thu báo động | Top 20% đại lý chiếm 68% · 280 At Risk → nguy cơ -15~20% |
| 3 | Thị trường miền Nam bị bỏ ngỏ | Miền Nam 5.5% · TP.HCM đứng thứ 7 (~5.8 tỷ) |
| 4 | Xe thể thao mất thị phần ngược chiều thị trường | Nhôm -65% · Thép -38% · toàn thị trường +120% |
| 5 | Chất lượng đại lý mới 2026 cải thiện rõ rệt | Retention T+2: 74.5% (2026) vs 45.7% (2025) |
| 6 | Quảng Ngãi tăng ~11x – cơ hội nhân rộng miền Trung | Hà Tĩnh ~4x · Quảng Ninh ~3.5x |

---

## KPI thực tế & mục tiêu

| KPI | Q1/2026 | Mục tiêu Q2/2026 |
|---|---|---|
| Doanh thu | 109 tỷ | 120 tỷ |
| Sản lượng | 72K | 80K |
| Doanh thu TB/đại lý | 137 triệu | 150 triệu |
| Tỷ trọng xe phổ thông | 70.15% | ≤68% |
| Đại lý At Risk | 280 (35%) | ≤250 |
| Tỷ lệ xử lý email | 100% | ≥98% |

---

## Công nghệ

| Công nghệ | Vai trò |
|---|---|
| Power BI | Visualization, DAX measures, Calculated Columns |
| PostgreSQL | Nguồn dữ liệu gốc |
| Medallion Architecture | Tổ chức Bronze → Silver → Gold |
| RFM Analysis | Xây dựng thuần DAX – không dùng Python |
| Cohort Analysis | Xây dựng thuần DAX – không dùng Python |
