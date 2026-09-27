AWS Data Engineering Pipeline

An end-to-end AWS data engineering pipeline that extracts operational data from Amazon RDS, replicates it using AWS Database Migration Service (DMS), processes it through an S3-based Bronze/Silver data lake architecture, loads curated data into Amazon Redshift, and makes the data available for analytics through Amazon QuickSight.

The pipeline is designed around AWS managed services, workflow orchestration, secure networking, and automated data transformation.

Architecture

                         AWS Data Engineering Pipeline

   ┌──────────────┐
   │  Amazon RDS  │
   │    Source    │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │    AWS DMS   │
   │ Replication  │
   └──────┬───────┘
          │
          ▼
   ┌────────────────────┐
   │     S3 Bronze      │
   │    Raw Data Lake   │
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │ AWS Glue Crawler   │
   │ + Glue Data Catalog│
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │     AWS Glue       │
   │ ETL / Transformation│
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │     S3 Silver      │
   │ Cleaned Data Lake  │
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │     AWS Glue       │
   │ Warehouse Transform│
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │   Amazon Redshift  │
   │                    │
   │  ┌──────────────┐  │
   │  │   Staging    │  │
   │  └──────┬───────┘  │
   │         ▼          │
   │  ┌──────────────┐  │
   │  │    Target    │  │
   │  └──────────────┘  │
   └─────────┬──────────┘
             │
             ▼
   ┌────────────────────┐
   │  Amazon QuickSight  │
   │ Dashboards / BI     │
   └────────────────────┘


 Orchestration & Security
 ─────────────────────────────────────────────────────────
 EventBridge → Step Functions → Lambda
                       │
             Secrets Manager / STS
                       │
                 IAM / VPC / VPC
                  Endpoints


## Project Overview

This project demonstrates a modern AWS data pipeline using a layered data architecture:

**Source → Replication → Bronze → Transformation → Silver → Warehouse → BI**

The main objective is to separate operational workloads from analytical workloads while creating a reliable and maintainable path for data ingestion, transformation, storage, and reporting.

### Core Data Flow

1. **Amazon RDS** acts as the operational source database.
2. **AWS DMS** replicates data from RDS.
3. **Amazon S3 Bronze** stores raw/landed source data.
4. **AWS Glue Crawler** discovers schemas and registers metadata.
5. **AWS Glue Data Catalog** provides centralized metadata.
6. **AWS Glue** performs data cleansing and transformation.
7. **Amazon S3 Silver** stores processed and standardized data.
8. A second **AWS Glue** process prepares warehouse-ready datasets.
9. **Amazon Redshift staging tables** receive the transformed data.
10. **Redshift stored procedures** apply warehouse-side business logic and load target tables.
11. **Amazon QuickSight** provides the final analytics and visualization layer.

## AWS Services Used

| Service | Role |
|---|---|
| **Amazon RDS** | Source relational/operational database |
| **AWS Database Migration Service (DMS)** | Database replication and data ingestion |
| **Amazon S3** | Data lake and intermediate storage |
| **AWS Glue Crawler** | Schema discovery and metadata collection |
| **AWS Glue Data Catalog** | Central metadata repository |
| **AWS Glue** | ETL and data transformation |
| **Amazon Redshift** | Analytical data warehouse |
| **Amazon QuickSight** | Business intelligence and dashboards |
| **AWS Step Functions** | Workflow orchestration |
| **AWS Lambda** | Serverless automation and custom workflow logic |
| **Amazon EventBridge** | Event-based or scheduled pipeline triggers |
| **AWS Secrets Manager** | Secure storage of credentials and secrets |
| **AWS STS** | Temporary security credentials |
| **Amazon VPC** | Private network infrastructure |
| **VPC Endpoints** | Private connectivity to supported AWS services |
| **IAM** | Authentication and least-privilege authorization |

## Data Lake Architecture

The S3 portion follows a simplified **medallion architecture**.

### Bronze Layer

The Bronze layer contains data as it arrives from the source system.

```text
RDS
 ↓
DMS
 ↓
S3 Bronze

Typical responsibilities:

Preserve source data

Provide a raw landing area

Support historical retention

Enable reprocessing

Support data lineage and troubleshooting

Silver Layer

The Silver layer contains cleaned and standardized data.

S3 Bronze
    ↓
AWS Glue
    ↓
S3 Silver

Typical transformations include:

Data type standardization

Null handling

Duplicate removal

Data validation

Date/time normalization

Column standardization

Business-rule transformations

Data quality checks

Warehouse Layer

The curated Silver data is prepared for Amazon Redshift.

S3 Silver
    ↓
AWS Glue
    ↓
Redshift Staging
    ↓
Stored Procedure
    ↓
Redshift Target

The target layer can contain analytical fact and dimension tables suitable for reporting and BI workloads.

Workflow Orchestration

AWS Step Functions coordinates the pipeline rather than performing the data transformations itself.

A typical workflow is:

EventBridge
     │
     ▼
Step Functions
     │
     ├── Start / monitor DMS
     │
     ├── Run / monitor Glue
     │
     ├── Load Redshift staging
     │
     ├── Execute warehouse logic
     │
     └── Complete / handle errors

Step Functions can provide:

Ordered execution

Retry handling

Error handling

Conditional branching

Workflow monitoring

Execution history

Service-to-service orchestration

Security Architecture

Security is built into the pipeline through AWS identity and networking services.

Secrets Manager

Database credentials and other sensitive configuration should be stored in AWS Secrets Manager rather than hard-coded in application code.

IAM

AWS services should use IAM roles with only the permissions required to perform their tasks.

STS

AWS STS can provide short-lived temporary credentials instead of relying on long-lived access keys.

VPC

Private resources such as RDS and Redshift can be deployed within controlled VPC networking.

VPC Endpoints

VPC endpoints can provide private connectivity between VPC resources and supported AWS services without requiring public internet access.

Example End-to-End Execution

A scheduled pipeline execution can follow this sequence:

1. EventBridge triggers the pipeline
             ↓
2. Step Functions starts the workflow
             ↓
3. DMS replicates data from RDS
             ↓
4. Raw data lands in S3 Bronze
             ↓
5. Glue Crawler discovers metadata
             ↓
6. Glue Data Catalog is updated
             ↓
7. Glue transforms Bronze data
             ↓
8. Clean data is written to S3 Silver
             ↓
9. Glue prepares warehouse-ready datasets
             ↓
10. Data is loaded into Redshift staging
             ↓
11. Redshift stored procedure processes the data
             ↓
12. Final data is loaded into target tables
             ↓
13. QuickSight queries the warehouse
             ↓
14. Dashboards provide business insights
