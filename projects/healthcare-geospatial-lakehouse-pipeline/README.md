# Healthcare & Geospatial Lakehouse Pipeline

## Project Overview

This project demonstrates a modern lakehouse data engineering pipeline for processing healthcare-related and geospatial activity data at scale.

The solution is designed to ingest raw files, clean and standardize records, apply business rules, enrich location-based data, validate data quality, and produce curated datasets for analytics and downstream consumption.

## Business Problem

Organizations that collect large volumes of activity, location, and healthcare-related data often struggle with inconsistent files, changing schemas, duplicate records, missing fields, and limited visibility into data quality.

Without a structured pipeline, it becomes difficult to trust the data, support reporting, or prepare datasets for external use cases.

## Solution Approach

The solution follows a medallion lakehouse architecture:

- Bronze layer stores raw ingested data
- Silver layer cleans, standardizes, and validates records
- Gold layer creates curated analytics-ready datasets
- Data quality checks monitor schema changes, empty files, missing values, and record counts
- Incremental processing reduces unnecessary reprocessing
- Governance concepts support controlled access and future data sharing

## Architecture

```text
Source Files / APIs
   |
   v
Cloud Storage Landing Zone
   |
   v
Bronze Layer
Raw ingested data with audit fields
   |
   v
Silver Layer
Cleaned, standardized, validated records
   |
   v
Gold Layer
Curated analytics-ready datasets
   |
   v
BI, Analytics, Data Sharing, Marketplace

```
```

## Data Pipeline Layers

### Bronze Layer

Purpose:
- Ingest raw source files
- Preserve original records
- Add ingestion metadata
- Track file name, load timestamp, process date, and source system
- Support replay and auditability

Example fields:
- source_file_name
- load_timestamp
- process_date
- source_system
- raw_payload
- latitude
- longitude
- user_id
- event_timestamp

### Silver Layer

Purpose:
- Clean and standardize raw records
- Cast fields into correct data types
- Remove duplicates
- Validate required fields
- Standardize timestamps
- Prepare geospatial and business attributes

Example transformations:
- Convert raw timestamp values into standard timestamp fields
- Validate latitude and longitude ranges
- Remove invalid or duplicate records
- Standardize user and device identifiers
- Extract city, state, and ZIP-level location attributes where available

### Gold Layer

Purpose:
- Create curated datasets for reporting and external use
- Aggregate activity at useful business grains
- Support high-confidence analytics outputs
- Prepare data for BI dashboards, exports, or sharing

Example outputs:
- gold_user_activity_daily
- gold_geospatial_activity_summary
- gold_state_activity_summary
- gold_user_engagement_metrics
- gold_high_confidence_activity_dataset

## Data Quality Framework

This project includes validation checks such as:

- Missing file detection
- Empty file detection
- Schema change detection
- Record count anomaly checks
- Duplicate record checks
- Null checks for required fields
- Latitude and longitude range validation
- Timestamp completeness checks

## Example Data Quality Rules

```sql
-- Check for invalid latitude and longitude values

select *
from silver_geospatial_activity
where latitude < -90
   or latitude > 90
   or longitude < -180
   or longitude > 180;
```

```sql
-- Check for missing required identifiers

select *
from silver_user_activity
where user_id is null
   or event_timestamp is null;
```

## Incremental Processing Pattern

The pipeline is designed to process only new or changed data when possible.

Example logic:

```python
from pyspark.sql import functions as F

new_data = (
    spark.read.format("parquet")
    .load("/mnt/landing/activity/")
    .filter(F.col("process_date") >= F.current_date())
)

clean_data = (
    new_data
    .dropDuplicates(["user_id", "event_timestamp", "latitude", "longitude"])
    .withColumn("load_timestamp", F.current_timestamp())
)
```

## Example Silver Transformation

```python
from pyspark.sql import functions as F

silver_activity = (
    bronze_activity
    .withColumn("event_timestamp", F.to_timestamp("event_timestamp"))
    .withColumn("latitude", F.col("latitude").cast("double"))
    .withColumn("longitude", F.col("longitude").cast("double"))
    .filter(F.col("user_id").isNotNull())
    .filter(F.col("event_timestamp").isNotNull())
    .filter((F.col("latitude").between(-90, 90)) & (F.col("longitude").between(-180, 180)))
    .dropDuplicates(["user_id", "event_timestamp", "latitude", "longitude"])
)
```

## Example Gold Aggregation

```sql
create or replace table gold_state_activity_summary as

select
    state,
    process_date,
    count(*) as total_events,
    count(distinct user_id) as unique_users,
    min(event_timestamp) as first_event_timestamp,
    max(event_timestamp) as last_event_timestamp
from silver_geospatial_activity
where state is not null
group by
    state,
    process_date;
```

## Monitoring and Alerting

The pipeline includes operational checks to identify issues before data reaches the reporting layer.

Example alert scenarios:
- Expected file did not arrive
- File arrived but contains zero records
- Required column is missing
- Record count drops below expected threshold
- Invalid location values exceed acceptable limits

## Governance and Security Concepts

The project supports governed data access patterns such as:

- Separate environments for development, testing, and production
- Controlled access by data layer
- Limited access to sensitive fields
- Audit-friendly ingestion metadata
- Curated Gold datasets for approved consumption
- Data sharing readiness through controlled final outputs

## Technologies Used

- Databricks
- PySpark
- SQL
- Delta Lake
- Azure Data Lake / Blob Storage
- Unity Catalog concepts
- Python
- Data quality checks
- Lakehouse architecture

## What This Project Demonstrates

This project shows the ability to:

- Build scalable lakehouse pipelines
- Design Bronze, Silver, and Gold data layers
- Process high-volume geospatial activity data
- Apply data quality and validation rules
- Build curated datasets for analytics and sharing
- Support governance and access control concepts
- Design production-style data engineering workflows

## Status

This is a portfolio case study based on real-world data engineering patterns. All project details are anonymized and do not include private client data, credentials, or proprietary code.




