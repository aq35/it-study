# Use Case: Data Analysis Pipeline with AWS Glue and Athena

## Overview

This use case demonstrates how to build an end-to-end ETL (Extract, Transform, Load) pipeline for processing and analyzing large datasets using AWS serverless services. You'll learn how to transform raw data into queryable formats and perform analytics without managing infrastructure.

## Problem Statement

Your organization collects logs, transactions, and user activity data from multiple sources in various formats (JSON, CSV, XML). Business analysts need to:

- Query this data using SQL without technical complexity
- Generate reports and dashboards
- Analyze trends and patterns
- Access data quickly without expensive data warehouse infrastructure

Current challenges:
- Raw data is unstructured and difficult to query
- Different file formats complicate analysis
- Manual data preparation is time-consuming
- Analysts lack technical skills to process raw data

You need an automated ETL pipeline that transforms raw data into a queryable format accessible through standard SQL.

## Key Learning Objectives

By completing this use case, you will:

1. **Design ETL Workflows**
   - Understand Extract, Transform, Load principles
   - Plan data transformation logic
   - Handle schema evolution and data quality

2. **Use AWS Glue for Data Processing**
   - Create Glue crawlers to discover schemas
   - Build Glue jobs for data transformation
   - Use the Glue Data Catalog as a metadata repository

3. **Query Data with Amazon Athena**
   - Run SQL queries on S3 data
   - Optimize query performance with partitioning
   - Manage query costs effectively

4. **Implement Data Lake Patterns**
   - Organize data in bronze/silver/gold layers
   - Apply schema-on-read principles
   - Design for scalability and cost-efficiency

5. **Optimize for Performance and Cost**
   - Use columnar formats (Parquet)
   - Implement data partitioning strategies
   - Monitor and control costs

## Expected Outcomes

After completing this use case, you will be able to:
- Build scalable data processing pipelines
- Transform raw data into analytics-ready formats
- Enable SQL-based analytics on large datasets
- Implement cost-effective data lake solutions

## Architecture Overview

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Data Analysis Pipeline                          │
└────────────────────────────────────────────────────────────────────────┘

    External               ┌──────────────────────────────────┐
    Data Sources   ───────▶│   Raw Data Layer (Bronze)       │
    (JSON/CSV)             │   S3: s3://bucket/raw/          │
                           └─────────────┬────────────────────┘
                                         │
                                         │  Discover Schema
                                         ▼
                                  ┌──────────────┐
                                  │  AWS Glue    │
                                  │   Crawler    │
                                  └──────┬───────┘
                                         │
                                         │  Update Schema
                                         ▼
                                  ┌──────────────┐
                                  │  Glue Data   │
                                  │   Catalog    │──────┐
                                  └──────┬───────┘      │
                                         │              │
                                         │ Schema       │ Schema
                                         │ Info         │ Info
                                         ▼              │
                           ┌─────────────────────────┐  │
                           │     AWS Glue ETL Job    │  │
                           │  (Transform & Clean)    │  │
                           └──────────┬──────────────┘  │
                                      │                 │
                                      │ Write           │
                                      ▼                 │
                           ┌──────────────────────────┐ │
                           │  Processed Data Layer    │ │
                           │  (Silver/Gold)           │ │
                           │  S3: s3://bucket/        │ │
                           │      processed/          │ │
                           │  Format: Parquet         │ │
                           │  Partitioned by date     │ │
                           └──────────┬───────────────┘ │
                                      │                 │
                                      │ Crawl           │
                                      ▼                 │
                                  ┌──────────────┐      │
                                  │  AWS Glue    │      │
                                  │   Crawler    │      │
                                  └──────┬───────┘      │
                                         │              │
                                         └──────────────┘
                                                │
                                                │ Query
                                                ▼
                           ┌─────────────────────────────┐
                           │     Amazon Athena           │
                           │  (SQL Query Engine)         │
                           └──────────┬──────────────────┘
                                      │
                                      │ Results
                                      ▼
                           ┌─────────────────────────────┐
                           │  Query Results              │
                           │  S3: s3://bucket/results/   │
                           └─────────────────────────────┘
                                      │
                                      │ Visualize
                                      ▼
                           ┌─────────────────────────────┐
                           │  QuickSight / BI Tools      │
                           │  (Optional)                 │
                           └─────────────────────────────┘
```

## Component Descriptions

### 1. S3 Data Lake (Storage Layers)
- **Raw Layer (Bronze)**: Stores original, unprocessed data
- **Processed Layer (Silver)**: Clean, validated, partitioned data
- **Analytics Layer (Gold)**: Aggregated, business-ready datasets

### 2. AWS Glue Crawler
- **Purpose**: Automatically discovers schemas and creates table definitions
- **Output**: Populates Glue Data Catalog with metadata
- **Scheduling**: Runs on-demand or scheduled basis

### 3. AWS Glue Data Catalog
- **Purpose**: Central metadata repository
- **Benefits**: Schema versioning, data discovery, integration with analytics tools
- **Compatibility**: Works with Athena, Redshift Spectrum, EMR

### 4. AWS Glue ETL Job
- **Purpose**: Transforms and processes data
- **Language**: PySpark or Scala
- **Features**: Built-in transformations, schema evolution, data quality

### 5. Amazon Athena
- **Purpose**: Serverless SQL query engine
- **Benefits**: No infrastructure, pay-per-query, standard SQL
- **Integration**: Queries Glue Data Catalog tables

## Step-by-Step Implementation

### Prerequisites

Before starting, ensure you have:
- [ ] AWS account with appropriate permissions
- [ ] AWS CLI installed and configured
- [ ] Basic understanding of SQL
- [ ] Familiarity with Python (for Glue jobs)

### Step 1: Set Up S3 Data Lake Structure

Create S3 buckets with organized folder structure:

```bash
# Set bucket name (must be globally unique)
DATA_LAKE_BUCKET="data-lake-analytics-$(date +%s)"
ATHENA_RESULTS_BUCKET="athena-query-results-$(date +%s)"

# Create buckets
aws s3 mb s3://${DATA_LAKE_BUCKET} --region us-east-1
aws s3 mb s3://${ATHENA_RESULTS_BUCKET} --region us-east-1

# Create folder structure
aws s3api put-object --bucket ${DATA_LAKE_BUCKET} --key raw/
aws s3api put-object --bucket ${DATA_LAKE_BUCKET} --key processed/
aws s3api put-object --bucket ${DATA_LAKE_BUCKET} --key scripts/

echo "Data lake structure created: s3://${DATA_LAKE_BUCKET}"
```

**Learning Point**: Organizing data in layers (raw, processed, analytics) follows data lake best practices and simplifies data governance.

### Step 2: Generate Sample Data

Create sample data files to simulate real-world scenarios:

```bash
# Create sample sales transaction data
cat > generate_sample_data.py <<'EOF'
import json
import csv
import random
from datetime import datetime, timedelta

def generate_transactions(num_records=1000):
    """Generate sample sales transaction data"""
    transactions = []
    start_date = datetime(2024, 1, 1)
    
    products = [
        {"id": "P001", "name": "Laptop", "category": "Electronics", "base_price": 999},
        {"id": "P002", "name": "Mouse", "category": "Electronics", "base_price": 25},
        {"id": "P003", "name": "Keyboard", "category": "Electronics", "base_price": 75},
        {"id": "P004", "name": "Monitor", "category": "Electronics", "base_price": 299},
        {"id": "P005", "name": "Desk Chair", "category": "Furniture", "base_price": 199},
    ]
    
    for i in range(num_records):
        product = random.choice(products)
        transaction_date = start_date + timedelta(days=random.randint(0, 365))
        
        transaction = {
            "transaction_id": f"TXN{i+1:06d}",
            "timestamp": transaction_date.isoformat(),
            "product_id": product["id"],
            "product_name": product["name"],
            "category": product["category"],
            "quantity": random.randint(1, 5),
            "unit_price": product["base_price"],
            "total_amount": product["base_price"] * random.randint(1, 5),
            "customer_id": f"CUST{random.randint(1, 100):04d}",
            "region": random.choice(["North", "South", "East", "West"]),
            "payment_method": random.choice(["Credit Card", "Debit Card", "PayPal", "Cash"])
        }
        transactions.append(transaction)
    
    return transactions

# Generate and save JSON data
transactions = generate_transactions(1000)

# Save as JSON Lines format (one JSON object per line)
with open('transactions.json', 'w') as f:
    for trans in transactions:
        f.write(json.dumps(trans) + '\n')

print(f"Generated {len(transactions)} transactions")
print("Files created: transactions.json")
EOF

# Run the data generation script
python3 generate_sample_data.py

# Upload to S3 raw layer with date partition
YEAR=$(date +%Y)
MONTH=$(date +%m)
DAY=$(date +%d)

aws s3 cp transactions.json \
  s3://${DATA_LAKE_BUCKET}/raw/transactions/year=${YEAR}/month=${MONTH}/day=${DAY}/

echo "Sample data uploaded to S3"
```

**Learning Point**: Organizing data with partitions (year/month/day) significantly improves query performance and reduces costs.

### Step 3: Create IAM Role for Glue

Create an IAM role with necessary permissions:

```bash
# Create trust policy for Glue
cat > glue-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "glue.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create the role
aws iam create-role \
  --role-name AWSGlueServiceRole-DataPipeline \
  --assume-role-policy-document file://glue-trust-policy.json

# Attach AWS managed policy for Glue
aws iam attach-role-policy \
  --role-name AWSGlueServiceRole-DataPipeline \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole

# Create custom policy for S3 access
cat > glue-s3-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::${DATA_LAKE_BUCKET}/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::${DATA_LAKE_BUCKET}"
      ]
    }
  ]
}
EOF

# Attach S3 policy
aws iam put-role-policy \
  --role-name AWSGlueServiceRole-DataPipeline \
  --policy-name GlueS3Access \
  --policy-document file://glue-s3-policy.json

# Get role ARN
GLUE_ROLE_ARN=$(aws iam get-role \
  --role-name AWSGlueServiceRole-DataPipeline \
  --query 'Role.Arn' \
  --output text)

echo "Glue Role ARN: ${GLUE_ROLE_ARN}"
```

**Learning Point**: Glue needs permissions to read source data, write processed data, and access the Data Catalog.

### Step 4: Create Glue Database

Set up a database in the Glue Data Catalog:

```bash
# Create database
aws glue create-database \
  --database-input '{
    "Name": "sales_analytics",
    "Description": "Database for sales transaction analytics"
  }'

echo "Glue database 'sales_analytics' created"
```

**Learning Point**: The Glue database is a logical container for tables, similar to a database in traditional RDBMS.

### Step 5: Create and Run Glue Crawler for Raw Data

Create a crawler to discover the schema of raw data:

```bash
# Create crawler for raw data
aws glue create-crawler \
  --name raw-transactions-crawler \
  --role ${GLUE_ROLE_ARN} \
  --database-name sales_analytics \
  --targets "{
    \"S3Targets\": [
      {
        \"Path\": \"s3://${DATA_LAKE_BUCKET}/raw/transactions/\"
      }
    ]
  }" \
  --schema-change-policy '{
    "UpdateBehavior": "UPDATE_IN_DATABASE",
    "DeleteBehavior": "LOG"
  }' \
  --configuration '{
    "Version": 1.0,
    "CrawlerOutput": {
      "Partitions": {"AddOrUpdateBehavior": "InheritFromTable"}
    }
  }'

echo "Crawler created. Starting crawl..."

# Start the crawler
aws glue start-crawler --name raw-transactions-crawler

# Wait for crawler to complete
echo "Waiting for crawler to complete (this may take a minute)..."
sleep 60

# Check crawler status
aws glue get-crawler --name raw-transactions-crawler \
  --query 'Crawler.State' \
  --output text

# View discovered tables
aws glue get-tables \
  --database-name sales_analytics \
  --query 'TableList[*].[Name, DatabaseName]' \
  --output table
```

**Learning Points**:
- Crawlers automatically infer schema from data
- Schema change policy determines how updates are handled
- Partitions are automatically detected from folder structure

### Step 6: Create Glue ETL Job

Create a Glue ETL job to transform raw data into optimized Parquet format:

```python
# Save as glue_etl_job.py
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql.functions import col, year, month, dayofmonth, to_timestamp

# Get job parameters
args = getResolvedOptions(sys.argv, ['JOB_NAME', 'SOURCE_DATABASE', 'SOURCE_TABLE', 'TARGET_BUCKET'])

# Initialize Glue context
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read data from Glue Data Catalog
datasource = glueContext.create_dynamic_frame.from_catalog(
    database=args['SOURCE_DATABASE'],
    table_name=args['SOURCE_TABLE'],
    transformation_ctx="datasource"
)

print(f"Source record count: {datasource.count()}")

# Convert to Spark DataFrame for transformations
df = datasource.toDF()

# Data transformations
# 1. Convert timestamp string to timestamp type
df = df.withColumn("transaction_timestamp", to_timestamp(col("timestamp")))

# 2. Add derived columns
df = df.withColumn("year", year(col("transaction_timestamp")))
df = df.withColumn("month", month(col("transaction_timestamp")))
df = df.withColumn("day", dayofmonth(col("transaction_timestamp")))

# 3. Calculate total with validation
df = df.withColumn("calculated_total", col("quantity") * col("unit_price"))

# 4. Data quality checks
# Filter out records with negative quantities or amounts
df_clean = df.filter(
    (col("quantity") > 0) & 
    (col("total_amount") > 0) &
    (col("unit_price") > 0)
)

print(f"Clean record count: {df_clean.count()}")

# Convert back to DynamicFrame
dynamic_frame_clean = DynamicFrame.fromDF(df_clean, glueContext, "dynamic_frame_clean")

# Write to S3 in Parquet format with partitioning
output_path = f"s3://{args['TARGET_BUCKET']}/processed/transactions/"

glueContext.write_dynamic_frame.from_options(
    frame=dynamic_frame_clean,
    connection_type="s3",
    connection_options={
        "path": output_path,
        "partitionKeys": ["year", "month", "day"]
    },
    format="parquet",
    transformation_ctx="datasink"
)

print(f"Data written to: {output_path}")

job.commit()
```

Upload the script and create the Glue job:

```bash
# Upload ETL script to S3
aws s3 cp glue_etl_job.py s3://${DATA_LAKE_BUCKET}/scripts/

# Create Glue ETL job
aws glue create-job \
  --name sales-etl-job \
  --role ${GLUE_ROLE_ARN} \
  --command '{
    "Name": "glueetl",
    "ScriptLocation": "s3://'${DATA_LAKE_BUCKET}'/scripts/glue_etl_job.py",
    "PythonVersion": "3"
  }' \
  --default-arguments '{
    "--SOURCE_DATABASE": "sales_analytics",
    "--SOURCE_TABLE": "transactions",
    "--TARGET_BUCKET": "'${DATA_LAKE_BUCKET}'",
    "--enable-metrics": "true",
    "--enable-continuous-cloudwatch-log": "true"
  }' \
  --max-retries 1 \
  --timeout 60 \
  --glue-version "3.0" \
  --number-of-workers 2 \
  --worker-type "G.1X"

echo "Glue ETL job created"
```

**Learning Points**:
- Glue uses Apache Spark for distributed data processing
- PySpark provides powerful data transformation capabilities
- Parquet format significantly reduces storage costs and improves query performance
- Partitioning by date enables partition pruning for faster queries

### Step 7: Run the ETL Job

Execute the ETL job:

```bash
# Start job run
JOB_RUN_ID=$(aws glue start-job-run \
  --job-name sales-etl-job \
  --query 'JobRunId' \
  --output text)

echo "Job run started: ${JOB_RUN_ID}"
echo "Waiting for job to complete (this may take a few minutes)..."

# Monitor job status
while true; do
  STATUS=$(aws glue get-job-run \
    --job-name sales-etl-job \
    --run-id ${JOB_RUN_ID} \
    --query 'JobRun.JobRunState' \
    --output text)
  
  echo "Job status: ${STATUS}"
  
  if [ "$STATUS" == "SUCCEEDED" ] || [ "$STATUS" == "FAILED" ] || [ "$STATUS" == "STOPPED" ]; then
    break
  fi
  
  sleep 30
done

# Get job metrics
aws glue get-job-run \
  --job-name sales-etl-job \
  --run-id ${JOB_RUN_ID} \
  --query 'JobRun.[ExecutionTime, DPUSeconds]' \
  --output table
```

**Learning Point**: Monitor job execution to understand resource usage and optimize for cost.

### Step 8: Crawler for Processed Data

Create another crawler for the processed data:

```bash
# Create crawler for processed data
aws glue create-crawler \
  --name processed-transactions-crawler \
  --role ${GLUE_ROLE_ARN} \
  --database-name sales_analytics \
  --targets "{
    \"S3Targets\": [
      {
        \"Path\": \"s3://${DATA_LAKE_BUCKET}/processed/transactions/\"
      }
    ]
  }" \
  --schema-change-policy '{
    "UpdateBehavior": "UPDATE_IN_DATABASE",
    "DeleteBehavior": "LOG"
  }' \
  --configuration '{
    "Version": 1.0,
    "CrawlerOutput": {
      "Partitions": {"AddOrUpdateBehavior": "InheritFromTable"}
    }
  }'

# Run the crawler
aws glue start-crawler --name processed-transactions-crawler

echo "Waiting for crawler to complete..."
sleep 60

# Check results
aws glue get-tables \
  --database-name sales_analytics \
  --query 'TableList[*].[Name, StorageDescriptor.Location]' \
  --output table
```

### Step 9: Configure Amazon Athena

Set up Athena for querying:

```bash
# Create Athena workgroup with query results location
aws athena create-work-group \
  --name analytics-workgroup \
  --configuration "ResultConfigurationUpdates={
    OutputLocation=s3://${ATHENA_RESULTS_BUCKET}/
  },EnforceWorkGroupConfiguration=true" \
  --description "Workgroup for sales analytics queries"

echo "Athena workgroup created"
```

**Learning Point**: Workgroups help organize queries, control costs, and enforce output locations.

### Step 10: Query Data with Athena

Run SQL queries on your processed data:

```bash
# Function to run Athena query
run_athena_query() {
  local query=$1
  local query_name=$2
  
  echo "Running query: ${query_name}"
  
  # Start query execution
  QUERY_ID=$(aws athena start-query-execution \
    --query-string "${query}" \
    --work-group analytics-workgroup \
    --query-execution-context Database=sales_analytics \
    --query 'QueryExecutionId' \
    --output text)
  
  # Wait for query to complete
  while true; do
    STATUS=$(aws athena get-query-execution \
      --query-execution-id ${QUERY_ID} \
      --query 'QueryExecution.Status.State' \
      --output text)
    
    if [ "$STATUS" == "SUCCEEDED" ] || [ "$STATUS" == "FAILED" ]; then
      break
    fi
    sleep 2
  done
  
  if [ "$STATUS" == "SUCCEEDED" ]; then
    # Get results
    aws athena get-query-results \
      --query-execution-id ${QUERY_ID} \
      --query 'ResultSet.Rows[*]' \
      --output table
  else
    echo "Query failed"
    aws athena get-query-execution \
      --query-execution-id ${QUERY_ID} \
      --query 'QueryExecution.Status.StateChangeReason'
  fi
}

# Example queries

# 1. Total sales by category
run_athena_query "
SELECT 
  category,
  COUNT(*) as transaction_count,
  SUM(total_amount) as total_sales,
  AVG(total_amount) as avg_transaction_value
FROM transactions
GROUP BY category
ORDER BY total_sales DESC
" "Sales by Category"

# 2. Daily sales trend
run_athena_query "
SELECT 
  year,
  month,
  day,
  COUNT(*) as transactions,
  SUM(total_amount) as daily_revenue
FROM transactions
WHERE year = 2024
GROUP BY year, month, day
ORDER BY year, month, day
LIMIT 10
" "Daily Sales Trend"

# 3. Top products
run_athena_query "
SELECT 
  product_name,
  COUNT(*) as units_sold,
  SUM(total_amount) as revenue
FROM transactions
GROUP BY product_name
ORDER BY revenue DESC
LIMIT 5
" "Top Products"

# 4. Sales by region
run_athena_query "
SELECT 
  region,
  COUNT(DISTINCT customer_id) as unique_customers,
  COUNT(*) as transactions,
  SUM(total_amount) as total_revenue
FROM transactions
GROUP BY region
ORDER BY total_revenue DESC
" "Regional Analysis"
```

**Learning Points**:
- Athena uses standard SQL syntax
- Queries are billed based on data scanned
- Partitioning and columnar formats (Parquet) reduce scan costs
- Results are stored in S3 for later retrieval

### Step 11: Optimize Query Performance

Implement optimization strategies:

```bash
# Create a view for frequently used aggregations
run_athena_query "
CREATE OR REPLACE VIEW daily_sales_summary AS
SELECT 
  year,
  month,
  day,
  category,
  COUNT(*) as transaction_count,
  SUM(total_amount) as total_sales,
  AVG(total_amount) as avg_sale
FROM transactions
GROUP BY year, month, day, category
" "Create Daily Summary View"

# Example: Query specific partition to reduce scan
run_athena_query "
SELECT 
  product_name,
  SUM(total_amount) as revenue
FROM transactions
WHERE year = 2024 AND month = 1
GROUP BY product_name
ORDER BY revenue DESC
" "Partitioned Query Example"
```

**Cost Optimization Tips**:
1. Always filter on partition columns (year, month, day)
2. Use columnar formats (Parquet) instead of JSON/CSV
3. Compress data (Parquet supports automatic compression)
4. Limit SELECT to only needed columns
5. Use views to pre-aggregate common queries

### Step 12: Set Up Monitoring

Monitor ETL pipeline and query costs:

```bash
# Create CloudWatch alarm for Glue job failures
aws cloudwatch put-metric-alarm \
  --alarm-name GlueJobFailures \
  --alarm-description "Alert on Glue job failures" \
  --namespace Glue \
  --metric-name glue.driver.aggregate.numFailedTasks \
  --dimensions Name=JobName,Value=sales-etl-job \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanThreshold

echo "CloudWatch alarm created for Glue monitoring"

# Check Athena query metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/Athena \
  --metric-name DataScannedInBytes \
  --dimensions Name=WorkGroup,Value=analytics-workgroup \
  --start-time $(date -u -d '1 day ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 3600 \
  --statistics Sum \
  --output table
```

## Advanced Enhancements

### 1. Incremental Processing

Process only new data instead of full reprocessing:

```python
# Add bookmark support to Glue job
job.init(args['JOB_NAME'], args)

# Enable job bookmarks
datasource = glueContext.create_dynamic_frame.from_catalog(
    database=args['SOURCE_DATABASE'],
    table_name=args['SOURCE_TABLE'],
    transformation_ctx="datasource",
    additional_options={"jobBookmarkKeys": ["transaction_id"]}
)
```

### 2. Data Quality Checks

Implement automated data quality validation:

```python
# Add data quality rules
from awsgluedq.transforms import EvaluateDataQuality

quality_rules = """
  Rules = [
    RowCount > 0,
    ColumnValues "total_amount" > 0,
    ColumnDataType "timestamp" = "STRING",
    Completeness "transaction_id" = 1.0
  ]
"""

quality_check = EvaluateDataQuality().process_rows(
    frame=datasource,
    ruleset=quality_rules
)
```

### 3. Schedule ETL Pipeline

Automate the pipeline with Glue Triggers:

```bash
# Create daily schedule trigger
aws glue create-trigger \
  --name daily-etl-trigger \
  --type SCHEDULED \
  --schedule "cron(0 2 * * ? *)" \
  --actions JobName=sales-etl-job \
  --start-on-creation
```

### 4. Create Aggregate Tables

Build pre-aggregated tables for common queries:

```sql
-- Create aggregate table for better performance
CREATE TABLE sales_analytics.monthly_summary
WITH (
  format = 'PARQUET',
  partitioned_by = ARRAY['year', 'month']
) AS
SELECT 
  category,
  region,
  year,
  month,
  COUNT(*) as transactions,
  SUM(total_amount) as total_revenue,
  AVG(total_amount) as avg_transaction
FROM sales_analytics.transactions
GROUP BY category, region, year, month
```

## Cost Optimization Strategies

### Storage Optimization
1. **Use Parquet**: 80-90% compression vs JSON
2. **Implement Lifecycle Policies**: Move old data to Glacier
3. **Delete Intermediate Results**: Clean up Athena query results regularly

### Query Optimization
1. **Partition Pruning**: Always filter on partition columns
2. **Column Selection**: Select only needed columns
3. **Data Compression**: Use SNAPPY or GZIP compression
4. **Result Reuse**: Cache common query results

### Glue Job Optimization
1. **Right-size Workers**: Use G.1X for small jobs, G.2X for large
2. **Enable Job Bookmarks**: Process only new data
3. **Batch Processing**: Run jobs during off-peak hours
4. **Monitor DPU Consumption**: Track and optimize resource usage

## Common Troubleshooting

### Issue: Crawler Not Finding Partitions
- **Solution**: Ensure partition format follows Hive convention (key=value)
- **Check**: S3 folder structure matches partition keys

### Issue: Athena Query Timing Out
- **Solution**: Reduce data scanned by filtering on partitions
- **Check**: Use EXPLAIN to understand query plan
- **Fix**: Break large queries into smaller chunks

### Issue: Glue Job Failing with OOM
- **Solution**: Increase worker memory (G.2X instead of G.1X)
- **Check**: Reduce number of partitions written simultaneously
- **Fix**: Use repartition() to control output parallelism

### Issue: High Athena Costs
- **Solution**: Convert to Parquet and partition data
- **Check**: Analyze queries in Cost Explorer
- **Fix**: Implement views for common aggregations

## Cleanup

Remove all resources to avoid charges:

```bash
# Delete Glue resources
aws glue delete-job --job-name sales-etl-job
aws glue delete-crawler --name raw-transactions-crawler
aws glue delete-crawler --name processed-transactions-crawler
aws glue delete-database --name sales_analytics

# Delete Athena workgroup
aws athena delete-work-group --work-group analytics-workgroup --recursive-delete-option

# Empty and delete S3 buckets
aws s3 rm s3://${DATA_LAKE_BUCKET} --recursive
aws s3 rb s3://${DATA_LAKE_BUCKET}
aws s3 rm s3://${ATHENA_RESULTS_BUCKET} --recursive
aws s3 rb s3://${ATHENA_RESULTS_BUCKET}

# Delete IAM role
aws iam delete-role-policy \
  --role-name AWSGlueServiceRole-DataPipeline \
  --policy-name GlueS3Access
aws iam detach-role-policy \
  --role-name AWSGlueServiceRole-DataPipeline \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSGlueServiceRole
aws iam delete-role --role-name AWSGlueServiceRole-DataPipeline

# Delete CloudWatch alarms
aws cloudwatch delete-alarms --alarm-names GlueJobFailures

# Clean up local files
rm -f generate_sample_data.py transactions.json glue_etl_job.py \
      glue-trust-policy.json glue-s3-policy.json
```

## Key Takeaways

✅ **ETL Pipelines** transform raw data into analytics-ready formats
✅ **AWS Glue** provides serverless data processing at scale
✅ **Parquet format** drastically reduces storage and query costs
✅ **Partitioning** improves query performance and reduces costs
✅ **Athena** enables SQL queries without infrastructure
✅ **Data Catalog** provides centralized metadata management

## Next Steps

- Explore AWS Lake Formation for data governance
- Integrate with QuickSight for visualization
- Implement incremental processing with bookmarks
- Study advanced Spark transformations
- Learn about data quality frameworks

## Additional Resources

- [AWS Glue Documentation](https://docs.aws.amazon.com/glue/)
- [Amazon Athena User Guide](https://docs.aws.amazon.com/athena/)
- [Building Data Lakes on AWS](https://aws.amazon.com/big-data/datalakes-and-analytics/)
- [Glue ETL Programming Guide](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming.html)
