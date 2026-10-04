# NYC Taxi Data Pipeline: Medallion Architecture

This repository implements a production-ready **Medallion Architecture (Bronze → Silver → Gold)** data pipeline using **PySpark** and **Delta Lake**. 
It simulates a stream of New York City Yellow Taxi trip data, cleans and refines it into structured formats, and delivers aggregated high-value business metrics for downstream Business Intelligence (BI) and reporting.

---

## 🏗️ Architecture Overview

The pipeline organizes data into three distinct architectural layers to guarantee quality, lineage, and performance:

```
[ Raw CSV Stream ] ──> 🥉 BRONZE LAYER (Permissive Ingestion)
                             │
                             ▼
                       🥈 SILVER LAYER (Cleaned, Enriched & Partitioned)
                             │
                             ▼
                       🥇 GOLD LAYER (Pre-aggregated Analytical Metrics)
```

### 1. 🥉 Bronze Layer (Ingestion)
* **Purpose**: Ingests raw data as-is from the land zone without complex schema transformations.
* **Implementation**: Reads incoming `generated_trips_raw.csv` files with an explicit schema wrapper using PySpark's `PERMISSIVE` mode to prevent schema corruption failures.
* **Schema Alignments**: Resolves naming mismatches between standard raw dumps (e.g., `tpep_pickup_datetime`) and target system.

### 2. 🥈 Silver Layer (Cleaned & Enriched)
* **Purpose**: Serves as the single source of truth for cleaned, parsed, and trustworthy enterprise records.
* **Data Quality Guardrails**: Drops anomalous entries like 0-passenger counts, negative fares, or empty trip distances.
* **Feature Engineering**: Derives `trip_duration_minutes`, assigns temporal attributes, and drops exact row duplicates using business key criteria.
* **Storage Optimization**: Persisted as a **Delta Lake table**, heavily optimized by partitioning on `pickup_year` and `pickup_month`.

### 3. 🥇 Gold Layer (Aggregated Business Analytics)
* **Purpose**: Power business dashboards, executives, and reporting tools with ultra-low latency.
* **Implementation**: Groups records by year, month, and vendor to pre-calculate high-performance metrics including:
  * Total trip frequencies and customer volumes.
  * `avg_revenue_per_trip` & `avg_trip_duration_minutes`.
  * `tip_percentage_of_fares` (Operational tipping rate performance indicator).

---

## 📂 Storage & Volume Layout

The pipeline expects and creates the following directory structure inside your localized development volume mount points:

```text
/Volumes/dev_nyc/
├── landing/               # Raw incoming data landing pad
│   └── nyc_taxi_trip_duration/generated_trips_raw.csv
|--bronze/                 # Raw and Immutable
|   |__nyc_taxi_trip_duration/raw_data
├── silver/                # Cleaned, structured Delta tables
│   └── nyc_taxi_trip_duration/cleaned_trips/
└── gold/                  # Analytical, dashboard-ready aggregates
    └── nyc_taxi_trip_duration/monthly_vendor_performance/
```

---



```


1. **Schema Enforcement over Inference**: Swapping `inferSchema: true` for an explicit schema speeds up processing drastically by avoiding pre-scan Spark jobs.
2. **Delta Lake Format Execution**: Ensures ACID compliance, metadata version management (Time Travel), and allows for high-speed file skipping.
3. **Partition Pruning**: Segmenting data physically by year and month guarantees downstream consumers only read folders matching their analytical target scope.
