# ML-Augmented Medallion Lakehouse Pipeline for Urban Mobility Analytics
### A Case Study on NYC Yellow Taxi Trip Data (Jan–May 2024)

## Overview
 
This project implements a production-style **end-to-end data engineering pipeline** over 16.79 million NYC yellow taxi trip records, following the **medallion lakehouse architecture** (Bronze → Silver → Gold) with an embedded **machine learning anomaly detection gate**.
 
The pipeline ingests, cleans, enriches, and models trip data for downstream analytics in Power BI — demonstrating core data engineering competencies including distributed processing, Delta Lake storage, multi-source integration, unsupervised ML, and dimensional modelling.
 
---

## Architecture
 
```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
│  NYC TLC Trip Records   TLC Zone Lookup   Open-Meteo Weather API│
└──────────────┬──────────────────┬─────────────────┬────────────┘
               │                  │                 │
               ▼                  ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                     BRONZE LAYER (Delta Lake)                   │
│             Raw ingestion — no transformation                   │
│    trip records / zone_lookup / weather (3,648 hourly rows)     │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SILVER LAYER (Delta Lake)                   │
│  Deduplication · Null filtering · Business rule filters         │
│  Feature engineering (duration, payment label, pickup time)     │
│  Joins: zone lookup (PULocationID / DOLocationID)               │
│         weather (pickup_date + pickup_hour)                     │
│                                                                 │
│  ┌──────────────────────────────────────────────────────┐      │
│  │         Isolation Forest Anomaly Gate (ML)           │      │
│  │  Trained on 10% sample · Scored via Pandas UDF       │      │
│  │  contamination=0.02 · 6 multivariate features        │      │
│  │  → silver_validated  |  → silver_anomalies           │      │
│  └──────────────────────────────────────────────────────┘      │
└──────────────────────────────┬──────────────────────────────────┘
                               │ silver_validated (14.4M rows)
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                      GOLD LAYER (Delta Lake)                    │
│              Star Schema Dimensional Model                      │
│                                                                 │
│   dim_time ──┐                                                  │
│   dim_payment─┼── fact_trips (265,712 aggregated rows)         │
│   dim_location┘                                                 │
│                                                                 │
│              ▼ Parquet export                                   │
│                   Power BI Desktop                              │
└─────────────────────────────────────────────────────────────────┘
```
 
---
 
## Key Features
 
| Feature | Detail |
|---|---|
| **Scale** | 16.79 million raw trip records (Jan–May 2024) |
| **Storage format** | Delta Lake on Google Drive (ACID, schema enforcement, time travel) |
| **Processing engine** | Apache Spark (PySpark) on Google Colab |
| **Data sources** | 3 integrated — TLC trip records, TLC zone lookup CSV, Open-Meteo hourly weather API |
| **ML anomaly gate** | Isolation Forest (scikit-learn) — unsupervised, 2% contamination, 6 features |
| **Dimensional model** | Star schema — `fact_trips`, `dim_time`, `dim_payment`, `dim_location` |
| **Visualisation** | Power BI Desktop via Parquet export |
 
---
 
## Data Sources
 
### 1. NYC TLC Yellow Taxi Trip Records
- **Period:** January – May 2024
- **Records:** 16,792,900
- **Source:** [NYC Taxi & Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) (Parquet files)
- **Key fields:** pickup/dropoff datetimes, location IDs, fare amount, trip distance, passenger count, payment type
### 2. TLC Taxi Zone Lookup
- **Records:** 265 zones
- **Source:** [TLC static CSV](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv)
- **Use:** Enriches trips with borough, zone name, and service zone for both pickup and dropoff locations
### 3. Open-Meteo Historical Weather API
- **Records:** 3,648 hourly observations
- **Period:** 2024-01-01 to 2024-05-31, New York City (40.71°N, 74.01°W)
- **Features:** `temperature_c`, `precipitation_mm`, `windspeed_kmh`
- **Join key:** `pickup_date` + `pickup_hour`
---
