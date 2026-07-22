# 📘 Customer Support Analytics – Medallion Architecture (AWS)

A scalable, ACID-compliant data pipeline designed to ingest, transform, and analyze customer support tickets and application logs using AWS cloud services. The project follows the Medallion Architecture (Raw → Processed → Curated) and enables reliable analytics through Redshift, Athena, and Power BI.

---

## Architecture Diagram

![Architecture Diagram](docs/Pipeline%20Architecture.png)

## 🚀 Architecture Overview

### 1️⃣ Raw Layer (Ingestion)
- Ticket ingestion loop reads daily ticket data from the database and uploads it to S3 (support-tickets/raw/).
- Log ingestion loop scans local log files and uploads them to S3 automatically (support-logs/raw/).
- Both scripts track ingestion dates using a date tracker file to avoid duplicates.

### 2️⃣ Processed Layer (Transformation)
- Lambda (Logs):
  - Triggered automatically when new log files arrive in S3.
  - Parses raw logs using regex, cleans invalid entries, fixes typos, removes duplicates, and converts to Parquet.
  - Saves processed data back to S3 in Parquet format (support-logs/processed/).
- Glue (Tickets):
  - Triggered by Lambda when new ticket files arrive.
  - Performs ETL transformations and writes clean Parquet files to S3 (support-tickets/processed/).

### 3️⃣ Curated Layer (Business-Ready Layer)

#### Lambda – Curated Logs Job
- Computes BI metrics:
  - total_logs
  - error_count
  - warning_count
  - avg_cpu_usage
  - avg_response_time
- Writes curated metrics to:
  - `s3://.../curated_layer/curated_logs/`

#### Glue – Curated Tickets Job
- Reads processed tickets
- Produces analysis-ready curated dataset
- Writes to:
  - `s3://.../curated_layer/tickets_curated/`

### 4️⃣ Query Layer (Athena + Glue Catalog)
- Glue Crawler updates metadata for all layers.
- Athena is used for:
  - Ticket trend analysis
  - CPU usage by user agent
  - Log-level distribution
  - Debug/error pattern analysis

### 5️⃣ Warehouse Layer (Redshift Serverless)
- Processed logs and tickets loaded into:
  - `support_logs`
  - `support_tickets`
- COPY commands use IAM roles for secure S3 → Redshift load.
- Redshift Serverless provides:
  - Auto-scaling
  - No cluster management
  - Fast BI queries

### 6️⃣ Visualization Layer (Power BI)
- Power BI connects to Redshift using ODBC.
- Dashboards include:
  - Ticket Insights: resolution time, agent performance, issue categories, channel distribution
  - Log Insights: CPU usage trends, response time analysis, log-level distribution, user agent activity

---

## 📊 Dashboard Highlights

### 🟠 Ticket Insights
- Total tickets, resolved/open/escalated counts
- Avg resolution time and interactions
- Agent-wise ticket load
- Issue category and priority analysis
- Channel-wise resolution performance

### 🔵 Support Logs
- CPU usage trend by timestamp
- Log level distribution (INFO, DEBUG, WARNING, ERROR)
- Response time range analysis
- User agent activity

---

## 🛠️ Tech Stack

| Layer | Tools |
| --- | --- |
| Raw Layer | Python, Boto3, S3 |
| Processed Layer | AWS Lambda, Glue visual editor |
| Curated Layer | Glue pyspark, lambda |
| Query Layer | Athena, Glue Data Catalog |
| Warehouse | Redshift Serverless |
| Visualization | Power BI |
| Automation | S3 Event Triggers, IAM Roles |

---

## 📁 Repository Structure

```text
project-customer-support-analytics/
│
├── data-ingestion/
│   ├── support-logs/
│   │   ├── support_logs_ingestion_to_S3.ipynb        # Raw log ingestion
│   │   ├── log_date_tracker.txt
│   │   └── sample.env
│   │
│   ├── support-tickets/
│   │   ├── support_tickets_ingestion_to_S3.ipynb     # Raw ticket ingestion
│   │   ├── careplus_support.db.sql
│   │   └── sample.env
│   │
│   └── meta_data.txt
│
├── data-transformation/
│   ├── support-log-transformation/
│   │   └── automate_support_log_ETL.ipynb            # Log transformation
│   │
│   ├── support-tickets-transformation/
│   │   └── automate_support_tickets_ETL_lambda.ipynb # Ticket transformation
│
├── data-warehousing-analytics/
│   ├── redshift-setup/
│   │   └── table-creation-queries.txt                # Redshift DDL
│   │
│   ├── athena-sql-queries/
│   │   └── sql-queries.txt                           # Athena ad-hoc analysis
│   │
│   └── dashboard/
│       └── Careplus Insights.pbix                    # Power BI dashboard
│
├── .gitignore
└── README.md
```

---

## 📈 Key Features

- Fully automated Medallion architecture—no manual intervention.
- Serverless architecture—scalable and cost-efficient.
- Data quality checks—duplicates, invalid timestamps, negative response times removed.
- Parquet optimization—faster query performance.
- Athena for ad-hoc exploration.
- Redshift for BI-grade warehousing.
- Real-time dashboard refresh—BI updates automatically when Redshift data changes.
- Delivered an ACID compliant Medallion pipeline ensuring atomic promotion, consistent schema checks, isolated S3 layers, and durable storage via S3/Redshift.

---

## 📊 Use Cases

- SLA breach analysis
- Agent productivity insights
- Ticket lifecycle analytics
- Application behavior monitoring
- Root-cause analysis
- Operational dashboards

---

## 📬 Contact

For questions or collaboration, feel free to reach out.
