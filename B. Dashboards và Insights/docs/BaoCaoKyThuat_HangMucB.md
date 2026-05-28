# Báo Cáo Kỹ Thuật – Hạng Mục B
## Hệ Thống Phân Tích Kinh Doanh Power BI – Thống Nhất Bike

---

## 1. TỔNG QUAN HỆ THỐNG

### 1.1 Bối cảnh & Vấn đề doanh nghiệp

Thống Nhất Bike là doanh nghiệp sản xuất và phân phối xe đạp B2B với hơn 200 SKU, phân phối qua 798 đại lý trên toàn quốc. Trước khi có hệ thống này, doanh nghiệp không có dashboard quản trị, không có dự báo tự động, và không có cảnh báo sớm về biến động hành vi đại lý.

Hạng mục B xây dựng hệ thống phân tích kinh doanh đa chiều trên Power BI, tích hợp trực tiếp dữ liệu T3/2026 từ pipeline Hạng mục A, trả lời đầy đủ 4 nhóm câu hỏi kinh doanh theo yêu cầu đề bài: thời gian, sản phẩm, đại lý và địa lý.

### 1.2 Phạm vi dữ liệu

| Thông số | Giá trị |
|---|---|
| Nguồn dữ liệu chính | `gold_fact_sales` (Gold Layer – Medallion Architecture) |
| Bảng phụ | `email_log` (Dashboard 6), `silver_province` (địa lý) |
| Thời gian | T1/2025 – T3/2026 |
| Tổng doanh thu | **109 tỷ đồng** |
| Số đơn hàng | **3,000+ đơn** |
| Tổng số lượng bán | **72,000 sản phẩm** |
| Số đại lý hoạt động | **807 đại lý** |
| Dữ liệu bổ sung từ Hạng mục A | **1,132 đơn T3/2026** (tỷ lệ xử lý thành công 100%) |

### 1.3 Kiến trúc dữ liệu – Medallion 3 tầng

Hệ thống áp dụng kiến trúc Medallion để tổ chức dữ liệu theo 3 tầng chất lượng tăng dần:

- **Bronze:** Dữ liệu thô từ PostgreSQL — `sales_order`, `order_line`, `customer`, `product`, `product_line`, `product_group`
- **Silver:** Dữ liệu đã chuẩn hóa địa lý — `silver_province`, `silver_customer_geo`
- **Gold:** Bảng denormalized tích hợp — `gold_fact_sales` (25,754 dòng giao dịch)

`gold_fact_sales` là nguồn dữ liệu duy nhất cho toàn bộ 6 dashboard, tích hợp sẵn chiều thời gian, sản phẩm, khách hàng và địa lý — giúp Power BI truy vấn nhanh mà không cần JOIN phức tạp.

### 1.4 Xử lý vấn đề dữ liệu gián đoạn

Dữ liệu chỉ có Q1/2025 và Q1/2026 — nếu dùng Date Table liên tục hoặc hàm `PREVIOUSMONTH()` mặc định của Power BI, trục thời gian sẽ hiện 12 điểm với 9 tháng trống ở giữa, gây hiểu nhầm phân tích.

Nhóm xử lý bằng cách tạo cột `YearMonth_Label` (dạng text categorical như "2025-T1") và sort theo `YearMonth_Sort` (số nguyên như 202501). Kết quả trục X chỉ hiện đúng 6 điểm thực tế theo thứ tự thời gian. Toàn bộ DAX time-intelligence dùng `FILTER + so sánh số nguyên` thay `PREVIOUSMONTH()` để tránh lỗi với dữ liệu gián đoạn.

### 1.5 Bộ KPI bắt buộc

Nhóm định nghĩa, tính toán và đặt mục tiêu cho 6 nhóm KPI theo yêu cầu đề bài:

| Nhóm KPI | Chỉ số | Thực tế Q1/2026 | Mục tiêu Q2/2026 |
|---|---|---|---|
| Sản lượng | Tổng số lượng bán / Số đơn hàng | 72K / 3K | 80K / 3,300 |
| Doanh thu | Tổng doanh thu / Doanh thu TB/đại lý | 109 tỷ / 137 triệu | 120 tỷ / 150 triệu |
| Tăng trưởng | YoY Q1 (xe phổ thông) | +148% | Duy trì >100% |
| Cơ cấu sản phẩm | Tỷ trọng xe phổ thông | 70.15% | ≤68% |
| Khách hàng/Đại lý | Champions / At Risk | 41 (5%) / 280 (35%) | 20 / ≤250 |
| Hiệu quả vận hành | Tỷ lệ xử lý email thành công | 100% | ≥98% |

---

## 2. DASHBOARD 1 – TỔNG QUAN KINH DOANH

Dashboard 1 trả lời câu hỏi: *"Tình hình kinh doanh tổng thể và trạng thái xử lý đơn hàng T3/2026 như thế nào?"*

### 2.1 Kết quả KPI tổng thể

Bốn chỉ số trọng tâm được hiển thị nổi bật ngay đầu trang: tổng doanh thu **109 tỷ đồng**, số đơn hàng **3,000+**, tổng số lượng bán **72,000 sản phẩm**, số đại lý hoạt động **807 đại lý**.

### 2.2 Tích hợp dữ liệu Hạng mục A

Dashboard 1 hiển thị trực tiếp kết quả pipeline xử lý T3/2026, chứng minh sự kết nối giữa Hạng mục A và B:

| Giai đoạn pipeline | Kết quả |
|---|---|
| Emails nhận được | 1,132 |
| Parse thành công | 1,132 (100%) |
| Ghi vào database | 1,132 (100%) |
| Order lines hợp lệ | 1,132 (100%) |

Pipeline Hạng mục A đạt tỷ lệ thành công tuyệt đối — toàn bộ 1,132 đơn T3/2026 được xử lý không lỗi, sẵn sàng cho phân tích.

---

## 3. DASHBOARD 2 – PHÂN TÍCH THỜI GIAN

Dashboard 2 trả lời các câu hỏi: *"Xu hướng doanh số thay đổi như thế nào? Tháng nào là mùa cao điểm? Q1/2026 tăng trưởng bao nhiêu so cùng kỳ?"*

### 3.1 Xu hướng doanh thu theo tháng

| Tháng | Doanh thu | Tăng trưởng MoM |
|---|---|---|
| 2025-T1 | 3.2 tỷ | – |
| 2025-T2 | 6.3 tỷ | +97% |
| 2025-T3 | 18.6 tỷ | **+195%** |
| 2026-T1 | 21.1 tỷ | – |
| 2026-T2 | 19.4 tỷ | -8% |
| 2026-T3 | 40.8 tỷ | **+110%** |

Tháng 3 luôn là đỉnh tuyệt đối cả 2 năm — đây là pattern mùa vụ định kỳ.

### 3.2 So sánh cùng kỳ Q1/2025 vs Q1/2026

| Nhóm sản phẩm | Q1/2025 | Q1/2026 | YoY |
|---|---|---|---|
| Xe phổ thông | 17 tỷ | 42 tỷ | **+148%** |
| Xe trẻ em nhóm 1 | 4 tỷ | 8.2 tỷ | +104% |
| Xe trẻ em nhóm 2 | 2 tỷ | 4 tỷ | +93% |
| Xe thể thao thép | 3 tỷ | 2 tỷ | **-38%** ⚠️ |
| Xe thể thao nhôm | 2 tỷ | 1 tỷ | **-65%** ⚠️ |

Toàn thị trường tăng mạnh trong Q1/2026, nhưng 2 dòng xe thể thao là ngoại lệ đáng lo ngại — đây là tín hiệu cần điều tra ngay.

### 3.3 Phân tích mùa cao điểm

Heatmap Matrix (fiscal_month × week_of_year) cho thấy tuần 10–13 (nửa sau tháng 3) là cao điểm tuyệt đối — trùng với thời điểm chuẩn bị hè và mua xe cho học sinh. Doanh nghiệp cần đảm bảo tồn kho đầy đủ trước tuần 8 hàng năm.

### 3.4 Tốc độ tăng trưởng theo nhóm sản phẩm

Multi-line Chart 5 đường màu cho thấy xe phổ thông luôn dẫn đầu về tốc độ tăng trưởng MoM, trong khi xe thể thao có xu hướng giảm từ T2/2025.

---

## 4. DASHBOARD 3 – PHÂN TÍCH SẢN PHẨM

Dashboard 3 trả lời: *"Nhóm sản phẩm nào đang dẫn đầu? Dòng xe nào tăng trưởng nhanh? Màu sắc nào chiếm ưu thế? Sản phẩm nào là Stars/Cash Cows/Dogs?"*

### 4.1 Cấp 1 – Phân tích 5 nhóm sản phẩm

| Nhóm | Doanh thu | Số lượng | Giá bán TB | Tỷ trọng |
|---|---|---|---|---|
| Xe phổ thông | 59.0 tỷ | 39,682 | 1,487,727 đ | **70.15%** |
| Xe trẻ em nhóm 1 | 12.2 tỷ | 9,805 | 1,249,980 đ | 11.17% |
| Xe trẻ em nhóm 2 | 5.6 tỷ | 6,204 | 912,189 đ | 5.09% |
| Xe thể thao thép | 4.2 tỷ | 2,277 | 1,820,597 đ | 3.87% |
| Xe thể thao nhôm | 3.1 tỷ | 1,144 | 2,640,740 đ | 2.82% |

Xe phổ thông chiếm 70.15% doanh thu — tỷ trọng cao bất thường cho thấy danh mục sản phẩm đang phụ thuộc quá lớn vào một nhóm.

### 4.2 Cấp 2 – Top dòng xe

**Xe New 26** (~20 tỷ) và **Xe New 24** (~12 tỷ) dẫn đầu tuyệt đối về doanh thu. Top 5 dòng xe tăng trưởng nhanh nhất (theo YoY Q1) được xác định bằng `Line_YoY_Growth`, với filter loại bỏ outlier là các dòng xe hoàn toàn mới chưa có data 2025.

### 4.3 Cấp 3 – Màu sắc chiếm ưu thế

Heatmap Matrix (line_name × color) với conditional formatting trắng → xanh đậm cho thấy màu **Batman và Café/nâu** thống trị trên hầu hết các dòng xe. Nhiều màu đuôi (BLACKPINK, bóng, Cam) chỉ xuất hiện ở 1–2 dòng xe với doanh số rất thấp — cơ hội tối ưu hóa tồn kho đáng kể.

### 4.4 BCG Matrix

BCG Matrix (Scatter Chart: X = thị phần, Y = tốc độ tăng trưởng YoY, Size = doanh thu) phân loại 77 dòng xe thành 4 nhóm chiến lược:

| Nhóm BCG | Dòng xe đại diện | Hành động gợi ý |
|---|---|---|
| **Stars** | Xe New 26, Xe New 24 | Đầu tư mạnh, tăng sản xuất |
| **Cash Cows** | Xe phổ thông cũ | Duy trì, tối ưu chi phí |
| **Question Marks** | Xe trẻ em mới | Theo dõi, cân nhắc đầu tư |
| **Dogs** | Xe thể thao nhôm/thép | Xem xét cắt giảm SKU |

---

## 5. DASHBOARD 4 – PHÂN TÍCH ĐẠI LÝ

Dashboard 4 trả lời: *"Đại lý nào đang hoạt động tốt? 20% đại lý lớn chiếm bao nhiêu % doanh thu? Đại lý nào có nguy cơ rời bỏ? Cohort nào trung thành nhất?"*

### 5.1 Phân khúc RFM

Nhóm xây dựng phân khúc RFM hoàn toàn bằng DAX, không phụ thuộc Python hay thư viện ngoài. Bảng `RFM_Table` tính toán 3 chỉ số cho từng đại lý với ngày tham chiếu **28/03/2026**:

- **Recency:** Số ngày kể từ lần mua cuối cùng
- **Frequency:** Số đơn hàng riêng biệt
- **Monetary:** Tổng giá trị mua hàng

Mỗi chỉ số được chấm điểm 1–5 bằng Calculated Columns trong `RFM_Table`, sau đó phân loại thành 5 segment dựa trên `RFM_Total`:

| Segment | Số đại lý | Tỷ lệ | Ý nghĩa |
|---|---|---|---|
| Potential | 283 | **35%** | Mới mua, tiềm năng cao |
| At Risk | 280 | **35%** | Lâu không mua — cần chăm sóc ngay |
| Loyal | 184 | 23% | Mua đều đặn, ổn định |
| Champions | 41 | 5% | Mua nhiều, gần đây, giá trị cao nhất |
| Lost | 19 | 2% | Ngừng mua — khó lấy lại |

Kết quả đáng lo ngại: **70% đại lý** (Potential + At Risk) cần được chăm sóc tích cực trong Q2/2026.

### 5.2 Phân tích Pareto – Concentration Risk

Pareto Chart (Top 50 đại lý theo doanh thu + đường tích lũy %) cho thấy top 20% đại lý (~159/798) chiếm khoảng **68% tổng doanh thu**. Đây là mức độ tập trung cao — mất 1 đại lý lớn có thể giảm ngay 3–5% doanh thu tức thì.

### 5.3 Cohort Retention Analysis

Nhóm áp dụng Cohort Analysis bằng DAX thuần để đo lường tỷ lệ đại lý quay lại mua hàng theo từng tháng kể từ lần mua đầu tiên. Kết quả thực tế:

| Cohort | T+0 | T+1 | T+2 |
|---|---|---|---|
| 2025-T1 | 100% | 32.6% | 45.7% |
| 2025-T2 | 100% | 47.6% | – |
| 2025-T3 | 100% | – | – |
| 2026-T1 | 100% | 48.2% | **74.5%** |
| 2026-T2 | 100% | 58.8% | – |
| 2026-T3 | 100% | – | – |

Cohort 2026 có retention cao hơn đáng kể so với cùng kỳ 2025 — cho thấy chất lượng đại lý mới được tuyển trong 2026 được cải thiện rõ rệt.

### 5.4 Đại lý giảm hoạt động

Table lọc các đại lý có tần suất đặt hàng giảm >20% trong Q1/2026 so Q1/2025, hiển thị trạng thái Nguy hiểm / Cần theo dõi / Ổn định. Danh sách này được dùng trực tiếp làm danh sách ưu tiên gọi điện chăm sóc khách hàng.

---

## 6. DASHBOARD 5 & 6 – ĐỊA LÝ VÀ VẬN HÀNH

### Dashboard 5 – Phân tích Địa lý

Dashboard 5 trả lời: *"Thị trường nào lớn nhất? Tỉnh nào đang tăng trưởng? Tỉnh nào sụt giảm và tại sao?"*

**Phân bố theo vùng miền:**

| Vùng | Doanh thu | Tỷ trọng |
|---|---|---|
| Miền Bắc | ~81.7 tỷ | **74.7%** |
| Miền Trung | ~21.4 tỷ | 19.5% |
| Miền Nam | ~6.0 tỷ | **5.5%** ⚠️ |

**Top 5 tỉnh doanh thu cao nhất:**

| Tỉnh | Doanh thu | Tỷ trọng |
|---|---|---|
| Hà Nội | 41.1 tỷ | 37.6% |
| Thanh Hóa | 9.4 tỷ | 8.6% |
| Hải Phòng | 7.4 tỷ | 6.8% |
| Bắc Ninh | 7.2 tỷ | 6.6% |
| Ninh Bình | 7.0 tỷ | 6.4% |

**Tỉnh tăng trưởng nhanh nhất (YoY):** Quảng Ngãi (~11x), Hà Tĩnh (~4x), Quảng Ninh (~3.5x). Quảng Ngãi tăng đột biến bất thường — cần điều tra nguyên nhân để nhân rộng mô hình.

**Tỉnh đáng lo ngại:** TP.HCM — thành phố lớn nhất Việt Nam — chỉ đứng thứ 7 với ~5.8 tỷ. Đây là dấu hiệu rõ ràng về việc thiếu mạng lưới phân phối tại miền Nam.

### Dashboard 6 – Trạng thái Vận hành

Dashboard 6 trả lời: *"Pipeline xử lý T3/2026 hoạt động như thế nào? Bao nhiêu đơn thành công, lỗi, chờ xử lý?"*

| KPI Vận hành | Giá trị |
|---|---|
| Emails nhận | 1,132 |
| Xử lý thành công | 1,132 |
| Tỷ lệ thành công | **100%** |
| Đơn lỗi / chờ | **0** |

Area Chart timeline xử lý theo ngày cho thấy ngày 27–31/3 có đột biến ~150 đơn/ngày — cuối tháng là cao điểm đặt hàng, cần chuẩn bị năng lực xử lý trước cho các tháng tiếp theo.

---

## 7. INSIGHTS KINH DOANH

Nhóm đưa ra 6 insight theo đúng chuẩn đề bài (Phát hiện → Ý nghĩa → Khuyến nghị). Chi tiết xem tại [`insights.md`](./insights.md).

**Insight 1 – Mùa vụ tháng 3 là pattern định kỳ, có thể dự báo được**
Doanh thu T3/2026 đạt 40.8 tỷ, tăng 110% so T2/2026 và cao hơn T3/2025 tới 119%. Pattern lặp lại hoàn toàn ở cả 2 năm — tháng 3 luôn là đỉnh tuyệt đối, tập trung vào tuần 10–13. Doanh nghiệp nên tăng sản xuất từ tháng 1, đảm bảo tồn kho đầy đủ trước tuần 8, và đặt SLA giao hàng nhanh hơn trong tháng 3.

**Insight 2 – Rủi ro tập trung doanh thu ở mức báo động**
Top 20% đại lý (~159/798) chiếm ~68% tổng doanh thu. 280 đại lý "At Risk" theo RFM có nguy cơ ngừng mua hàng trong Q2/2026 — tác động tiềm tàng lên đến 15–20% doanh thu. Cần triển khai ngay chương trình loyalty và gọi điện chăm sóc trực tiếp 280 đại lý At Risk.

**Insight 3 – Thị trường miền Nam bị bỏ ngỏ hoàn toàn**
Miền Nam chỉ đóng góp 5.5% tổng doanh thu (~6 tỷ), trong khi Hà Nội chiếm 37.6% (41.1 tỷ). TP.HCM chỉ đứng thứ 7. Mở 5 đại lý mới tại TP.HCM trong Q2/2026, mục tiêu thị phần miền Nam lên 10% vào cuối 2026.

**Insight 4 – Xe thể thao đang mất thị phần trong khi toàn thị trường tăng mạnh**
Xe thể thao nhôm -65%, xe thể thao thép -38% trong Q1/2026 — 2 nhóm duy nhất có YoY âm khi tổng thị trường tăng +120%. BCG Matrix xếp cả 2 vào nhóm Dogs. Cần điều tra nguyên nhân để quyết định can thiệp hoặc cắt giảm SKU trong Q2/2026.

**Insight 5 – Chất lượng đại lý mới năm 2026 cải thiện đáng kể**
Cohort 2026-T1 đạt retention T+2 là 74.5%, cao hơn gần gấp đôi so với Cohort 2025-T1 (45.7%). Cohort 2026-T2 đạt retention T+1 là 58.8%, vượt trội so với 47.6% của 2025-T2. Cần phân tích và nhân rộng cách tiếp cận đại lý giai đoạn này cho Q2–Q3/2026.

**Insight 6 – Quảng Ngãi tăng trưởng đột biến 11x — cơ hội mở rộng miền Trung**
Quảng Ngãi có tốc độ tăng trưởng YoY cao nhất toàn hệ thống (~11x), bỏ xa tỉnh thứ 2 (Hà Tĩnh ~4x). Cần điều tra ngay để nhân rộng sang các tỉnh miền Trung có điều kiện tương tự như Bình Định, Quảng Nam.
