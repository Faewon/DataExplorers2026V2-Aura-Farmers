# DataExplorers 2026 – Category B: Business Intelligence Dashboard

A multi-dimensional business analytics system built on **Power BI** for **Thong Nhat Bike** — a B2B bicycle manufacturer and distributor with 200+ SKUs and 798 dealers nationwide.

> Directly integrates March 2026 data from the Category A pipeline · 6 dashboards · RFM & Cohort Analysis built entirely in DAX

---

## Overview Metrics

| Metric | Value |
|---|---|
| Data period | Jan/2025 – Mar/2026 |
| Total revenue | 109 billion VND |
| Number of orders | 3,000+ |
| Products sold | 72,000 |
| Active dealers | 807 |
| Mar/2026 orders from Category A | 1,132 · 100% processed successfully |

---

## Repository Structure

```
├── dashboard/
│   ├── PowerBIDashboard.pbix              # Main Power BI file
│   ├── Business Overview.jpg              # Dashboard 1
│   ├── Time Analysis.jpg                  # Dashboard 2
│   ├── Product Analysis.jpg               # Dashboard 3
│   ├── Dealer Analysis.jpg                # Dashboard 4
│   ├── Geographic Analysis.jpg            # Dashboard 5
│   └── Operations Status.jpg              # Dashboard 6
│
├── docs/
│   ├── BaoCaoKyThuat_HangMucB.md          # Full technical report
│   └── Insights.md                        # 6 insights · Finding → Meaning → Recommendation
│
├── DataExplorers2026 – Round ... Exam     # Original exam brief
└── README.md
```

---

## Data Architecture – 3-Layer Medallion

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
  GOLD   ── gold_fact_sales  (25,754 rows · single source for all 6 dashboards)
             email_log       (dedicated source for Dashboard 6)
```

---

## 6 Dashboards

| # | Name | Business Question |
|---|---|---|
| 1 | Business Overview | What's the overall situation? What's the Mar/2026 pipeline status? |
| 2 | Time Analysis | What are the sales trends? Peak season? YoY Q1? |
| 3 | Product Analysis | Which group leads? Who are the BCG Stars/Dogs? Which color dominates? |
| 4 | Dealer Analysis | 5 RFM segments? 20/80 Pareto rule? Cohort retention? |
| 5 | Geographic Analysis | Which market is largest? Which provinces show unusual growth/decline? |
| 6 | Operations Status | Mar/2026 email pipeline: success / error / pending? |

---

## Key Insights

Full details in [`docs/Insights.md`](./docs/Insights.md).

| # | Insight | Key Data |
|---|---|---|
| 1 | March seasonality is predictable | Mar/2026 reached 40.8B · +110% MoM · peak in weeks 10–13 |
| 2 | Alarming revenue concentration risk | Top 20% of dealers account for 68% · 280 At Risk → -15~20% risk |
| 3 | The Southern market is underserved | South 5.5% · HCMC ranks 7th (~5.8B) |
| 4 | Sport bikes losing market share against the market trend | Aluminum -65% · Steel -38% · overall market +120% |
| 5 | New 2026 dealer quality has clearly improved | T+2 retention: 74.5% (2026) vs 45.7% (2025) |
| 6 | Quang Ngai grew ~11x – opportunity to replicate across Central Vietnam | Ha Tinh ~4x · Quang Ninh ~3.5x |

---

## Actual KPIs & Targets

| KPI | Q1/2026 | Q2/2026 Target |
|---|---|---|
| Revenue | 109B | 120B |
| Volume | 72K | 80K |
| Avg revenue/dealer | 137M | 150M |
| Standard bike share | 70.15% | ≤68% |
| At-Risk dealers | 280 (35%) | ≤250 |
| Email processing rate | 100% | ≥98% |

---

## Technology

| Technology | Role |
|---|---|
| Power BI | Visualization, DAX measures, Calculated Columns |
| PostgreSQL | Original data source |
| Medallion Architecture | Bronze → Silver → Gold organization |
| RFM Analysis | Built entirely in DAX – no Python used |
| Cohort Analysis | Built entirely in DAX – no Python used |
