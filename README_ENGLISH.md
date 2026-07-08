# Thong Nhat Bicycles – Data Explorers Round 2
**Team: Aura Farmers**

---

## Project Overview

This project builds an end-to-end data system for **Thong Nhat Bike** – a bicycle manufacturer and distributor with over 200 SKUs, 5 product groups, and a nationwide network of **798 dealers**.

Before this project, the company had no management dashboard, no automated forecasting system, and every order had to be entered into the ERP manually. The project addresses 3 major problems:

| Category | Description |
|---|---|
| **A – Operations** | Automated ETL pipeline processing orders from email/PDF |
| **B – Analytics** | 6 Power BI Dashboards + 6 business insights |
| **C – Forecasting** | Q2/2026 demand forecast + strategic action matrix |

### Overall Data (Jan/2025 – Mar/2026)

| Metric | Value |
|---|---|
| Total revenue | 109 billion VND |
| Number of orders | 3,000+ |
| Products sold | 72,000 |
| Active dealers | 807 |
| Emails processed (Mar/2026) | 1,132 / 1,132 (100%) |

---

## System Architecture

The system follows a 3-layer **Medallion architecture**:

```
Bronze  →  Silver  →  Gold
(raw)      (clean)    (analytics-ready)
```

- **Bronze**: Raw data from PostgreSQL, stored as-is
- **Silver**: Standardized geography, cleaned customer names (797/798 customers mapped)
- **Gold**: `gold_fact_sales` – 25,754 rows, the single source for all 6 dashboards

### Tech Stack

| Component | Technology |
|---|---|
| Orchestration | Apache Airflow 2.8.1 |
| Database | PostgreSQL 13 |
| Language | Python 3.11 |
| Containerization | Docker Compose (5 services) |
| PDF parsing | pdfplumber |
| BI / Dashboard | Power BI |

---

## Directory Structure

```
.
├── A. Automated Order Processing/
│   ├── airflow/dags/
│   │   └── email_pipeline_dag.py        # Main email-processing DAG
│   ├── init-scripts/
│   │   ├── 01_create_tables.sql
│   │   ├── 02_import_data.sql
│   │   ├── 03_create_email_log.sql
│   │   ├── 04_create_silver_tables.sql
│   │   └── 05_create_gold_tables.sql
│   ├── src/
│   │   ├── extract.py                   # Read MIME, extract PDF
│   │   ├── validators.py                # Validate orders
│   │   ├── loaders.py                   # Write to PostgreSQL
│   │   ├── router.py                    # Route files to processed/failed
│   │   ├── silver.py                    # Silver layer standardization
│   │   └── warehouse.py                 # Refresh fact tables
│   ├── .gitignore
│   ├── converter.py
│   ├── docker-compose.yml
│   ├── dockerfile.airflow
│   ├── requirements.txt
│   └── tnbike_database_schema.md        # Full database schema
│
├── B. Dashboards and Insights/
│   ├── dashboard/
│   │   ├── Business Overview.jpg
│   │   ├── Time Analysis.jpg
│   │   ├── Product Analysis.jpg
│   │   ├── Dealer Analysis.jpg
│   │   ├── Geographic Analysis.jpg
│   │   ├── Operations Status.jpg
│   │   └── PowerBIDashboard.pbix        # Original Power BI file
│   └── docs/
│       ├── BaoCaoKyThuat_HangMucB.md
│       ├── Insights.md
│       ├── DataExplorers2026 – Round 2 Exam.pdf
│       └── README.md
│
├── C. Demand Forecasting and Strategy/
│   ├── Part_3.ipynb                     # Analysis & forecasting notebook
│   ├── Readme.md
│   └── requirements.txt
│
├── Data/                                # Source data
│   ├── README.md                        # Data description
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
├── DataExplorers2026 – Round 2 Exam.pdf
├── requirements.txt
└── README.md                            ← (this file)
```

---

## Category A – Automated Order Processing

### Results

| Metric | Value |
|---|---|
| Total emails received | 1,132 |
| Successfully processed | 1,132 (100%) |
| Errors | 0 |
| Validation pass rate | 100% |

### DAG: `email_order_pipeline`

Processes up to **8 mapped tasks** in parallel:

```
list_eml_files
    → process_single_email (×N, parallel)
        → summarize_results
        → run_silver_layer
        → refresh_fact_sales
        → refresh_gold_fact_sales
```

### Per-Email ETL Flow

```
.eml → read metadata → extract PDF (MIME)
     → parse PDF header → read item table (pdfplumber)
     → validate (qty × price = total, matches email body total)
     → cross-check product master → upsert sales_order
     → delete & rewrite order_line (idempotent)
     → write email_log → route → processed/ or failed/
```

### Main Modules

| Module | Function |
|---|---|
| `extract.py` | Decode MIME headers (RFC 2047), extract PDF, read tables via pdfplumber |
| `validators.py` | Check required fields, line-item formulas, order totals, product master |
| `loaders.py` | Upsert `sales_order`, replace `order_line`, upsert `email_log` |
| `router.py` | Route files to `processed/` or `failed/`, clean up temp PDFs |
| `silver.py` | Standardize customer names (remove redundant labels) |

### Setup & Run

```bash
# Start the full stack
docker compose up -d

# Trigger the DAG (manual trigger)
airflow dags trigger email_order_pipeline

# Check results
psql -U postgres -d tnbike -c "SELECT COUNT(*) FROM email_log WHERE status='success';"
```

**Requirements:** Docker, Docker Compose. Place `.eml` files into `maildata/incoming/`.

---

## Category B – Dashboards & Insights

### 6 Power BI Dashboards (source: `gold_fact_sales`)

| # | Dashboard | Main Content |
|---|---|---|
| 1 | Business Overview | 4 KPIs: 109B / 3K+ orders / 72K products / 807 dealers |
| 2 | Time Analysis | MoM, YoY trends by product group |
| 3 | Product Analysis | Revenue structure, BCG Matrix, colors |
| 4 | Dealer Analysis | RFM segmentation, Pareto, cohort retention |
| 5 | Geographic Analysis | Province/region heatmap, top provinces, growth |
| 6 | Operations Status | Pipeline status, email log, daily processing |

> **Technical note:** Data has gaps (Q1/2025 and Q1/2026). Use `YearMonth_Label` (text) + `YearMonth_Sort` (int) instead of `PREVIOUSMONTH()` to avoid timeline axis errors.

### 6 Key Business Insights

1. **March seasonality is predictable** – Mar/2026 reached 40.8B (+110% MoM), a pattern repeated for 2 years → inventory should be prepared starting January.
2. **Revenue concentration risk** – Top 20% (~159 dealers) account for 68% of revenue; 280 at-risk dealers could impact 15–20% of revenue.
3. **The South is underserved** – Only 5.5% (~6B), with HCMC ranking 7th. A major untapped opportunity.
4. **Sport bikes losing market share** – Aluminum -65%, Steel -38% YoY while the overall market grew +120%.
5. **New 2026 dealer quality improved** – Cohort 2026-Q1 T+2 retention reached 74.5% vs. 45.7% for 2025-Q1.
6. **Quang Ngai grew ~11x** – The highest growth in the system; should be replicated in Binh Dinh and Quang Nam.

---

## Category C – Forecasting & Strategy

### Q2/2026 Forecast Results

**Total forecasted revenue: ~65–75 billion VND**

| Product Group | Q1/2026 (billion) | Q2 Forecast (billion) | Share |
|---|---|---|---|
| Standard bikes | 43.06 | ~42–49 | ~54% |
| Kids' bikes group 1 | 9.21 | ~8–10 | ~11% |
| Steel sport bikes | 2.97 | ~3–4 | ~4% |
| Aluminum sport bikes | 1.94 | ~1.5–2.5 | ~2% |
| Kids' bikes group 2 | 2.20 | ~2–3 | ~3% |

### 6 Priority Strategic Actions Matrix

| # | Action | Target KPI |
|---|---|---|
| 1 | Prepare Q3 inventory starting in June | No stock-outs for top 5 lines in Mar/2027 |
| 2 | Re-activate 100 at-risk dealers in early Q2 | Active dealers ≥ 450 by end of Q2/2026 |
| 3 | Expand in Central Vietnam: Nghe An, Ha Tinh, Quang Ninh | +20 Central Vietnam dealers in Q2 |
| 4 | Push Star products: GN 06-24 2.0 Bike, Women's Bike, Bunny 16 Bike | Stars reach 35% of total revenue |
| 5 | Optimize color catalog – narrow down to 15 core colors | Reduce inventory costs by 15% |
| 6 | EOL BCG "Dogs" lines that declined >80% | Free up capacity for Stars |

### Running the Forecast Notebook

```bash
cd "C. Demand Forecasting and Strategy"
pip install -r requirements.txt
jupyter notebook Part_3.ipynb
```

---

## Database Schema

Full schema available at: `A. Automated Order Processing/tnbike_database_schema.md`

**Main Tables:**

| Table | Description |
|---|---|
| `sales_order` | 1,132 orders from Mar/2026 |
| `order_line` | Line-item details |
| `email_log` | Log of 1,132 processed emails (100% success) |
| `silver_customer_geo` | 797/798 customers with geographic mapping |
| `silver_province` | 34 standardized provinces/cities |
| `gold_fact_sales` | 25,754 rows – the single analytics source |

---

## Summary of Achievements

| Category | Result |
|---|---|
| A – Operations | 1,132/1,132 orders processed successfully, 0 errors, stable Airflow parallelism (8 tasks) |
| B – Analytics | 6 Power BI dashboards, 6 actionable business insights |
| C – Forecasting | Q2/2026 forecast ~65–75B, 6-action strategic matrix |

**Q2/2026 Target KPIs:** Revenue 120B · Volume 80K units · Standard bike share ≤68% · At-risk dealers ≤250.

---

*Team Aura Farmers – Data Explorers Round 2, 2026*
