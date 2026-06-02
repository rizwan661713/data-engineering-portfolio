# Snowflake + dbt Analytics Foundation

## Project Overview

This project demonstrates how to design a modern Snowflake and dbt analytics foundation for moving business logic out of Power BI and into reusable, governed transformation models.

The goal is to create a cleaner, more maintainable data platform where source data is transformed through structured dbt layers and made available for reporting, validation, and future analytics use cases.

## Business Problem

Many organizations have important transformation logic embedded inside Power BI dataflows, semantic models, and report-level calculations. This makes reporting harder to maintain, harder to test, and harder to reuse across teams.

This project solves that by moving key transformation logic upstream into Snowflake and dbt.

## Solution Approach

The solution follows a layered analytics engineering pattern:

- Source data lands in Snowflake
- dbt staging models clean and standardize raw source tables
- Intermediate models apply business logic and joins
- Final reporting models provide analytics-ready outputs for Power BI
- GitHub supports version control and controlled promotion
- dbt tests and documentation improve trust and maintainability

## Architecture

```text
Source Systems
   |
   v
Snowflake Raw / Landing Layer
   |
   v
dbt Staging Models
   |
   v
dbt Intermediate Models
   |
   v
Final Analytics Models
   |
   v
Power BI Reports
```

## Example Model Layers

### Staging Layer

Purpose:
- Rename columns
- Standardize data types
- Remove unnecessary fields
- Apply light cleanup
- Keep one model close to one source table

Example models:
- stg_salesforce__accounts
- stg_salesforce__leads
- stg_asana__projects
- stg_asana__tasks

### Intermediate Layer

Purpose:
- Apply business rules
- Join related source data
- Normalize project and organization logic
- Prepare reusable building blocks

Example models:
- int_salesforce__healthcare_organizations
- int_asana__project_tasks
- int_program__milestone_events
- int_program__status_history

### Final Analytics Layer

Purpose:
- Provide clean reporting-ready datasets
- Support Power BI dashboards
- Reduce duplicate report logic
- Improve consistency across metrics

Example models:
- dim_healthcare_organization
- dim_program_project
- fct_milestone
- fct_project_status
- rpt_program_implementation_summary

## Key Features

- Snowflake database and schema design
- dbt project structure
- Staging, intermediate, and final model layers
- Git-based development workflow
- CI validation before merging to main branches
- Power BI repointing support
- Data quality and metric validation
- Documentation for client handoff

## Technologies Used

- Snowflake
- dbt Cloud
- SQL
- GitHub
- Power BI
- Data modeling
- CI/CD concepts

## What This Project Demonstrates

This project shows the ability to:

- Design a scalable analytics engineering foundation
- Convert Power BI transformation logic into reusable SQL/dbt models
- Build clean dimensional and reporting models
- Support governed development using Git workflows
- Improve reporting reliability through validation and testing
- Create documentation that allows business users and analysts to maintain the platform

## Sample dbt Test Examples

```yaml
version: 2

models:
  - name: dim_healthcare_organization
    description: Healthcare organization dimension used for program implementation reporting.
    columns:
      - name: healthcare_organization_id
        tests:
          - not_null
          - unique

  - name: fct_milestone
    description: Milestone fact table used to track project implementation progress.
    columns:
      - name: milestone_id
        tests:
          - not_null
          - unique
      - name: milestone_date
        tests:
          - not_null
```

## Status

This is a portfolio case study based on real-world data engineering patterns. All project details are anonymized and do not include private client data, credentials, or proprietary code.

