# Enterprise Data Governance & Cataloging

## Project Overview

This project demonstrates an enterprise data governance and cataloging approach for improving trust, visibility, ownership, and usability of data assets across an organization.

The goal is to help business and technical teams understand what data exists, where it comes from, who owns it, how it is used, and whether it can be trusted for reporting and analytics.

## Business Problem

Many organizations have data spread across databases, reporting tools, spreadsheets, pipelines, and cloud platforms. Over time, teams lose visibility into where key data lives, how metrics are defined, who owns the data, and whether reports are using trusted sources.

This creates common issues such as:

- Duplicate reports and datasets
- Unclear business definitions
- Low trust in reporting numbers
- Unknown data ownership
- Limited visibility into lineage
- Difficulty finding approved data assets
- Weak documentation around critical tables and metrics

## Solution Approach

The solution focuses on building a practical governance framework around metadata, ownership, data quality, and lineage.

Key areas include:

- Cataloging important tables, dashboards, and data assets
- Defining business-friendly descriptions
- Assigning data owners and stewards
- Documenting data lineage from source to reporting
- Creating standard business definitions for key metrics
- Identifying trusted and certified data assets
- Supporting data quality checks and issue tracking

## Governance Architecture

```text
Source Systems
   |
   v
Data Pipelines / ETL / ELT
   |
   v
Cloud Data Platform
   |
   v
Curated Data Models
   |
   v
BI Reports and Dashboards
   |
   v
Data Catalog and Governance Layer
```

## Key Governance Domains

### Metadata Management

Purpose:
- Document tables, columns, reports, and dashboards
- Capture technical and business descriptions
- Improve searchability of data assets
- Help users understand how data should be used

Example metadata fields:
- asset_name
- asset_type
- database_name
- schema_name
- table_name
- column_name
- business_description
- technical_description
- owner
- steward
- refresh_frequency
- sensitivity_level

### Data Ownership

Purpose:
- Assign clear accountability for important data assets
- Identify who can answer questions about the data
- Support issue resolution and approval workflows

Example ownership roles:
- Data Owner
- Data Steward
- Technical Owner
- Business SME
- Report Owner

### Business Glossary

Purpose:
- Standardize definitions for important business terms and metrics
- Reduce confusion across teams
- Create a shared language between business and technical users

Example glossary terms:
- Active Customer
- Net Revenue
- Claim Count
- Enrollment Date
- Program Start Date
- Implementation Status
- Location Event
- Unique User

### Data Lineage

Purpose:
- Show how data flows from source systems to final reports
- Help troubleshoot reporting issues
- Improve impact analysis before changes
- Support audit and compliance needs

Example lineage flow:

```text
Salesforce Account
   |
   v
Raw Account Table
   |
   v
Staging Account Model
   |
   v
Customer Dimension
   |
   v
Executive Dashboard
```

### Data Quality

Purpose:
- Identify issues before they impact reporting
- Track completeness, accuracy, uniqueness, and consistency
- Build trust in critical data products

Example data quality checks:
- Required field is not null
- Primary key is unique
- Record count is within expected range
- Date fields are valid
- Reference values match approved lists
- No duplicate business keys
- Refresh completed successfully

## Example Data Quality Rules

```sql
-- Check for duplicate customer IDs

select
    customer_id,
    count(*) as record_count
from dim_customer
group by customer_id
having count(*) > 1;
```

```sql
-- Check for missing required business fields

select *
from fct_program_activity
where program_id is null
   or activity_date is null;
```

```sql
-- Check for invalid status values

select *
from fct_project_status
where status not in (
    'Not Started',
    'In Progress',
    'Complete',
    'Blocked',
    'On Hold'
);
```

## Example Governance Metadata Table

```sql
create or replace table governance.data_asset_inventory (
    asset_id varchar,
    asset_name varchar,
    asset_type varchar,
    database_name varchar,
    schema_name varchar,
    table_name varchar,
    business_description varchar,
    technical_description varchar,
    data_owner varchar,
    data_steward varchar,
    refresh_frequency varchar,
    sensitivity_level varchar,
    certification_status varchar,
    created_at timestamp,
    updated_at timestamp
);
```

## Example Certified Data Asset Workflow

```text
1. Identify critical data asset
2. Document business and technical metadata
3. Assign owner and steward
4. Validate source-to-target lineage
5. Add data quality checks
6. Review with business stakeholders
7. Mark asset as trusted or certified
8. Monitor quality and usage over time
```

## Catalog Implementation Pattern

The cataloging approach includes:

- Prioritizing high-value datasets first
- Documenting tables and columns used in key reports
- Connecting business terms to physical data assets
- Creating ownership and stewardship fields
- Capturing lineage from source to dashboard
- Tagging sensitive or restricted fields
- Identifying certified datasets for reporting

## Example Catalog Fields

```text
Asset Name: dim_customer
Asset Type: Table
Business Description: Standard customer dimension used for reporting and analytics.
Owner: Sales Operations
Steward: Data Governance Team
Technical Owner: Data Engineering
Refresh Frequency: Daily
Certification Status: Certified
Sensitivity Level: Internal
```

## Tools and Technologies Used

- Alation
- SQL
- Snowflake
- Databricks
- Power BI
- Metadata Management
- Data Cataloging
- Data Quality
- Data Lineage
- Data Governance

## What This Project Demonstrates

This project shows the ability to:

- Build a practical data governance framework
- Document technical and business metadata
- Define ownership and stewardship models
- Improve trust in reporting and analytics assets
- Support data lineage and impact analysis
- Create reusable data quality checks
- Help business users find and understand approved data assets
- Bridge the gap between data engineering, analytics, and business teams

## Status

This is a portfolio case study based on real-world enterprise data governance patterns. All project details are anonymized and do not include private client data, credentials, or proprietary code.








