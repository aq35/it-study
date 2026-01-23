# Use Case: Event-Driven Architecture with AWS Lambda, S3, and SQS

## Overview

This use case demonstrates how to build a scalable, event-driven system that automatically processes files uploaded to Amazon S3. You'll learn how AWS services work together seamlessly to handle workloads without manual intervention, making your applications more responsive and efficient.

## Problem Statement

Your organization receives hundreds of data files daily from various sources. These files need to be processed, validated, and stored in different formats. Manual processing is time-consuming and error-prone. You need an automated solution that:

- Automatically detects when new files arrive
- Processes files asynchronously to handle high volumes
- Retries failed operations automatically
- Scales automatically based on workload
- Provides visibility into processing status

## Key Learning Objectives

By completing this use case, you will:

1. **Understand Event-Driven Architecture**
   - Learn how events trigger automatic actions
   - Understand loose coupling between components
   - Grasp the benefits of asynchronous processing

2. **Configure S3 Event Notifications**
   - Set up S3 to emit events on object creation
   - Understand S3 event structure and metadata
   - Configure event filtering and routing

3. **Implement AWS Lambda Functions**
   - Write serverless functions to process events
   - Handle errors and implement retry logic
   - Optimize function performance and cost

4. **Use SQS for Reliable Processing**
   - Create queues for decoupling components
   - Implement dead-letter queues for failed messages
   - Understand message visibility and retention

5. **Monitor and Troubleshoot**
   - Use CloudWatch Logs for debugging
   - Set up alarms for failure detection
   - Track processing metrics

## Expected Outcomes

After completing this use case, you will be able to:
- Design and implement event-driven architectures
- Build resilient, self-healing systems
- Handle high-volume workloads automatically
- Apply these patterns to various business scenarios

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Event-Driven Processing Flow                 │
└─────────────────────────────────────────────────────────────────┘

    Upload File
        │
        ▼
   ┌─────────┐           S3 Event           ┌──────────────┐
   │ Amazon  │────────Notification──────────▶│   Amazon     │
   │   S3    │                               │     SQS      │
   │ Bucket  │                               │    Queue     │
   └─────────┘                               └──────┬───────┘
        │                                           │
        │                                           │ Poll for
        │ Read Object                               │ Messages
        │                                           │
        ▼                                           ▼
   ┌─────────┐                               ┌──────────────┐
   │  File   │                               │     AWS      │
   │ Content │◀──────────────────────────────│    Lambda    │
   │         │         Get Object            │   Function   │
   └─────────┘                               └──────┬───────┘
                                                    │
                                                    │ Failed
                                                    │ Messages
                                                    ▼
                                             ┌──────────────┐
                                             │  Dead Letter │
                                             │    Queue     │
                                             │    (DLQ)     │
                                             └──────────────┘
                                                    │
                                                    │ Alert
                                                    ▼
                                             ┌──────────────┐
                                             │   Amazon     │
                                             │     SNS      │
                                             │    Topic     │
                                             └──────────────┘
```

## Component Descriptions

### 1. Amazon S3 (Source Bucket)
- **Purpose**: Stores incoming files
- **Configuration**: Event notifications enabled
- **Access**: Lambda needs read permissions

### 2. Amazon SQS (Processing Queue)
- **Purpose**: Buffers events for reliable processing
- **Benefits**: Decouples S3 from Lambda, enables batch processing
- **Configuration**: Standard queue with visibility timeout

### 3. AWS Lambda (Processor Function)
- **Purpose**: Processes files automatically
- **Trigger**: SQS queue
- **Runtime**: Python 3.9+ (or your preferred language)

### 4. Dead Letter Queue (DLQ)
- **Purpose**: Captures failed messages for investigation
- **Configuration**: Receives messages after max retries
- **Monitoring**: Alerts when messages arrive

### 5. Amazon SNS (Notifications)
- **Purpose**: Alerts team about failures
- **Subscribers**: Email, SMS, or other notification channels

## Step-by-Step Implementation

### Prerequisites

Before starting, ensure you have:
- [ ] AWS account with appropriate permissions
- [ ] AWS CLI installed and configured
- [ ] Basic understanding of Python (or your chosen language)
- [ ] Familiarity with JSON and YAML

### Step 1: Create the S3 Source Bucket

Create an S3 bucket to receive incoming files:

```bash
# Set your bucket name (must be globally unique)
BUCKET_NAME="your-event-driven-source-$(date +%s)"

# Create the bucket
aws s3 mb s3://${BUCKET_NAME} --region us-east-1

# Enable versioning for better file tracking
aws s3api put-bucket-versioning \
  --bucket ${BUCKET_NAME} \
  --versioning-configuration Status=Enabled
```

**Learning Point**: S3 bucket names must be globally unique. Using a timestamp ensures uniqueness.

### Step 2: Create SQS Queues

Create the main processing queue and a dead-letter queue:

```bash
# Create the Dead Letter Queue first
DLQ_URL=$(aws sqs create-queue \
  --queue-name event-driven-dlq \
  --query 'QueueUrl' \
  --output text)

# Get the DLQ ARN
DLQ_ARN=$(aws sqs get-queue-attributes \
  --queue-url ${DLQ_URL} \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

# Create the main processing queue with DLQ configured
QUEUE_URL=$(aws sqs create-queue \
  --queue-name event-driven-processing-queue \
  --attributes '{
    "MessageRetentionPeriod": "86400",
    "VisibilityTimeout": "300",
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"'${DLQ_ARN}'\",\"maxReceiveCount\":\"3\"}"
  }' \
  --query 'QueueUrl' \
  --output text)

# Get the queue ARN
QUEUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url ${QUEUE_URL} \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' \
  --output text)

echo "Queue URL: ${QUEUE_URL}"
echo "Queue ARN: ${QUEUE_ARN}"
```

**Learning Points**:
- Dead Letter Queue (DLQ) captures messages that fail processing after 3 attempts
- Visibility timeout (300s) ensures messages aren't reprocessed while being handled
- Message retention (86400s = 24 hours) keeps messages available for processing

### Step 3: Configure S3 to Send Events to SQS

First, create a queue policy to allow S3 to send messages:

```bash
# Create queue policy file
cat > queue-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SQS:SendMessage",
      "Resource": "${QUEUE_ARN}",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::${BUCKET_NAME}"
        }
      }
    }
  ]
}
EOF

# Apply the policy to the queue
aws sqs set-queue-attributes \
  --queue-url ${QUEUE_URL} \
  --attributes file://queue-policy.json

# Configure S3 event notification
cat > s3-notification.json <<EOF
{
  "QueueConfigurations": [
    {
      "QueueArn": "${QUEUE_ARN}",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {
              "Name": "prefix",
              "Value": "incoming/"
            },
            {
              "Name": "suffix",
              "Value": ".json"
            }
          ]
        }
      }
    }
  ]
}
EOF

# Apply notification configuration to S3 bucket
aws s3api put-bucket-notification-configuration \
  --bucket ${BUCKET_NAME} \
  --notification-configuration file://s3-notification.json
```

**Learning Points**:
- S3 needs explicit permission to send messages to SQS
- Event filtering reduces unnecessary processing (only .json files in incoming/ folder)
- Multiple event types can trigger notifications (Put, Post, Copy, CompleteMultipartUpload)

### Step 4: Create IAM Role for Lambda

Create a role with permissions to read from S3 and SQS:

```bash
# Create trust policy
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

# Create the role
aws iam create-role \
  --role-name EventDrivenLambdaRole \
  --assume-role-policy-document file://lambda-trust-policy.json

# Attach AWS managed policy for Lambda basic execution
aws iam attach-role-policy \
  --role-name EventDrivenLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create custom policy for S3 and SQS access
cat > lambda-permissions.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "arn:aws:s3:::${BUCKET_NAME}/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "${QUEUE_ARN}"
    }
  ]
}
EOF

# Create and attach the custom policy
aws iam put-role-policy \
  --role-name EventDrivenLambdaRole \
  --policy-name EventDrivenPermissions \
  --policy-document file://lambda-permissions.json

# Get the role ARN (wait a few seconds for role propagation)
sleep 10
ROLE_ARN=$(aws iam get-role \
  --role-name EventDrivenLambdaRole \
  --query 'Role.Arn' \
  --output text)

echo "Role ARN: ${ROLE_ARN}"
```

**Learning Points**:
- Lambda needs permissions to write logs (AWSLambdaBasicExecutionRole)
- Principle of least privilege: only grant necessary S3 and SQS permissions
- Roles allow Lambda to access AWS resources securely without hardcoded credentials

### Step 5: Create the Lambda Function

Create a Lambda function to process the files:

```python
# Save as lambda_function.py
import json
import boto3
import os
from datetime import datetime

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    """
    Process files uploaded to S3 via SQS events
    """
    print(f"Processing {len(event['Records'])} records")
    
    successful = 0
    failed = 0
    
    for record in event['Records']:
        try:
            # Parse SQS message body (contains S3 event)
            s3_event = json.loads(record['body'])
            
            # Extract S3 event details
            for s3_record in s3_event['Records']:
                bucket_name = s3_record['s3']['bucket']['name']
                object_key = s3_record['s3']['object']['key']
                
                print(f"Processing file: s3://{bucket_name}/{object_key}")
                
                # Get the object from S3
                response = s3_client.get_object(
                    Bucket=bucket_name,
                    Key=object_key
                )
                
                # Read and parse the content
                file_content = response['Body'].read().decode('utf-8')
                data = json.loads(file_content)
                
                # Process the data (example: validation and transformation)
                processed_data = process_file_data(data, object_key)
                
                # Store processed result (example: write to another S3 location)
                output_key = object_key.replace('incoming/', 'processed/')
                s3_client.put_object(
                    Bucket=bucket_name,
                    Key=output_key,
                    Body=json.dumps(processed_data, indent=2),
                    ContentType='application/json'
                )
                
                print(f"Successfully processed: {object_key} -> {output_key}")
                successful += 1
                
        except Exception as e:
            print(f"Error processing record: {str(e)}")
            failed += 1
            # Re-raise to trigger retry mechanism
            # Note: In production, implement granular error handling to distinguish
            # between retryable errors (temporary issues) and non-retryable errors
            # (data format issues) to prevent infinite retry loops
            raise
    
    result = {
        'statusCode': 200,
        'body': json.dumps({
            'successful': successful,
            'failed': failed,
            'timestamp': datetime.utcnow().isoformat()
        })
    }
    
    print(f"Processing complete: {successful} successful, {failed} failed")
    return result

def process_file_data(data, filename):
    """
    Process and validate the file data
    Customize this based on your business logic
    """
    processed = {
        'source_file': filename,
        'processed_at': datetime.utcnow().isoformat(),
        'record_count': len(data) if isinstance(data, list) else 1,
        'data': data
    }
    
    # Add your validation and transformation logic here
    # Example: Check required fields, transform formats, enrich data
    
    return processed
```

Package and deploy the Lambda function:

```bash
# Create deployment package
zip lambda_function.zip lambda_function.py

# Create the Lambda function
aws lambda create-function \
  --function-name EventDrivenFileProcessor \
  --runtime python3.9 \
  --role ${ROLE_ARN} \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://lambda_function.zip \
  --timeout 300 \
  --memory-size 512 \
  --environment Variables={BUCKET_NAME=${BUCKET_NAME}}

# Get the function ARN
FUNCTION_ARN=$(aws lambda get-function \
  --function-name EventDrivenFileProcessor \
  --query 'Configuration.FunctionArn' \
  --output text)

echo "Function ARN: ${FUNCTION_ARN}"
```

**Learning Points**:
- Lambda function processes SQS messages in batches for efficiency
- Error handling is crucial: exceptions trigger retries via SQS
- Environment variables make functions configurable
- Timeout and memory should match workload requirements

### Step 6: Connect SQS to Lambda

Configure Lambda to be triggered by SQS:

```bash
# Create event source mapping
aws lambda create-event-source-mapping \
  --function-name EventDrivenFileProcessor \
  --event-source-arn ${QUEUE_ARN} \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5

echo "Event source mapping created successfully"
```

**Learning Points**:
- Batch size determines how many messages Lambda processes at once
- Batching window allows collecting messages for efficiency
- Lambda automatically polls SQS and manages scaling

### Step 7: Set Up Monitoring and Alerts

Create CloudWatch alarms for the DLQ:

```bash
# Create SNS topic for alerts
TOPIC_ARN=$(aws sns create-topic \
  --name event-driven-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe your email to the topic (replace with your actual email)
# REPLACE 'your-email@example.com' with your actual email address
aws sns subscribe \
  --topic-arn ${TOPIC_ARN} \
  --protocol email \
  --notification-endpoint your-email@example.com

echo "Please check your email and confirm the SNS subscription"

# Create CloudWatch alarm for DLQ messages
aws cloudwatch put-metric-alarm \
  --alarm-name EventDrivenDLQAlarm \
  --alarm-description "Alert when messages arrive in DLQ" \
  --namespace AWS/SQS \
  --metric-name ApproximateNumberOfMessagesVisible \
  --dimensions Name=QueueName,Value=event-driven-dlq \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions ${TOPIC_ARN}

echo "CloudWatch alarm created"
```

**Learning Points**:
- Monitoring DLQ helps identify systematic failures
- SNS enables multi-channel notifications (email, SMS, Lambda, etc.)
- Proactive alerts prevent issues from going unnoticed

### Step 8: Test the System

Test the complete workflow:

```bash
# Create a test file
cat > test-data.json <<EOF
{
  "transaction_id": "TXN-001",
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "amount": 100.50,
  "currency": "USD",
  "status": "pending"
}
EOF

# Upload to S3 (triggers the workflow)
aws s3 cp test-data.json s3://${BUCKET_NAME}/incoming/test-data.json

echo "File uploaded. Processing should start automatically."
echo "Check CloudWatch Logs for function execution details."

# Wait a few seconds and check the processed output
sleep 10
aws s3 ls s3://${BUCKET_NAME}/processed/

# Download and verify the processed file
aws s3 cp s3://${BUCKET_NAME}/processed/test-data.json processed-result.json
cat processed-result.json
```

**Monitor the execution**:

```bash
# View Lambda logs
aws logs tail /aws/lambda/EventDrivenFileProcessor --follow

# Check SQS queue metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/SQS \
  --metric-name NumberOfMessagesSent \
  --dimensions Name=QueueName,Value=event-driven-processing-queue \
  --start-time $(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 60 \
  --statistics Sum
```

### Step 9: Test Failure Scenarios

Understand how the system handles failures:

```bash
# Upload an invalid JSON file to trigger an error
echo "Invalid JSON content" > invalid.json
aws s3 cp invalid.json s3://${BUCKET_NAME}/incoming/invalid.json

# Monitor DLQ for failed messages
aws sqs receive-message \
  --queue-url ${DLQ_URL} \
  --max-number-of-messages 10
```

**Learning Points**:
- Failed messages automatically move to DLQ after max retries
- DLQ preserves failed messages for investigation
- Alerts notify you of processing issues

## Advanced Enhancements

Once you've mastered the basics, try these enhancements:

### 1. Add Processing States
Use DynamoDB to track processing status:
- Record when files arrive
- Update status as processing progresses
- Enable reprocessing of failed items

### 2. Implement Partial Batch Failure
Configure Lambda to report partial failures:
```python
# Return failed message IDs to retry only those
return {
    'batchItemFailures': [
        {'itemIdentifier': message_id}
        for message_id in failed_messages
    ]
}
```

### 3. Add Content-Based Routing
Use SNS with message filtering:
- Route different file types to different processors
- Fan out to multiple consumers
- Implement pub/sub patterns

### 4. Implement Circuit Breaker
Pause processing when downstream services fail:
- Monitor error rates
- Temporarily disable event source mapping
- Prevent cascading failures

## Cost Optimization Tips

1. **Right-size Lambda memory**: Test different memory settings to find optimal cost/performance
2. **Use SQS batching**: Process multiple messages per invocation
3. **Set appropriate retention**: Don't keep messages longer than needed
4. **Monitor invocations**: Use CloudWatch to identify optimization opportunities
5. **Use S3 Lifecycle policies**: Move or delete processed files automatically

## Common Troubleshooting

### Issue: Lambda not triggered by SQS
- **Check**: Event source mapping is active
- **Check**: Lambda has permissions to poll SQS
- **Check**: Queue has messages (ApproximateNumberOfMessagesVisible > 0)

### Issue: Files processed multiple times
- **Check**: Lambda function completes within visibility timeout
- **Check**: Function doesn't throw exceptions after successful processing
- **Check**: Increase visibility timeout if processing takes longer

### Issue: Messages going to DLQ immediately
- **Check**: Lambda function has S3 read permissions
- **Check**: Bucket name in code matches actual bucket
- **Check**: File format matches parsing logic

## Cleanup

To avoid ongoing charges, delete all resources:

```bash
# Delete Lambda function
aws lambda delete-function --function-name EventDrivenFileProcessor

# Delete event source mapping (get UUID first)
MAPPING_UUID=$(aws lambda list-event-source-mappings \
  --function-name EventDrivenFileProcessor \
  --query 'EventSourceMappings[0].UUID' \
  --output text 2>/dev/null)
if [ ! -z "$MAPPING_UUID" ]; then
  aws lambda delete-event-source-mapping --uuid ${MAPPING_UUID}
fi

# Delete SQS queues
aws sqs delete-queue --queue-url ${QUEUE_URL}
aws sqs delete-queue --queue-url ${DLQ_URL}

# Delete S3 bucket contents and bucket
aws s3 rm s3://${BUCKET_NAME} --recursive
aws s3 rb s3://${BUCKET_NAME}

# Delete IAM role and policies
aws iam delete-role-policy \
  --role-name EventDrivenLambdaRole \
  --policy-name EventDrivenPermissions
aws iam detach-role-policy \
  --role-name EventDrivenLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name EventDrivenLambdaRole

# Delete SNS topic
aws sns delete-topic --topic-arn ${TOPIC_ARN}

# Delete CloudWatch alarm
aws cloudwatch delete-alarms --alarm-names EventDrivenDLQAlarm

# Clean up local files
rm -f queue-policy.json s3-notification.json lambda-trust-policy.json \
      lambda-permissions.json lambda_function.py lambda_function.zip \
      test-data.json invalid.json processed-result.json
```

## Key Takeaways

✅ **Event-driven architecture** enables automatic, scalable processing
✅ **SQS** provides reliable message delivery and retry mechanisms
✅ **Lambda** processes events without managing servers
✅ **S3 event notifications** trigger workflows based on file operations
✅ **Dead Letter Queues** capture failures for investigation
✅ **Monitoring and alerts** ensure system health

## Next Steps

- Explore Amazon EventBridge for more complex event routing
- Learn about Step Functions for orchestrating multi-step workflows
- Investigate Kinesis for real-time streaming data
- Study AWS SAM or Serverless Framework for infrastructure as code

## Additional Resources

- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/sqs/)
- [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)
- [Event-Driven Architecture Best Practices](https://aws.amazon.com/event-driven-architecture/)
