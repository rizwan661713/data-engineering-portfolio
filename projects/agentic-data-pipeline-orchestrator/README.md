# Agentic Data Pipeline Orchestrator

## Project Overview

This project demonstrates an AI-assisted data pipeline orchestration framework that helps monitor pipeline health, detect failures, summarize issues, and recommend next actions for data engineering teams.

The goal is to combine traditional data engineering orchestration with agentic AI concepts so pipelines are easier to operate, troubleshoot, and maintain.

## Business Problem

Modern data platforms often run many pipelines across ingestion, transformation, validation, and reporting layers. When a pipeline fails, data engineers usually need to manually check logs, identify the failed step, review recent changes, validate upstream files, and decide whether to retry, fix, or escalate.

This becomes difficult when there are:

- Multiple source systems
- Daily and hourly pipeline schedules
- Schema changes
- Missing files
- Data quality failures
- Downstream dashboard dependencies
- Limited visibility into root cause

This project solves that by adding an AI-assisted orchestration layer that can analyze pipeline metadata, logs, and data quality results to produce clear operational recommendations.

## Solution Approach

The solution uses a metadata-driven pipeline framework with an AI agent layer.

The framework captures:

- Pipeline run history
- Task-level execution status
- Source file arrival status
- Schema validation results
- Data quality check results
- Error logs
- Retry attempts
- Downstream impact
- Suggested resolution steps

The AI assistant reviews this operational metadata and produces a clear summary for data engineers.

## Architecture

```text
Source Systems / Files / APIs
   |
   v
Pipeline Orchestrator
Airflow / Databricks Workflows / dbt Cloud Jobs
   |
   v
Pipeline Metadata Store
Run status, logs, checks, dependencies
   |
   v
Data Quality Layer
Validation rules and anomaly checks
   |
   v
AI Agent Layer
Summarize the issue, detect the likely root cause, and recommend action
   |
   v
Notification Layer
Slack / Email / Ticket / Dashboard
```
## Core Components

### 1. Pipeline Metadata Store

Purpose:
- Track every pipeline run
- Store run status and timestamps
- Capture failed task names
- Store retry counts
- Capture upstream and downstream dependencies

Example table:

```sql
create or replace table ops.pipeline_run_history (
    pipeline_run_id varchar,
    pipeline_name varchar,
    environment varchar,
    status varchar,
    start_time timestamp,
    end_time timestamp,
    duration_seconds number,
    failed_task_name varchar,
    error_message varchar,
    retry_count number,
    triggered_by varchar,
    created_at timestamp
);
```

### 2. Task Dependency Map

Purpose:
- Understand which tasks depend on each other
- Identify downstream impact when a task fails
- Support better root cause analysis

Example table:

```sql
create or replace table ops.pipeline_dependency_map (
    pipeline_name varchar,
    task_name varchar,
    upstream_task_name varchar,
    downstream_task_name varchar,
    target_table varchar,
    downstream_report varchar,
    business_owner varchar
);
```

### 3. Data Quality Results

Purpose:
- Store validation results from each pipeline run
- Track failed checks
- Identify recurring issues
- Support AI-generated issue summaries

Example table:

```sql
create or replace table ops.data_quality_results (
    check_id varchar,
    pipeline_run_id varchar,
    table_name varchar,
    check_name varchar,
    check_type varchar,
    status varchar,
    failed_record_count number,
    threshold_value number,
    actual_value number,
    check_timestamp timestamp
);
```

## Example Data Quality Checks

### Missing Required Fields

```sql
select *
from silver_customer_activity
where customer_id is null
   or event_timestamp is null;
```

### Duplicate Business Keys

```sql
select
    customer_id,
    event_timestamp,
    count(*) as record_count
from silver_customer_activity
group by
    customer_id,
    event_timestamp
having count(*) > 1;
```

### Record Count Anomaly

```sql
select
    process_date,
    count(*) as record_count
from silver_customer_activity
group by process_date
having count(*) < 1000;
```

## AI Agent Workflow

The AI agent follows a simple operational reasoning flow:

```text
1. Read latest failed pipeline run
2. Review failed task name and error message
3. Check source file arrival status
4. Review schema validation results
5. Review data quality failures
6. Check downstream impacted tables and reports
7. Generate plain-English summary
8. Recommend next action
9. Send notification to engineering team
```

## Example AI-Generated Incident Summary

```text
Pipeline: customer_activity_daily_load
Status: Failed
Failed Task: validate_schema
Likely Cause: Source file is missing the required column customer_id
Downstream Impact: gold_customer_activity_summary and Executive Activity Dashboard may not refresh
Recommended Action: Contact source system owner or update schema mapping if this is an approved change
Priority: High
```

## Example Python Agent Logic

```python
def analyze_pipeline_failure(run_metadata, dq_results, dependency_map):
    summary = {
        "pipeline_name": run_metadata.get("pipeline_name"),
        "status": run_metadata.get("status"),
        "failed_task": run_metadata.get("failed_task_name"),
        "likely_cause": None,
        "downstream_impact": [],
        "recommended_action": None,
        "priority": "Medium"
    }

    error_message = run_metadata.get("error_message", "").lower()

    if "missing column" in error_message or "schema" in error_message:
        summary["likely_cause"] = "Schema change detected in source data"
        summary["recommended_action"] = "Review source schema and update mapping if the change is approved"
        summary["priority"] = "High"

    elif "file not found" in error_message:
        summary["likely_cause"] = "Expected source file did not arrive"
        summary["recommended_action"] = "Check source delivery process and confirm file availability"
        summary["priority"] = "High"

    elif dq_results and any(row["status"] == "failed" for row in dq_results):
        summary["likely_cause"] = "Data quality validation failed"
        summary["recommended_action"] = "Review failed data quality checks and sample invalid records"
        summary["priority"] = "Medium"

    else:
        summary["likely_cause"] = "Unknown pipeline failure"
        summary["recommended_action"] = "Review detailed logs and retry after validation"

    summary["downstream_impact"] = [
        item["downstream_report"]
        for item in dependency_map
        if item.get("pipeline_name") == run_metadata.get("pipeline_name")
    ]

    return summary
```

## Example Notification Message

```text
Data Pipeline Alert

Pipeline: customer_activity_daily_load
Environment: Production
Status: Failed
Failed Task: validate_schema
Likely Cause: Schema change detected in source data
Impact: Executive Activity Dashboard may not refresh
Recommended Action: Review source schema and update mapping if approved
```

## Orchestration Pattern

This project can be implemented with:

- Airflow DAGs
- Databricks Workflows
- dbt Cloud jobs
- GitHub Actions
- Snowflake Tasks
- Azure Data Factory

Example orchestration flow:

```text
Start Pipeline
   |
   v
Check Source File Arrival
   |
   v
Run Bronze Ingestion
   |
   v
Validate Schema
   |
   v
Run Silver Transformation
   |
   v
Run Data Quality Checks
   |
   v
Run Gold Aggregation
   |
   v
Update Pipeline Metadata
   |
   v
AI Agent Failure Summary
   |
   v
Send Notification
```

## Failure Categories

The orchestrator classifies failures into categories:

- Missing source file
- Empty file
- Schema change
- Data type mismatch
- Duplicate records
- Null required fields
- Record count anomaly
- Transformation failure
- Downstream dependency failure
- Unknown system error

## Monitoring Dashboard Metrics

Example operational metrics:

- Total pipeline runs
- Successful runs
- Failed runs
- Average runtime
- Retry count
- Most common failure category
- Failed data quality checks
- Impacted downstream reports
- Mean time to resolution
- Repeated failure patterns

## Governance and Control

This project supports production-style controls such as:

- Environment-based deployment
- Role-based access to operational metadata
- Audit history for pipeline runs
- Clear ownership for pipelines and reports
- Controlled retry and escalation logic
- Documentation for failure handling

## Technologies Used

- Python
- SQL
- Databricks Workflows
- Airflow concepts
- dbt Cloud jobs
- Snowflake
- Delta Lake
- GitHub Actions concepts
- Data quality checks
- AI/LLM-assisted incident summarization
- Slack or email alerting

## What This Project Demonstrates

This project shows the ability to:

- Design production-style pipeline orchestration
- Track pipeline metadata and task dependencies
- Build data quality validation into pipelines
- Use AI to summarize failures and recommend actions
- Improve data engineering operations and incident response
- Reduce manual troubleshooting time
- Support reliable analytics and downstream reporting
- Combine modern data engineering with agentic AI concepts

## Status

This is a portfolio case study based on modern data engineering and AI-assisted operations patterns. All project details are anonymized and do not include private client data, credentials, or proprietary code.



