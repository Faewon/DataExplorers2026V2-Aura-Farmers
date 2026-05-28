# Data Explorers 2026 - Hạng mục C: Dự báo nhu cầu & Chiến lược 🚀

Notebook này là một phần trong quy trình xử lý dữ liệu của nhóm **Aura Farmers**.
File `Part_3.ipynb` tập trung giải quyết **Hạng mục C** với mục tiêu xử lý dữ liệu, trích xuất đặc trưng (*Feature Engineering*) và xây dựng mô hình học máy nhằm dự báo nhu cầu bán hàng cũng như phân tích chiến lược cho hệ thống đại lý.

---

# Mục tiêu chính 🎯

## 1. Dự báo doanh số

Dự đoán:

* **Doanh thu (Revenue)**
* **Sản lượng bán ra (Quantity)**

cho quý **Q2/2026** (tháng 4, 5, 6) ở nhiều cấp độ:

* Tổng quan toàn hệ thống
* Nhóm sản phẩm (*Product Group*)
* Dòng sản phẩm (*Product Line*)
* Mẫu xe (*SKU / Product*)

---

## 2. Phân tích xu hướng sản phẩm

Nhận diện:

* Màu sắc được ưa chuộng
* Các cải tiến hoặc phiên bản có tiềm năng tăng trưởng
* Xu hướng tiêu dùng trong thời gian tới

---

## 3. Phân tích hoạt động đại lý (RFM)

Đánh giá:

* Khả năng đặt hàng của đại lý
* Mức độ rủi ro rời bỏ (*Churn Risk*)
* Độ ưu tiên trong chiến lược tiếp thị (*Marketing Priority*)

---

# Cấu trúc dự án 📂

```text
Data_Explorers_2026/
│
├── datasets/                 # Chứa dữ liệu customer, email, sales, v.v.
│
├── Part_3.ipynb              # Data Preprocessing, Feature Engineering & Forecasting
├── requirements.txt          # Danh sách thư viện Python cần thiết
└── README.md                 # Tài liệu hướng dẫn dự án
```

---

# Luồng xử lý dữ liệu trong `Part_3.ipynb` ⚙️

Notebook thực hiện quy trình End-to-End với các bước chính sau:

## 1. Load & Khám phá dữ liệu

Tích hợp nhiều bảng dữ liệu:

* `fact_sales`
* `product`
* `product_line`
* `product_group`
* `customer`
* `province`

Các bước kiểm tra bao gồm:

* Phân tích missing values
* Kiểm tra độ đa dạng của danh mục sản phẩm
* Phân tích Category / Line / SKU

---

## 2. Data Aggregation (Tổng hợp dữ liệu)

Tổng hợp dữ liệu bán hàng theo:

* Ngày
* Tuần
* Tháng

Đồng thời phân tích đa chiều theo:

* Category
* Product Line
* SKU

---

## 3. Feature Engineering (Trích xuất đặc trưng)

### Time-based Features

Tạo các đặc trưng liên quan đến thời gian:

* Month
* Quarter
* Week
* `is_holiday`

### Lag & Rolling Features

Xây dựng:

* Lag 1, 2, 3
* Rolling Mean 3 tháng
* Rolling Standard Deviation 3 tháng

### Growth Features

Tính toán:

* MoM (*Month-over-Month Growth*)
* YoY (*Year-over-Year Growth*)

### Missing Values & Encoding

* Xử lý dữ liệu thiếu bằng:

  * Forward Fill (*FFill*)
  * Backward Fill (*BFill*)
  * Mean Imputation
* Mã hóa biến phân loại (*Categorical Encoding*)

---

# Mô hình hóa (Modeling) 🤖

Áp dụng các mô hình Machine Learning nhằm dự báo:

* Revenue
* Quantity

cho các tháng:

* 04/2026
* 05/2026
* 06/2026

Các thuật toán sử dụng:

* Gradient Boosting Regressor
* Random Forest
* Logistic Regression
* Và các mô hình hỗ trợ khác từ `scikit-learn`

---

# Phân tích Đại lý (Dealer Analysis - RFM) 🏪

Thực hiện phân tích RFM:

* **Recency** – Thời gian từ lần mua gần nhất
* **Frequency** – Tần suất đặt hàng
* **Monetary** – Tổng giá trị giao dịch

Từ đó:

* Phân loại mức độ rủi ro:

  * High Risk
  * Medium Risk
  * Low Risk
* Xác định:

  * Marketing Priority
  * Chiến lược chăm sóc phù hợp

---

# Công nghệ & Thư viện sử dụng 🛠️

## Ngôn ngữ

* Python 3

## Xử lý dữ liệu

* pandas
* numpy

## Trực quan hóa dữ liệu

* matplotlib
* seaborn

## Machine Learning

* scikit-learn

  * Gradient Boosting Regressor
  * Random Forest
  * Logistic Regression
  * Các mô hình hỗ trợ khác

---

# Hướng dẫn cài đặt & Khởi chạy 💻

## 1. Clone Repository

```bash
git clone https://github.com/AIVIETNAM-AIO-MQQ/data_explorers.git
cd Data_Explorers_2026
```

---
## 2. Kích hoạt Virtual Environment

### Tạo Virtual Environment

```bash
python -m venv venv
```
---
### Kích hoạt trên macOS / Linux

```bash
source venv/bin/activate
```
---
### Kích hoạt trên Windows (CMD)

```bash
venv\Scripts\activate
```
---
### Kích hoạt trên Windows (PowerShell)

```powershell
venv\Scripts\Activate.ps1
```

---

## 3. Cài đặt thư viện phụ thuộc

```bash
pip install -r requirements.txt
```


---

## 4. Thiết lập dữ liệu

Đảm bảo thư mục `datasets/` đã chứa đầy đủ dữ liệu cần thiết trước khi chạy notebook.

---

## 5. Chạy Jupyter Notebook

```bash
jupyter notebook Part_3.ipynb
```

---

# Kết quả kỳ vọng 📈

Sau khi hoàn thành notebook:

* Dự báo được xu hướng doanh số Q2/2026
* Nhận diện sản phẩm tiềm năng
* Đánh giá hiệu quả hoạt động đại lý
* Hỗ trợ xây dựng chiến lược marketing và phân phối phù hợp

---

# Nhóm thực hiện 👥

**Aura Farmers**
Data Explorers 2026
