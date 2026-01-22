# Use Case: Cost Management and Optimization on AWS

## Overview

This use case demonstrates how to effectively monitor, analyze, and optimize AWS costs. You'll learn to implement cost management strategies, set up budgets and alerts, and apply cost optimization techniques while maintaining performance and reliability.

## Problem Statement

Your organization is using AWS for various workloads, but cloud costs are growing unpredictably. Management needs:

- **Cost Visibility**: Understanding where money is being spent
- **Budget Control**: Preventing cost overruns
- **Optimization Opportunities**: Identifying ways to reduce costs without impacting performance
- **Forecasting**: Predicting future costs based on current trends
- **Accountability**: Attributing costs to specific teams, projects, or applications

Current challenges:
- Unexpected monthly bills
- Difficulty tracking cost drivers
- No alerts for cost anomalies
- Underutilized resources consuming budget
- Lack of cost optimization culture

You need a comprehensive cost management strategy that provides visibility, control, and continuous optimization.

## Key Learning Objectives

By completing this use case, you will:

1. **Set Up Cost Monitoring**
   - Configure AWS Cost Explorer for analysis
   - Create custom cost reports
   - Understand cost allocation tags

2. **Implement Budget Controls**
   - Create budgets with threshold alerts
   - Set up anomaly detection
   - Configure automated responses to cost events

3. **Use AWS Trusted Advisor**
   - Identify cost optimization opportunities
   - Review and implement recommendations
   - Monitor cost-related security risks

4. **Optimize Compute Costs**
   - Right-size instances based on utilization
   - Leverage Reserved Instances and Savings Plans
   - Use Spot Instances for flexible workloads

5. **Optimize Storage and Data Transfer**
   - Implement S3 lifecycle policies
   - Reduce data transfer costs
   - Clean up unused resources

6. **Enable Cost Governance**
   - Implement tagging strategies
   - Use Service Control Policies (SCPs)
   - Create cost-aware culture

## Expected Outcomes

After completing this use case, you will be able to:
- Monitor and forecast AWS costs effectively
- Implement proactive cost controls
- Identify and realize cost savings opportunities
- Build cost-conscious cloud architectures
- Create accountability through cost allocation

## Architecture Overview

```
┌────────────────────────────────────────────────────────────────────────┐
│                    Cost Management Architecture                         │
└────────────────────────────────────────────────────────────────────────┘

                         AWS Account Resources
    ┌──────────────────────────────────────────────────────────┐
    │  EC2 │ RDS │ S3 │ Lambda │ CloudFront │ ... │ ETC       │
    └────────┬─────────────────────────────────────────────────┘
             │
             │ Generate Cost & Usage Data
             ▼
    ┌─────────────────────────────────────┐
    │   Cost and Usage Reports (CUR)       │
    │   Detailed billing data              │
    └─────────────┬───────────────────────┘
                  │
                  │ Store Reports
                  ▼
         ┌──────────────────┐
         │   S3 Bucket      │
         │   (CUR Storage)  │
         └─────────┬────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
     ▼             ▼             ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│  Cost   │  │   AWS    │  │   Athena /   │
│Explorer │  │ Budgets  │  │  QuickSight  │
│         │  │          │  │  (Analytics) │
└────┬────┘  └────┬─────┘  └──────────────┘
     │            │
     │ Analyze    │ Alert on Threshold
     │            ▼
     │      ┌──────────────┐
     │      │     SNS      │
     │      │    Topic     │
     │      └──────┬───────┘
     │             │
     │             │ Notifications
     │             ▼
     │      ┌──────────────┐
     │      │   Email /    │
     │      │   Lambda /   │
     │      │   Slack      │
     │      └──────────────┘
     │
     │ Recommendations
     ▼
┌─────────────────────┐      ┌──────────────────────┐
│  Trusted Advisor    │      │  Compute Optimizer   │
│  (Cost Checks)      │      │  (Rightsizing)       │
└──────────┬──────────┘      └──────────┬───────────┘
           │                            │
           │ Provide Recommendations    │
           └───────────┬────────────────┘
                       │
                       │ Implement
                       ▼
           ┌────────────────────────┐
           │  Cost Optimization     │
           │  Actions:              │
           │  - Right-sizing        │
           │  - Reserved Instances  │
           │  - Resource cleanup    │
           │  - Lifecycle policies  │
           └────────────────────────┘

                 Tagging Strategy
    ┌──────────────────────────────────────────────┐
    │  All Resources Tagged:                        │
    │  - Project                                    │
    │  - Environment (Dev/Staging/Prod)            │
    │  - Owner                                      │
    │  - Cost Center                                │
    └──────────────────────────────────────────────┘
```

## Component Descriptions

### Monitoring & Analysis
- **Cost Explorer**: Interactive cost analysis and visualization
- **Cost and Usage Reports (CUR)**: Detailed billing data
- **AWS Budgets**: Set spending thresholds and alerts
- **Cost Anomaly Detection**: ML-based anomaly identification

### Optimization Tools
- **Trusted Advisor**: Best practice recommendations
- **Compute Optimizer**: ML-powered rightsizing recommendations
- **Cost Optimization Hub**: Centralized savings opportunities

### Governance & Control
- **Tagging**: Resource categorization for cost allocation
- **Service Control Policies**: Preventive cost controls
- **Resource Groups**: Organize and manage resources by tags

## Step-by-Step Implementation

### Prerequisites

Before starting, ensure you have:
- [ ] AWS account with billing access
- [ ] Admin or billing administrator permissions
- [ ] AWS CLI installed and configured
- [ ] Basic understanding of AWS services and pricing

### Step 1: Enable Cost Allocation Tags

Set up tagging strategy for cost attribution:

```bash
# Activate cost allocation tags via AWS CLI
# First, define your tagging strategy
cat > tagging-strategy.txt <<EOF
Required Tags for All Resources:
1. Project - Which project or application
2. Environment - dev, staging, prod
3. Owner - Email of responsible person/team
4. CostCenter - Department or cost center code
EOF

# Activate user-defined cost allocation tags
aws ce update-cost-allocation-tags-status \
  --cost-allocation-tags-status \
    TagKey=Project,Status=Active \
    TagKey=Environment,Status=Active \
    TagKey=Owner,Status=Active \
    TagKey=CostCenter,Status=Active

echo "Cost allocation tags activated (may take 24 hours to appear in Cost Explorer)"

# Example: Tag existing resources
# Tag an EC2 instance
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags \
    Key=Project,Value=WebApp \
    Key=Environment,Value=Production \
    Key=Owner,Value=devops@example.com \
    Key=CostCenter,Value=Engineering

# Tag an S3 bucket
aws s3api put-bucket-tagging \
  --bucket my-application-bucket \
  --tagging 'TagSet=[
    {Key=Project,Value=WebApp},
    {Key=Environment,Value=Production},
    {Key=Owner,Value=storage-team@example.com},
    {Key=CostCenter,Value=Engineering}
  ]'

# Tag RDS instance
aws rds add-tags-to-resource \
  --resource-name arn:aws:rds:us-east-1:123456789012:db:my-database \
  --tags \
    Key=Project,Value=WebApp \
    Key=Environment,Value=Production \
    Key=Owner,Value=dba@example.com \
    Key=CostCenter,Value=Engineering
```

**Learning Points**:
- Tags enable cost allocation and filtering
- Tags must be activated in billing before appearing in reports
- Consistent tagging is critical for cost visibility
- Tag policies can enforce tagging standards

### Step 2: Set Up AWS Budgets

Create budgets with alerts to prevent overspending:

```bash
# Create SNS topic for budget alerts
BUDGET_TOPIC_ARN=$(aws sns create-topic \
  --name cost-budget-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe email to budget alerts
aws sns subscribe \
  --topic-arn ${BUDGET_TOPIC_ARN} \
  --protocol email \
  --notification-endpoint finance@example.com

echo "Please confirm SNS subscription via email"
echo "Topic ARN: ${BUDGET_TOPIC_ARN}"

# Get AWS Account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create monthly budget with alerts
cat > budget-config.json <<EOF
{
  "BudgetName": "MonthlyTotalCostBudget",
  "BudgetType": "COST",
  "TimeUnit": "MONTHLY",
  "BudgetLimit": {
    "Amount": "1000",
    "Unit": "USD"
  },
  "CostFilters": {},
  "CostTypes": {
    "IncludeTax": true,
    "IncludeSubscription": true,
    "UseBlended": false,
    "IncludeRefund": false,
    "IncludeCredit": false,
    "IncludeUpfront": true,
    "IncludeRecurring": true,
    "IncludeOtherSubscription": true,
    "IncludeSupport": true,
    "IncludeDiscount": true,
    "UseAmortized": false
  }
}
EOF

# Create budget notification configuration
cat > budget-notifications.json <<EOF
[
  {
    "Notification": {
      "NotificationType": "ACTUAL",
      "ComparisonOperator": "GREATER_THAN",
      "Threshold": 80,
      "ThresholdType": "PERCENTAGE"
    },
    "Subscribers": [
      {
        "SubscriptionType": "SNS",
        "Address": "${BUDGET_TOPIC_ARN}"
      }
    ]
  },
  {
    "Notification": {
      "NotificationType": "ACTUAL",
      "ComparisonOperator": "GREATER_THAN",
      "Threshold": 100,
      "ThresholdType": "PERCENTAGE"
    },
    "Subscribers": [
      {
        "SubscriptionType": "SNS",
        "Address": "${BUDGET_TOPIC_ARN}"
      }
    ]
  },
  {
    "Notification": {
      "NotificationType": "FORECASTED",
      "ComparisonOperator": "GREATER_THAN",
      "Threshold": 100,
      "ThresholdType": "PERCENTAGE"
    },
    "Subscribers": [
      {
        "SubscriptionType": "SNS",
        "Address": "${BUDGET_TOPIC_ARN}"
      }
    ]
  }
]
EOF

# Create the budget
aws budgets create-budget \
  --account-id ${ACCOUNT_ID} \
  --budget file://budget-config.json \
  --notifications-with-subscribers file://budget-notifications.json

echo "Budget created with 80%, 100% actual and 100% forecasted alerts"

# Create project-specific budget
cat > project-budget.json <<EOF
{
  "BudgetName": "WebAppProjectBudget",
  "BudgetType": "COST",
  "TimeUnit": "MONTHLY",
  "BudgetLimit": {
    "Amount": "500",
    "Unit": "USD"
  },
  "CostFilters": {
    "TagKeyValue": ["user:Project\$WebApp"]
  },
  "CostTypes": {
    "IncludeTax": true,
    "IncludeSubscription": true,
    "UseBlended": false
  }
}
EOF

aws budgets create-budget \
  --account-id ${ACCOUNT_ID} \
  --budget file://project-budget.json \
  --notifications-with-subscribers file://budget-notifications.json

echo "Project-specific budget created"
```

**Budget Best Practices**:
- Create budgets at multiple levels (account, project, environment)
- Set multiple thresholds (e.g., 80%, 90%, 100%)
- Use forecasted alerts to prevent overruns
- Review and adjust budgets quarterly

### Step 3: Enable Cost and Usage Reports (CUR)

Set up detailed billing reports:

```bash
# Create S3 bucket for Cost and Usage Reports
CUR_BUCKET="aws-cur-reports-$(date +%s)"
aws s3 mb s3://${CUR_BUCKET} --region us-east-1

# Create bucket policy for CUR
cat > cur-bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "billingreports.amazonaws.com"
      },
      "Action": [
        "s3:GetBucketAcl",
        "s3:GetBucketPolicy"
      ],
      "Resource": "arn:aws:s3:::${CUR_BUCKET}"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "billingreports.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::${CUR_BUCKET}/*"
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket ${CUR_BUCKET} \
  --policy file://cur-bucket-policy.json

# Note: Creating CUR via CLI requires additional setup
# Easier to do via AWS Console: Billing Dashboard > Cost & Usage Reports
echo "S3 bucket created for CUR: s3://${CUR_BUCKET}"
echo "Please complete CUR setup in AWS Console at:"
echo "https://console.aws.amazon.com/billing/home#/reports"
```

**CUR Setup in Console**:
1. Navigate to Billing Dashboard → Cost & Usage Reports
2. Click "Create report"
3. Configure:
   - Report name: "detailed-cost-usage-report"
   - Include resource IDs: ✓
   - Data refresh settings: Automatically refresh
   - Time granularity: Hourly
   - Report versioning: Overwrite existing report
   - Compression: GZIP
   - Format: Parquet (for better query performance)
   - S3 bucket: Select the bucket created above

### Step 4: Analyze Costs with Cost Explorer

Use Cost Explorer API to analyze spending:

```bash
# Get cost and usage for last 30 days
START_DATE=$(date -u -d '30 days ago' +%Y-%m-%d)
END_DATE=$(date -u +%Y-%m-%d)

# Total costs by service
aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity DAILY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[*].[TimePeriod.Start, Groups[*].[Keys[0], Metrics.UnblendedCost.Amount]]' \
  --output table

# Costs by linked account (for Organizations)
aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=LINKED_ACCOUNT \
  --output table

# Costs filtered by tag (e.g., by Project)
aws ce get-cost-and-usage \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=TAG,Key=Project \
  --output table

# Get cost forecast for next 30 days
FORECAST_START=$(date -u +%Y-%m-%d)
FORECAST_END=$(date -u -d '30 days' +%Y-%m-%d)

aws ce get-cost-forecast \
  --time-period Start=${FORECAST_START},End=${FORECAST_END} \
  --metric UNBLENDED_COST \
  --granularity MONTHLY \
  --query '{ForecastedCost:Total.Amount,Unit:Total.Unit}' \
  --output table
```

**Key Metrics to Track**:
- Daily and monthly trends
- Cost by service
- Cost by tag (project, environment, team)
- Forecasted costs
- Reserved Instance utilization

### Step 5: Implement Cost Anomaly Detection

Enable ML-based cost anomaly detection:

```bash
# Create cost anomaly monitor for overall account
MONITOR_ARN=$(aws ce create-anomaly-monitor \
  --anomaly-monitor '{
    "MonitorName": "AccountMonitor",
    "MonitorType": "DIMENSIONAL",
    "MonitorDimension": "SERVICE"
  }' \
  --query 'MonitorArn' \
  --output text)

echo "Anomaly Monitor created: ${MONITOR_ARN}"

# Create anomaly subscription for alerts
SUBSCRIPTION_ARN=$(aws ce create-anomaly-subscription \
  --anomaly-subscription '{
    "SubscriptionName": "CostAnomalyAlerts",
    "Threshold": 100,
    "Frequency": "IMMEDIATE",
    "MonitorArnList": ["'${MONITOR_ARN}'"],
    "Subscribers": [
      {
        "Type": "SNS",
        "Address": "'${BUDGET_TOPIC_ARN}'"
      }
    ]
  }' \
  --query 'SubscriptionArn' \
  --output text)

echo "Anomaly Subscription created: ${SUBSCRIPTION_ARN}"

# Check for anomalies
aws ce get-anomalies \
  --date-interval Start=${START_DATE} \
  --max-results 10 \
  --query 'Anomalies[*].[AnomalyId,AnomalyScore.MaxScore,Impact.TotalImpact,RootCauses[0].Service]' \
  --output table
```

**Learning Points**:
- ML models detect unusual spending patterns
- Threshold controls alert sensitivity
- Immediate frequency provides real-time alerts
- Root cause analysis helps identify issues quickly

### Step 6: Review Trusted Advisor Recommendations

Check cost optimization opportunities:

```bash
# Note: Full Trusted Advisor API requires Business or Enterprise Support
# Using Support API to check Trusted Advisor

# List cost optimization checks
aws support describe-trusted-advisor-checks \
  --language en \
  --query 'checks[?category==`cost_optimizing`].[name,id,description]' \
  --output table

# Get specific check results (example: Underutilized EC2 instances)
# First, refresh the check
aws support refresh-trusted-advisor-check \
  --check-id Qch7DwouX1 # ID for "Low Utilization Amazon EC2 Instances"

# Wait a few seconds for refresh
sleep 10

# Get check results
aws support describe-trusted-advisor-check-result \
  --check-id Qch7DwouX1 \
  --query 'result.flaggedResources[*].[metadata]' \
  --output table
```

**Common Trusted Advisor Cost Checks**:
- Low utilization EC2 instances
- Unassociated Elastic IP addresses
- Idle RDS instances
- Underutilized EBS volumes
- Unassociated Elastic Load Balancers

### Step 7: Implement EC2 Right-Sizing

Use Compute Optimizer for instance recommendations:

```bash
# Enable Compute Optimizer (one-time setup)
aws compute-optimizer update-enrollment-status \
  --status Active

echo "Compute Optimizer enabled (recommendations available in 12-24 hours)"

# Get EC2 instance recommendations
aws compute-optimizer get-ec2-instance-recommendations \
  --query 'instanceRecommendations[*].[instanceArn,currentInstanceType,finding,recommendationOptions[0].instanceType,recommendationOptions[0].estimatedMonthlySavings.value]' \
  --output table

# Get recommendations for specific instance
aws compute-optimizer get-ec2-instance-recommendations \
  --instance-arns arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0 \
  --output json

# Script to automate right-sizing based on recommendations
cat > rightsizing-script.sh <<'EOF'
#!/bin/bash

# Get all under-provisioned and over-provisioned instances
RECOMMENDATIONS=$(aws compute-optimizer get-ec2-instance-recommendations \
  --filters Name=Finding,Values=Underprovisioned,Overprovisioned \
  --query 'instanceRecommendations[*].[instanceArn,currentInstanceType,finding,recommendationOptions[0].instanceType]' \
  --output text)

echo "Rightsizing Recommendations:"
echo "${RECOMMENDATIONS}"

# Parse and display savings
while read -r line; do
  INSTANCE_ARN=$(echo $line | awk '{print $1}')
  CURRENT_TYPE=$(echo $line | awk '{print $2}')
  FINDING=$(echo $line | awk '{print $3}')
  RECOMMENDED_TYPE=$(echo $line | awk '{print $4}')
  
  INSTANCE_ID=$(echo $INSTANCE_ARN | awk -F/ '{print $2}')
  
  echo "Instance: ${INSTANCE_ID}"
  echo "  Current: ${CURRENT_TYPE}"
  echo "  Finding: ${FINDING}"
  echo "  Recommended: ${RECOMMENDED_TYPE}"
  echo "  Action: Stop instance, modify type, restart"
  echo ""
  
  # Uncomment to auto-resize (use with caution!)
  # aws ec2 stop-instances --instance-ids ${INSTANCE_ID}
  # aws ec2 wait instance-stopped --instance-ids ${INSTANCE_ID}
  # aws ec2 modify-instance-attribute --instance-id ${INSTANCE_ID} --instance-type ${RECOMMENDED_TYPE}
  # aws ec2 start-instances --instance-ids ${INSTANCE_ID}
  
done <<< "${RECOMMENDATIONS}"
EOF

chmod +x rightsizing-script.sh
echo "Right-sizing script created: ./rightsizing-script.sh"
```

**Right-Sizing Best Practices**:
- Review utilization over at least 2 weeks
- Consider peak vs. average utilization
- Test in non-production first
- Schedule changes during maintenance windows
- Monitor performance after changes

### Step 8: Optimize Storage Costs

Implement S3 lifecycle policies and cleanup:

```bash
# Create S3 lifecycle policy
cat > s3-lifecycle-policy.json <<EOF
{
  "Rules": [
    {
      "Id": "TransitionToIA",
      "Status": "Enabled",
      "Filter": {
        "Prefix": ""
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER_IR"
        },
        {
          "Days": 180,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "STANDARD_IA"
        }
      ],
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    },
    {
      "Id": "DeleteOldLogs",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "logs/"
      },
      "Expiration": {
        "Days": 90
      }
    },
    {
      "Id": "CleanIncompleteUploads",
      "Status": "Enabled",
      "Filter": {
        "Prefix": ""
      },
      "AbortIncompleteMultipartUpload": {
        "DaysAfterInitiation": 7
      }
    }
  ]
}
EOF

# Apply lifecycle policy to bucket
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-application-bucket \
  --lifecycle-configuration file://s3-lifecycle-policy.json

echo "S3 lifecycle policy applied"

# Find and delete unattached EBS volumes
UNATTACHED_VOLUMES=$(aws ec2 describe-volumes \
  --filters Name=status,Values=available \
  --query 'Volumes[*].[VolumeId,Size,VolumeType,CreateTime]' \
  --output text)

echo "Unattached EBS Volumes:"
echo "${UNATTACHED_VOLUMES}"

# Script to clean up old snapshots
cat > cleanup-snapshots.sh <<'EOF'
#!/bin/bash

# Delete snapshots older than 90 days
CUTOFF_DATE=$(date -u -d '90 days ago' +%Y-%m-%d)

OLD_SNAPSHOTS=$(aws ec2 describe-snapshots \
  --owner-ids self \
  --query "Snapshots[?StartTime<='${CUTOFF_DATE}'].[SnapshotId,StartTime,Description]" \
  --output text)

echo "Snapshots older than 90 days:"
echo "${OLD_SNAPSHOTS}"

# Uncomment to delete (use with caution!)
# while read -r line; do
#   SNAPSHOT_ID=$(echo $line | awk '{print $1}')
#   echo "Deleting snapshot: ${SNAPSHOT_ID}"
#   aws ec2 delete-snapshot --snapshot-id ${SNAPSHOT_ID}
# done <<< "${OLD_SNAPSHOTS}"
EOF

chmod +x cleanup-snapshots.sh
echo "Snapshot cleanup script created"

# Find unused Elastic IPs
UNUSED_EIPS=$(aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' \
  --output table)

echo "Unused Elastic IPs (charged when not associated):"
echo "${UNUSED_EIPS}"
```

**Storage Cost Optimization**:
- Use appropriate storage classes
- Implement lifecycle policies
- Delete old snapshots and volumes
- Use S3 Intelligent-Tiering for unpredictable access
- Compress data before storing
- Release unused Elastic IPs

### Step 9: Implement Reserved Instances and Savings Plans

Analyze and purchase commitments for predictable workloads:

```bash
# Get RI recommendations
aws ce get-reservation-purchase-recommendation \
  --service "Amazon Elastic Compute Cloud - Compute" \
  --lookback-period-in-days SIXTY_DAYS \
  --term-in-years ONE_YEAR \
  --payment-option PARTIAL_UPFRONT \
  --query 'Recommendations[*].[RecommendationDetails.InstanceDetails.EC2InstanceDetails.InstanceType,RecommendationDetails.RecommendedNumberOfInstancesToPurchase,RecommendationDetails.EstimatedMonthlySavingsAmount]' \
  --output table

# Get Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --lookback-period-in-days SIXTY_DAYS \
  --term-in-years ONE_YEAR \
  --payment-option PARTIAL_UPFRONT \
  --query 'SavingsPlansPurchaseRecommendation.SavingsPlansPurchaseRecommendationDetails[*].[HourlyCommitmentToPurchase,EstimatedSavingsAmount,EstimatedMonthlySavingsAmount]' \
  --output table

# Check current RI utilization
aws ce get-reservation-utilization \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --query 'UtilizationsByTime[*].[TimePeriod.Start,Total.UtilizationPercentage]' \
  --output table

# Check Savings Plans utilization
aws ce get-savings-plans-utilization \
  --time-period Start=${START_DATE},End=${END_DATE} \
  --granularity MONTHLY \
  --query 'SavingsPlansUtilizationsByTime[*].[TimePeriod.Start,Total.Utilization.UtilizationPercentage]' \
  --output table
```

**Commitment Strategy**:
- Analyze 30-60 days of usage before purchasing
- Start with 1-year terms for flexibility
- Use Savings Plans for general compute commitments
- Use RIs for specific, predictable instance types
- Monitor utilization monthly
- Aim for 70-80% coverage of baseline usage

### Step 10: Create Cost Dashboard and Reports

Set up automated cost reporting:

```bash
# Create Lambda function for weekly cost reports
cat > cost-report-lambda.py <<'EOF'
import boto3
import json
from datetime import datetime, timedelta

ce = boto3.client('ce')
sns = boto3.client('sns')

def lambda_handler(event, context):
    """Send weekly cost report via SNS"""
    
    # Get last 7 days of costs
    end_date = datetime.utcnow().date()
    start_date = end_date - timedelta(days=7)
    
    # Get cost by service
    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': str(start_date),
            'End': str(end_date)
        },
        Granularity='DAILY',
        Metrics=['UnblendedCost'],
        GroupBy=[
            {
                'Type': 'DIMENSION',
                'Key': 'SERVICE'
            }
        ]
    )
    
    # Parse results
    total_cost = 0
    service_costs = {}
    
    for result in response['ResultsByTime']:
        for group in result['Groups']:
            service = group['Keys'][0]
            cost = float(group['Metrics']['UnblendedCost']['Amount'])
            service_costs[service] = service_costs.get(service, 0) + cost
            total_cost += cost
    
    # Format report
    report = f"AWS Cost Report ({start_date} to {end_date})\n\n"
    report += f"Total Cost: ${total_cost:.2f}\n\n"
    report += "Top Services:\n"
    
    sorted_services = sorted(service_costs.items(), key=lambda x: x[1], reverse=True)
    for service, cost in sorted_services[:10]:
        if cost > 0.01:  # Only show services with significant cost
            report += f"  {service}: ${cost:.2f}\n"
    
    # Send via SNS
    sns.publish(
        TopicArn=event['TopicArn'],
        Subject='Weekly AWS Cost Report',
        Message=report
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps('Report sent successfully')
    }
EOF

# Create IAM role for Lambda
cat > lambda-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name CostReportLambdaRole \
  --assume-role-policy-document file://lambda-trust-policy.json

# Attach policies
aws iam attach-role-policy \
  --role-name CostReportLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

cat > lambda-cost-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage",
        "ce:GetCostForecast"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "${BUDGET_TOPIC_ARN}"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name CostReportLambdaRole \
  --policy-name CostExplorerAccess \
  --policy-document file://lambda-cost-policy.json

# Get role ARN
LAMBDA_ROLE_ARN=$(aws iam get-role \
  --role-name CostReportLambdaRole \
  --query 'Role.Arn' \
  --output text)

# Wait for role propagation
sleep 10

# Create Lambda deployment package
zip cost-report-lambda.zip cost-report-lambda.py

# Create Lambda function
FUNCTION_ARN=$(aws lambda create-function \
  --function-name WeeklyCostReport \
  --runtime python3.9 \
  --role ${LAMBDA_ROLE_ARN} \
  --handler cost-report-lambda.lambda_handler \
  --zip-file fileb://cost-report-lambda.zip \
  --timeout 60 \
  --query 'FunctionArn' \
  --output text)

# Create EventBridge rule for weekly schedule (every Monday at 9 AM UTC)
RULE_ARN=$(aws events put-rule \
  --name WeeklyCostReportSchedule \
  --schedule-expression "cron(0 9 ? * MON *)" \
  --query 'RuleArn' \
  --output text)

# Add Lambda permission for EventBridge
aws lambda add-permission \
  --function-name WeeklyCostReport \
  --statement-id AllowEventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn ${RULE_ARN}

# Add Lambda as target
aws events put-targets \
  --rule WeeklyCostReportSchedule \
  --targets "Id=1,Arn=${FUNCTION_ARN},Input='{\"TopicArn\":\"${BUDGET_TOPIC_ARN}\"}'"

echo "Weekly cost report automation configured"
```

## Cost Optimization Checklist

### Compute
- [ ] Right-size EC2 instances based on utilization
- [ ] Use Spot Instances for fault-tolerant workloads
- [ ] Purchase Reserved Instances or Savings Plans for baseline
- [ ] Stop dev/test instances during off-hours
- [ ] Use Auto Scaling to match capacity to demand
- [ ] Consider Lambda for sporadic workloads

### Storage
- [ ] Implement S3 lifecycle policies
- [ ] Use appropriate storage classes (IA, Glacier)
- [ ] Delete old EBS snapshots
- [ ] Remove unattached EBS volumes
- [ ] Use EBS gp3 instead of gp2 when possible
- [ ] Enable S3 Intelligent-Tiering for unpredictable access

### Networking
- [ ] Use VPC endpoints to avoid NAT Gateway costs
- [ ] Minimize cross-region data transfer
- [ ] Use CloudFront for content delivery
- [ ] Review data transfer patterns
- [ ] Consolidate data transfers when possible

### Database
- [ ] Right-size RDS instances
- [ ] Use Aurora Serverless for variable workloads
- [ ] Stop non-production databases when not in use
- [ ] Use read replicas efficiently
- [ ] Enable automated backups retention policies

### General
- [ ] Implement comprehensive tagging
- [ ] Regular review of Trusted Advisor recommendations
- [ ] Monitor and act on Cost Anomaly Detection alerts
- [ ] Quarterly budget reviews and adjustments
- [ ] Regular cleanup of unused resources
- [ ] Enable cost allocation tags

## Common Cost Pitfalls to Avoid

### ❌ Expensive Mistakes:
1. **Data Transfer**: Not using VPC endpoints, excessive cross-region transfers
2. **Idle Resources**: Running resources 24/7 when only needed 8-10 hours/day
3. **Over-Provisioning**: Using larger instances than needed "just in case"
4. **No Lifecycle Policies**: Keeping all data in expensive storage forever
5. **Unused Resources**: Elastic IPs, load balancers, NAT gateways without traffic
6. **No Budgets**: Discovering overspend after the fact
7. **Poor Tagging**: Unable to attribute costs to projects/teams
8. **Ignoring Recommendations**: Not acting on Trusted Advisor findings

### ✅ Cost-Saving Actions:
1. **Right-Size**: Match resources to actual workload requirements
2. **Automate Schedules**: Stop/start non-production resources
3. **Use Commitments**: Purchase RIs/Savings Plans for predictable workloads
4. **Implement Lifecycle**: Move data to cheaper storage tiers automatically
5. **Regular Cleanup**: Monthly review and deletion of unused resources
6. **Proactive Monitoring**: Set up budgets and anomaly detection
7. **Tagging Discipline**: Enforce tagging policies organization-wide
8. **Act on Insights**: Regular review and implementation of recommendations

## Cleanup

Remove test resources:

```bash
# Delete Lambda function
aws lambda delete-function --function-name WeeklyCostReport

# Delete EventBridge rule
aws events remove-targets --rule WeeklyCostReportSchedule --ids 1
aws events delete-rule --name WeeklyCostReportSchedule

# Delete IAM role
aws iam delete-role-policy --role-name CostReportLambdaRole --policy-name CostExplorerAccess
aws iam detach-role-policy --role-name CostReportLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name CostReportLambdaRole

# Delete anomaly detection
aws ce delete-anomaly-subscription --subscription-arn ${SUBSCRIPTION_ARN}
aws ce delete-anomaly-monitor --monitor-arn ${MONITOR_ARN}

# Delete budgets
aws budgets delete-budget --account-id ${ACCOUNT_ID} --budget-name MonthlyTotalCostBudget
aws budgets delete-budget --account-id ${ACCOUNT_ID} --budget-name WebAppProjectBudget

# Delete SNS topic
aws sns delete-topic --topic-arn ${BUDGET_TOPIC_ARN}

# Empty and delete CUR bucket
aws s3 rm s3://${CUR_BUCKET} --recursive
aws s3 rb s3://${CUR_BUCKET}

# Clean up local files
rm -f budget-config.json budget-notifications.json project-budget.json \
      cur-bucket-policy.json s3-lifecycle-policy.json cost-report-lambda.py \
      cost-report-lambda.zip lambda-trust-policy.json lambda-cost-policy.json \
      rightsizing-script.sh cleanup-snapshots.sh tagging-strategy.txt
```

## Key Takeaways

✅ **Cost Visibility** is the foundation of cost management
✅ **Tagging** enables accountability and cost attribution
✅ **Budgets and Alerts** prevent unexpected spending
✅ **Right-Sizing** matches resources to actual needs
✅ **Automation** ensures consistent cost optimization
✅ **Regular Reviews** identify new optimization opportunities
✅ **Culture Matters** - make cost awareness everyone's responsibility

## Next Steps

- Explore AWS Cost Optimization Hub for centralized recommendations
- Implement AWS Organizations for multi-account cost management
- Set up showback/chargeback models for business units
- Integrate cost data with BI tools for custom reporting
- Establish FinOps practices across your organization
- Learn about AWS Well-Architected Cost Optimization Pillar

## Additional Resources

- [AWS Cost Management](https://aws.amazon.com/aws-cost-management/)
- [AWS Cost Optimization Best Practices](https://aws.amazon.com/pricing/cost-optimization/)
- [FinOps Foundation](https://www.finops.org/)
- [AWS Cost Optimization Blog](https://aws.amazon.com/blogs/aws-cost-management/)
- [AWS Pricing Calculator](https://calculator.aws/)
