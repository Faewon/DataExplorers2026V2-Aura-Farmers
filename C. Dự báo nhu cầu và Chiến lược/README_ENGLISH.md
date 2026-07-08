# Data Explorers 2026 - Category C: Demand Forecasting & Strategy

This notebook is part of the data pipeline built by team **Aura Farmers**.
`Part_3.ipynb` focuses on **Category C**, aiming to process data, perform feature engineering, and build machine learning models to forecast sales demand and analyze strategy for the dealer network.

---

# Main Objectives

## 1. Sales Forecasting

Predict:

* **Revenue**
* **Quantity sold**

for **Q2/2026** (April, May, June) at multiple levels:

* System-wide overview
* Product Group
* Product Line
* SKU / Product

---

## 2. Product Trend Analysis

Identify:

* Preferred colors
* Improvements or versions with growth potential
* Upcoming consumption trends

---

## 3. Dealer Activity Analysis (RFM)

Assess:

* Dealer ordering capacity
* Churn Risk level
* Marketing Priority

---

# Project Structure

```text
│
├── Part_3.ipynb              # Data Preprocessing, Feature Engineering & Forecasting
├── requirements.txt          # List of required Python libraries
└── README.md                 # Project documentation
```

---

# Data Processing Flow in `Part_3.ipynb`

The notebook follows an end-to-end process with the following main steps:

## 1. Load & Explore Data

Integrates multiple data tables:

* `fact_sales`
* `product`
* `product_line`
* `product_group`
* `customer`
* `province`

Checks performed include:

* Missing values analysis
* Product catalog diversity check
* Category / Line / SKU analysis

---

## 2. Data Aggregation

Aggregates sales data by:

* Day
* Week
* Month

Along with multi-dimensional analysis by:

* Category
* Product Line
* SKU

---

## 3. Feature Engineering

### Time-based Features

Creates time-related features:

* Month
* Quarter
* Week
* `is_holiday`

### Lag & Rolling Features

Builds:

* Lag 1, 2, 3
* 3-month Rolling Mean
* 3-month Rolling Standard Deviation

### Growth Features

Calculates:

* MoM (Month-over-Month Growth)
* YoY (Year-over-Year Growth)

### Missing Values & Encoding

* Handles missing data using:

  * Forward Fill (FFill)
  * Backward Fill (BFill)
  * Mean Imputation
* Categorical Encoding

---

# Modeling

Applies Machine Learning models to forecast:

* Revenue
* Quantity

for the months:

* 04/2026
* 05/2026
* 06/2026

Algorithms used:

* Gradient Boosting Regressor
* Random Forest
* Logistic Regression
* Other supporting models from `scikit-learn`

---

# Dealer Analysis (RFM)

Performs RFM analysis:

* **Recency** – Time since the last purchase
* **Frequency** – Order frequency
* **Monetary** – Total transaction value

From this:

* Classifies risk level:

  * High Risk
  * Medium Risk
  * Low Risk
* Determines:

  * Marketing Priority
  * Appropriate engagement strategy

---

# Technology & Libraries

## Language

* Python 3

## Data Processing

* pandas
* numpy

## Data Visualization

* matplotlib
* seaborn

## Machine Learning

* scikit-learn

  * Gradient Boosting Regressor
  * Random Forest
  * Logistic Regression
  * Other supporting models

---

# Setup & Run Instructions

## 1. Clone Repository

```bash
git clone https://github.com/AIVIETNAM-AIO-MQQ/data_explorers.git
cd Data_Explorers_2026
```

---
## 2. Activate Virtual Environment

### Create Virtual Environment

```bash
python -m venv venv
```
---
### Activate on macOS / Linux

```bash
source venv/bin/activate
```
---
### Activate on Windows (CMD)

```bash
venv\Scripts\activate
```
---
### Activate on Windows (PowerShell)

```powershell
venv\Scripts\Activate.ps1
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```


---

## 4. Set Up the Data

Make sure the `datasets/` folder contains all the required data before running the notebook.

---

## 5. Run Jupyter Notebook

```bash
jupyter notebook Part_3.ipynb
```

---

# Expected Results

Upon completion of the notebook:

* Forecasted Q2/2026 sales trends
* Identified potential products
* Assessed dealer performance
* Support for building appropriate marketing and distribution strategies

---

# Team

**Aura Farmers**
Data Explorers 2026
