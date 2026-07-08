# DataExplorer — Email-to-Warehouse ETL Pipeline

An automated order-processing pipeline for **Thong Nhat Bicycle Company**, built for the Data Explorers 2026 competition. The system receives order emails from dealers (`.eml` files with PDF attachments), extracts order data, validates it, and loads it into PostgreSQL according to the provided schema.

The project applies a **Medallion Architecture** to clearly separate data layers: the **Bronze/Source** layer stores transactional data in its original schema, the **Silver** layer cleans and enriches customer, product, and geographic data, and the **Gold** layer aggregates the data into fact tables used for analytics. This design preserves compatibility with the organizers' requirements while adding cleaner data layers for reporting and BI.

## Goal

Automatically process all March 2026 orders from email/PDF and write validated data into the database.

Main outputs:

- `email_log`: records the processing status of each email
- `sales_order`: order headers
- `order_line`: order line items
- `fact_sales`: flat fact table following the original schema
- `silver_province`, `silver_customer_geo`: cleaned and standardized geography layer
- `gold_fact_sales`: enhanced fact table using both legacy and Silver geography in parallel

## Tech Stack

- Apache Airflow 2.8.1
- PostgreSQL 13 / Debezium PostgreSQL image
- Python 3.11
- Docker Compose
- `pdfplumber`
- `psycopg2-binary`

## Pipeline Deployment

### Requirements

- Docker
- Docker Compose

### Startup

```bash
docker compose up -d
```

Main services:

| Service | Description |
|---|---|
| `postgres-airflow` | Airflow metadata database |
| `postgres_dataexp` | Business PostgreSQL database, `tnbike` schema |
| `airflow-webserver` | Airflow web interface |
| `airflow-scheduler` | Executes DAGs, schedules the pipeline |
| `airflow-init` | Runs Airflow DB migrations and creates the user |

### Default Credentials

| Service | User | Password |
|---|---|---|
| Airflow Web UI | `airflow` | `airflow` |
| Business PostgreSQL | `admin` | `admin` |
| Airflow PostgreSQL | `airflow` | `airflow` |

### Resetting the Entire Database

To start over and rerun all init scripts:

```bash
docker compose down -v
docker compose up --build -d
```

Note: `down -v` will delete Docker volumes, including the current PostgreSQL data.

## How to Run the Pipeline

1. On first run, extract all `.eml` files into:

```text
maildata/incoming/
```

   **If restarting the pipeline, run the following command in the terminal:**

```bash
 mv maildata/processed/*.eml maildata/incoming/ && mv maildata/failed/*.eml maildata/incoming 
```

2. Open the Airflow UI:

```text
http://localhost:8085
```

3. Trigger the DAG by clicking the Play button under Actions:

```text
email_order_pipeline
```

Or trigger it via CLI:

```bash
docker exec -it dataexplorer-airflow-scheduler-1 airflow dags trigger email_order_pipeline
```

After processing, emails will be moved to:

```text
maildata/processed/  # if the email was processed successfully
maildata/failed/     # if the email failed to process
```

Logs of the email data extraction process are stored in the table:

```text
tnbike.email_log
```

4. 


## System Architecture
![Pipeline Architecture](docs/images/pipeline-architecture.svg)

## DAG Flow

**DAG ID:** `email_order_pipeline`  
**Trigger:** manual  
**Parallelism:** up to 8 mapped tasks

| # | Task | Description |
|---|---|---|
| 1 | `list_eml_files` | Scans the `maildata/incoming/` folder for the list of emails |
| 2 | `process_single_email` | Mapped task: one task per email. Parses email/PDF, validates, loads to DB, logs, routes file |
| 3 | `summarize_results` | Aggregates success/failure counts and classifies errors |
| 4 | `run_silver_layer` | Runs the data cleaning/enrichment steps |
| 5 | `refresh_fact_sales` | Refreshes the `fact_sales` table per the legacy schema |
| 6 | `refresh_gold_fact_sales` | Refreshes the `gold_fact_sales` table using Silver geography |

## Per-Email ETL

Each email is processed as an independent unit within the mapped task `process_single_email`.

```text
.eml
→ read email metadata
→ extract the attached PDF file
→ read the PDF header and line-item table
→ validate quantity, unit price, and line total
→ ensure the product code exists in the product master
→ resolve or auto-create the customer by tax ID (MST)
→ upsert into sales_order
→ replace the order's order_line to support idempotent reruns
→ write processing status to email_log
→ move the email to processed/ or failed/
```

### `extract.py` - Data Extraction

- Decodes MIME headers of the email.
- Retrieves the text/plain content from the email body.
- Saves the attached PDF file to a temporary folder.
- Extracts anchor fields from the email:
  - `so_number`
  - `MST` (tax ID)
  - customer name
  - customer address
  - total amount declared in the email
- Reads the line-item table in the PDF using `pdfplumber`
- Uses regex fallback when the PDF table cannot be read in tabular structure.

### `validators.py` - Validation

- Checks required fields for each line item.
- Validates the formula: `quantity × unit_price ≈ line_total`
- Cross-checks the total extracted from the PDF against the total anchored in the email.
- Checks that `product_code` exists in `tnbike.product`
- Auto-creates missing products as an unresolved row when needed: `UNRESOLVED PRODUCT {product_code}`

### `loaders.py` - Writing Data to the Database

- Upserts `sales_order`
- Deletes and rewrites `order_line` for the same order to support idempotent reruns.
- Upserts `email_log`
- Resolves customers by tax ID (MST).
- Auto-creates a new customer when the MST is valid but not yet present in tnbike.customer.

### `router.py` - Post-Processing File Routing

- Moves successfully processed emails into: `processed/`.
- Moves failed emails into `failed/`.
- Cleans up temporary PDF files after processing.

## Error Handling

Business or data errors are handled directly within each mapped email-processing task. Instead of failing the entire DAG, the task returns a `status dict` and writes the processing status to the `email_log` table.

Common error categories:

| Error Category | Cause |
|---|---|
| `missing_customer` | Email body does not contain a valid tax ID (MST) to identify the customer |
| `missing_product` | Unable to extract `product_code` from the PDF |
| `line_total_mismatch` | Extracted line total does not match quantity × unit price or the order total |
| `unknown` | Other business/data errors outside the above categories |

Infrastructure or critical code errors — for example, PostgreSQL connection failures, module import errors, or SQL warehouse errors — are still left to fail normally in Airflow, for easier detection and troubleshooting.

---

## Silver Layer

The Silver layer is used to clean, standardize, and enrich data after it has been loaded into the base tables.

### Customer Name Standardization

Some emails retain field labels in the customer name, such as `Name:` or `Dealer:`. The Silver layer standardizes these values before they are used for analytics.

Example:

```text
Name: PHUC AN CO., LTD
→ PHUC AN CO., LTD

Dealer: BICYCLE SHOP A
→ BICYCLE SHOP A
```

## Project Structure

```text
DataExplorer/
├── airflow/
│   └── dags/
│       └── email_pipeline_dag.py
├── docs/
│   └── images/
│       └── pipeline-architecture.svg
├── init-scripts/
│   ├── 01_create_tables.sql
│   ├── 02_import_data.sql
│   ├── 03_create_email_log.sql
│   ├── 04_create_silver_tables.sql
│   └── 05_create_gold_tables.sql
├── maildata/
│   ├── incoming/
│   ├── processed/
│   └── failed/
├── src/
│   ├── __init__.py
│   ├── extract.py
│   ├── validators.py
│   ├── loaders.py
│   ├── router.py
│   ├── silver.py
│   └── warehouse.py
├── docker-compose.yml
├── dockerfile.airflow
├── requirements.txt
├── tnbike_database_schema.md
└── README.md
```

## Manual Checks
After running the DAG, the queries below can be used to quickly check pipeline status and post-processing data quality.

### Email Processing Status

```sql
SELECT processing_status, COUNT(*)
FROM tnbike.email_log
GROUP BY processing_status
ORDER BY processing_status;
```

### Number of Orders in March 2026

```sql
SELECT COUNT(*) AS march_order_count
FROM tnbike.sales_order
WHERE order_date >= DATE '2026-03-01'
  AND order_date < DATE '2026-04-01';
```

### Number of Order Lines in March 2026

```sql
SELECT COUNT(*) AS march_order_line_count
FROM tnbike.order_line ol
JOIN tnbike.sales_order so
    ON ol.order_id = so.order_id
WHERE so.order_date >= DATE '2026-03-01'
  AND so.order_date < DATE '2026-04-01';
```

### Number of Fact Rows in March 2026

```sql
SELECT COUNT(*) AS march_fact_rows
FROM tnbike.fact_sales
WHERE order_date >= DATE '2026-03-01'
  AND order_date < DATE '2026-04-01';
```

### Geographic Coverage of the Silver Layer

```sql
SELECT
    COUNT(*) AS total_customers,
    COUNT(scg.customer_code) AS mapped_customers,
    COUNT(*) - COUNT(scg.customer_code) AS unmatched_customers
FROM tnbike.customer c
LEFT JOIN tnbike.silver_customer_geo scg
    ON c.customer_code = scg.customer_code;
```

### Product Hierarchy Coverage

```sql
SELECT
    COUNT(*) AS fact_rows_without_product_line,
    SUM(quantity) AS quantity,
    SUM(line_total) AS revenue
FROM tnbike.fact_sales
WHERE line_id_fk IS NULL;
```

### Remaining Products Missing a Product Line

```sql
SELECT
    product_code,
    product_name,
    COUNT(*) AS fact_rows,
    SUM(quantity) AS total_qty,
    SUM(line_total) AS total_revenue
FROM tnbike.fact_sales
WHERE line_id_fk IS NULL
GROUP BY product_code, product_name
ORDER BY total_revenue DESC
LIMIT 30;
```

## Manual Check of the Silver Layer

Run the following inside the Airflow scheduler container:

```bash
docker exec -it dataexplorer-airflow-scheduler-1 bash
cd /opt/airflow
PYTHONPATH=/opt/airflow/src python -c "from silver import run_silver_layer; run_silver_layer()"
```

Expected current results:

```text
unresolved_product_count = 0
silver_province_count = 34
mapped_customer_count = 797
unmatched_customer_count = 1
```

## Development Notes

### Recreating the Airflow User

If the Airflow account was not created automatically, it can be created manually with:

```bash
docker exec -it dataexplorer-airflow-scheduler-1 airflow users create \
  --username airflow \
  --password airflow \
  --firstname admin \
  --lastname admin \
  --role Admin \
  --email admin@example.com
```

Check the list of users:

```bash
docker exec -it dataexplorer-airflow-scheduler-1 airflow users list
```

### Moving Processed Emails Back for Reruns

If emails have already been moved to processed/ and the DAG needs to be rerun from scratch:

```bash
docker exec -it dataexplorer-airflow-scheduler-1 bash
mv /opt/airflow/maildata/processed/*.eml /opt/airflow/maildata/incoming/
```

## Git Notes

Do not commit raw data, runtime-generated files, or sensitive configuration files:

```text
.env
CSVData/
EDA/output/
maildata/incoming/*
maildata/processed/*
maildata/failed/*
pdfdata/
processing/
*.rar
*.pdf
*.log
```

Components that should be committed:

```text
src/
airflow/dags/
init-scripts/
docker-compose.yml
dockerfile.airflow
requirements.txt
README.md
tnbike_database_schema.md
```

## Current Status

Current status:

```text
Bronze ingestion: functional
Silver enrichment: functional and validated
fact_sales refresh: functional
gold_fact_sales: analytics-ready layer using Silver geography
```

Remaining data quality limitations:

```text
- 1 customer cannot be mapped geographically because both the address and legacy province are NULL.
- Some products still lack a product_line because the corresponding catalogue line does not exist in the provided product_line table.
- The original province table is retained as legacy/source geography but is not treated as the canonical geography source for clean analytics.
```
