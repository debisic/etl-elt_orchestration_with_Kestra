# Kestra Orchestration for NYC Taxi Data Pipeline

A comprehensive data engineering project demonstrating workflow orchestration using [Kestra](https://kestra.io/) to build ELT pipelines for NYC taxi data. This project showcases both PostgreSQL and Google Cloud Platform (GCP) BigQuery implementations with various scheduling and backfill capabilities.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Workflows](#workflows)
- [Configuration](#configuration)
- [Data Quality Validation Queries](#data-quality-validation-queries)
- [Acknowledgments](#acknowledgments)



## 🎯 Overview

This project demonstrates modern data engineering practices using Kestra for workflow orchestration. It processes NYC taxi trip data (yellow and green taxi datasets) from the [DataTalksClub repository](https://github.com/DataTalksClub/nyc-tlc-data/releases) and implements:

- **Manual and scheduled workflows** for data ingestion
- **ELT pipelines** to PostgreSQL and Google BigQuery
- **Data deduplication** using unique row identifiers
- **Infrastructure as Code** for GCP resources
- **Backfill capabilities** for historical data processing
- **Staging and merge patterns** for data quality

### Key Features

✅ Manual and automated data extraction from external sources  
✅ PostgreSQL integration with staging tables and MERGE operations  
✅ Google Cloud Storage (GCS) and BigQuery integration  
✅ Partitioned tables for optimized query performance  
✅ Idempotent workflows with deduplication logic  
✅ Scheduled execution with cron triggers  
✅ Infrastructure provisioning through Kestra  
✅ Docker-based local development environment

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Data Source (GitHub)                      │
│        NYC TLC Taxi Data (Compressed CSV files)             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │   Kestra Workflows    │
         │   (Orchestration)     │
         └───────┬───────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
┌──────────────┐  ┌──────────────────┐
│  PostgreSQL  │  │  Google Cloud    │
│   Database   │  │  Platform (GCP)  │
│              │  │                  │
│ • Staging    │  │ • Cloud Storage  │
│ • Final      │  │ • BigQuery       │
│   Tables     │  │ • Partitioned    │
└──────────────┘  │   Tables         │
                  └──────────────────┘
```

### Technology Stack

- **Orchestration**: Kestra v1.1
- **Databases**: PostgreSQL 18
- **Cloud Platform**: Google Cloud Platform (BigQuery, GCS)
- **Containerization**: Docker & Docker Compose
- **Administration**: pgAdmin 4
- **Data Format**: CSV (compressed with gzip)

## 📦 Prerequisites

Before getting started, ensure you have the following installed:

- **Docker** (v20.10+) and **Docker Compose** (v2.0+)
- **Git** for cloning the repository
- **GCP Account** with billing enabled (for GCP workflows)
- **GCP Service Account** with appropriate permissions (for GCP workflows)

### Required GCP Permissions

If using GCP workflows, your service account needs:
- `Storage Admin` or `Storage Object Admin` (for GCS)
- `BigQuery Admin` or `BigQuery Data Editor` (for BigQuery)
- Ability to create datasets and tables

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/debisic/etl-elt_orchestration_with_Kestra.git
cd etl-elt_orchestration_with_Kestra
```

### 2. Set Up GCP Credentials (Optional - for GCP workflows)

If you plan to use GCP workflows, you'll need to configure your GCP service account credentials using Kestra's secrets management.

**Watch this video tutorial for detailed setup instructions:**

📺 [Kestra Secrets Management Guide](https://www.youtube.com/watch?v=u0yuOYG-qMI)

The video covers how to securely store and reference your GCP service account credentials in Kestra workflows.

### 3. Start the Services

```bash
docker-compose up
```

This will start:
- **PostgreSQL** (port 5432) - Database for taxi data
- **pgAdmin** (port 8085) - Database administration UI
- **Kestra Postgres** - Kestra's metadata database
- **Kestra** (port 8080) - Orchestration platform

### 4. Access Kestra UI

Open your browser and navigate to:

```
http://localhost:8080
```

**Login credentials:**
- Email: `admin@kestra.io`
- Password: `Admin1234`

### 5. Access pgAdmin (Optional)

Navigate to:

```
http://localhost:8085
```

**Login credentials:**
- Email: `admin@example.com`
- Password: `root`

## 📁 Project Structure

```
orchestkestra/
│
├── 01_manual_monthly_flow.yml          # Manual PostgreSQL ingestion
├── 02_schedule_backfill.yml            # Scheduled PostgreSQL ingestion
├── 03_gcp_kv.yml                       # GCP configuration (key-value store)
├── 04_gcp_infra_provisioning.yml       # GCP infrastructure setup
├── 05_gcp_taxi_elt.yml                 # Manual GCP ELT pipeline
├── 06_gcp_taxi_elt_scheduled.yml       # Scheduled GCP ELT pipeline
│
├── docker-compose.yml                  # Docker services configuration
├── kestra-elt_key.json                 # GCP service account key (not in git)
├── .env_encoded                        # Encoded GCP credentials (not in git)
└── README.md                           # This file
```

## 🔄 Workflows

### PostgreSQL Workflows

#### 1. Manual Monthly Flow (`01_manual_monthly_flow.yml`)

**Purpose**: Manually ingest NYC taxi data into PostgreSQL for a specific month.

**Features:**
- Interactive UI with dropdown selections for taxi type, year, and month
- Downloads compressed CSV data from GitHub
- Creates staging and final tables
- Uses MERGE operation for deduplication
- MD5 hashing for unique row identification

**Usage:** Navigate to Flows → zoomcamp → `01_postgres_taxi` → Execute and select taxi type, year, and month.

#### 2. Scheduled Backfill Flow (`02_schedule_backfill.yml`)

Automated monthly ingestion with scheduling support. Runs on the 1st of each month (Green: 9:00 AM, Yellow: 10:00 AM) with concurrency limit and backfill capability.

### GCP Workflows

#### 3. GCP Key-Value Configuration (`03_gcp_kv.yml`)

Stores GCP configuration (project ID, location, bucket name, dataset) as Kestra key-value pairs. Execute once to configure; other workflows reference these values.

#### 4. GCP Infrastructure Provisioning (`04_gcp_infra_provisioning.yml`)

Creates GCS bucket and BigQuery dataset. Run once before ELT pipelines.

#### 5. GCP Taxi ELT (`05_gcp_taxi_elt.yml`)

Manual ELT pipeline: Downloads CSV → Uploads to GCS → Creates external table → Generates unique IDs → Merges into partitioned BigQuery table with deduplication.

#### 6. GCP Scheduled ELT (`06_gcp_taxi_elt_scheduled.yml`)

Automated version of workflow #5. Runs monthly on the 1st (Green: 9:00 AM, Yellow: 10:00 AM).

## ⚙️ Configuration

### Database Connection

PostgreSQL connection details are defined in workflow `pluginDefaults`:

```yaml
pluginDefaults:
  - type: io.kestra.plugin.jdbc.postgresql
    values:
      url: jdbc:postgresql://postgres_db:5432/ny_green_taxi
      username: green
      password: green
```

### GCP Configuration

GCP settings are stored as Kestra KV pairs and referenced in workflows:

```yaml
pluginDefaults:
  - type: io.kestra.plugin.gcp
    values:
      serviceAccount: "{{ kv('GCP_SERVICE_ACCOUNT') }}"
      projectId: "{{ kv('GCP_PROJECT_ID') }}"
      location: "{{ kv('GCP_LOCATION') }}"
      bucket: "{{ kv('GCP_BUCKET_NAME') | trim }}"
```

## 📊 Data Quality Validation Queries

### Querying Data

**PostgreSQL via pgAdmin:**
```sql
SELECT COUNT(*) FROM public.green_tripdata;
SELECT * FROM public.green_tripdata LIMIT 10;
```

**BigQuery via GCP Console:**
```sql
SELECT COUNT(*) FROM `kestra-orchestration-486016.ny_taxi.green_tripdata`;
SELECT * FROM `kestra-orchestration-486016.ny_taxi.green_tripdata` 
WHERE DATE(lpep_pickup_datetime) = '2019-01-01' 
LIMIT 10;
```

### Validation Queries

**Check for records outside expected year (Yellow Taxi):**
```sql
-- Count yellow taxi records outside the expected for the whole of 2020
SELECT COUNT(*)
FROM `kestra-orchestration-486016.ny_taxi.yellow_tripdata`
WHERE EXTRACT(YEAR FROM tpep_pickup_datetime) = 2019;
```

**Check for records outside expected year (Green Taxi):**
```sql
-- Count green taxi records for whole of 2020
SELECT COUNT(*)
FROM `kestra-orchestration-486016.ny_taxi.green_tripdata`
WHERE EXTRACT(YEAR FROM lpep_pickup_datetime) = 2020;
```

**Query specific year data (Yellow Taxi):**
```sql
-- count yellow taxi records for march 2021, query works when only march data is ingested for 2021
SELECT count(*)
FROM `kestra-orchestration-486016.ny_taxi.yellow_tripdata`
WHERE EXTRACT(YEAR FROM tpep_pickup_datetime) = 2021;
```

**Note**: Replace `kestra-orchestration-486016` with your actual GCP project ID.


Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request


## 📄 License

This project is created for educational purposes as part of the DataTalks.Club Data Engineering Zoomcamp.

## 🙏 Acknowledgments

- [DataTalksClub](https://datatalks.club/) for the Data Engineering Zoomcamp
- [Kestra](https://kestra.io/) for the excellent orchestration platform
- [NYC TLC](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) for providing the taxi trip data
