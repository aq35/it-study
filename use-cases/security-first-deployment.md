# Use Case: Security-First Deployment on AWS

## Overview

This use case demonstrates how to deploy a secure, production-ready application on AWS by implementing security best practices from the ground up. You'll learn about network isolation, identity and access management, data encryption, and security monitoring.

## Problem Statement

Your organization needs to deploy a web application that handles sensitive customer data. Security requirements include:

- **Network Security**: Isolate resources in a private network with controlled access
- **Access Control**: Implement least privilege access for users and services
- **Data Protection**: Encrypt data both in transit and at rest
- **Audit and Compliance**: Track all access and changes to resources
- **Secrets Management**: Store and rotate credentials securely
- **Incident Detection**: Monitor for security threats and anomalies

Current challenges:
- Default AWS configurations are too permissive
- Credentials are often hardcoded in applications
- Network traffic is not properly segmented
- Lack of visibility into security events
- Compliance requirements must be met

You need a comprehensive security architecture that protects data while maintaining operational efficiency.

## Key Learning Objectives

By completing this use case, you will:

1. **Design Secure Network Architecture**
   - Create VPCs with public and private subnets
   - Configure security groups and NACLs
   - Implement network segmentation and isolation

2. **Apply IAM Best Practices**
   - Use roles instead of access keys
   - Implement least privilege policies
   - Enable MFA for sensitive operations

3. **Implement Data Encryption**
   - Encrypt data at rest using KMS
   - Enforce TLS for data in transit
   - Manage encryption keys securely

4. **Manage Secrets Securely**
   - Use AWS Secrets Manager for credentials
   - Implement automatic secret rotation
   - Avoid hardcoded secrets in code

5. **Enable Security Monitoring**
   - Configure CloudTrail for audit logging
   - Use CloudWatch for security metrics
   - Set up alerts for suspicious activity

6. **Follow Security Compliance**
   - Implement security controls
   - Enable compliance reporting
   - Regular security assessments

## Expected Outcomes

After completing this use case, you will be able to:
- Design and implement secure AWS architectures
- Apply defense-in-depth security principles
- Protect sensitive data comprehensively
- Meet security compliance requirements
- Detect and respond to security incidents

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Secure Application Architecture                       │
└─────────────────────────────────────────────────────────────────────────┘

                              Internet
                                 │
                                 │ HTTPS (443)
                                 ▼
                          ┌─────────────┐
                          │   Internet  │
                          │   Gateway   │
                          └──────┬──────┘
                                 │
    ┌────────────────────────────┼────────────────────────────┐
    │                           VPC                            │
    │                    10.0.0.0/16                          │
    │                                                          │
    │  ┌───────────────────────────────────────────────────┐  │
    │  │           Public Subnet (10.0.1.0/24)            │  │
    │  │         Availability Zone A                       │  │
    │  │                                                   │  │
    │  │  ┌──────────────┐         ┌──────────────┐      │  │
    │  │  │ Application  │         │     NAT      │      │  │
    │  │  │Load Balancer │         │   Gateway    │      │  │
    │  │  │   (ALB)      │         │              │      │  │
    │  │  └──────┬───────┘         └──────┬───────┘      │  │
    │  │         │                         │              │  │
    │  └─────────┼─────────────────────────┼──────────────┘  │
    │            │                         │                  │
    │            │ Route to Private        │                  │
    │            │                         │ Internet Access  │
    │            ▼                         ▼                  │
    │  ┌───────────────────────────────────────────────────┐  │
    │  │          Private Subnet (10.0.2.0/24)            │  │
    │  │         Availability Zone A                       │  │
    │  │                                                   │  │
    │  │  ┌──────────────┐         ┌──────────────┐      │  │
    │  │  │     EC2      │         │     EC2      │      │  │
    │  │  │ Web Server 1 │         │ Web Server 2 │      │  │
    │  │  │              │         │              │      │  │
    │  │  │ [IAM Role]   │         │ [IAM Role]   │      │  │
    │  │  └──────┬───────┘         └──────┬───────┘      │  │
    │  │         │                         │              │  │
    │  │         │ Access RDS              │              │  │
    │  │         ▼                         ▼              │  │
    │  └───────────────────────────────────────────────────┘  │
    │            │                         │                  │
    │            │                         │                  │
    │  ┌─────────┴─────────────────────────┴──────────────┐  │
    │  │          Private Subnet (10.0.3.0/24)            │  │
    │  │         Database Subnet                           │  │
    │  │                                                   │  │
    │  │  ┌──────────────────────────────────┐            │  │
    │  │  │         Amazon RDS               │            │  │
    │  │  │   (Encrypted at Rest)            │            │  │
    │  │  │   Multi-AZ Deployment            │            │  │
    │  │  └──────────────────────────────────┘            │  │
    │  └───────────────────────────────────────────────────┘  │
    │                                                          │
    └──────────────────────────────────────────────────────────┘

              Security & Monitoring Layer
    ┌─────────────────────────────────────────────┐
    │  ┌──────────────┐      ┌──────────────┐    │
    │  │     AWS      │      │   AWS KMS    │    │
    │  │  CloudTrail  │      │  (Encryption │    │
    │  │  (Audit Log) │      │     Keys)    │    │
    │  └──────────────┘      └──────────────┘    │
    │                                             │
    │  ┌──────────────┐      ┌──────────────┐    │
    │  │   Secrets    │      │  CloudWatch  │    │
    │  │   Manager    │      │   (Metrics   │    │
    │  │ (Credentials)│      │   & Alarms)  │    │
    │  └──────────────┘      └──────────────┘    │
    └─────────────────────────────────────────────┘
```

## Component Descriptions

### Network Layer
- **VPC**: Isolated network environment
- **Public Subnets**: Host load balancers and NAT gateways
- **Private Subnets**: Host application servers (no direct internet access)
- **Database Subnets**: Dedicated subnets for database instances

### Security Controls
- **Security Groups**: Stateful firewalls at instance level
- **NACLs**: Stateless firewalls at subnet level
- **IAM Roles**: Grant permissions without access keys
- **KMS**: Manage encryption keys centrally

### Monitoring & Compliance
- **CloudTrail**: Log all API calls for audit
- **CloudWatch**: Monitor metrics and set alarms
- **Secrets Manager**: Store and rotate credentials
- **VPC Flow Logs**: Network traffic analysis

## Step-by-Step Implementation

### Prerequisites

Before starting, ensure you have:
- [ ] AWS account with administrative access
- [ ] AWS CLI installed and configured
- [ ] Basic understanding of networking concepts
- [ ] Familiarity with Linux and SSH

### Step 1: Create VPC with Private and Public Subnets

Create a secure network foundation:

```bash
# Set variables
VPC_NAME="secure-app-vpc"
VPC_CIDR="10.0.0.0/16"
PUBLIC_SUBNET_CIDR="10.0.1.0/24"
PRIVATE_APP_SUBNET_CIDR="10.0.2.0/24"
PRIVATE_DB_SUBNET_CIDR="10.0.3.0/24"
REGION="us-east-1"
AZ_A="${REGION}a"

# Create VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block ${VPC_CIDR} \
  --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=${VPC_NAME}}]" \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC Created: ${VPC_ID}"

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
  --vpc-id ${VPC_ID} \
  --enable-dns-hostnames

# Create Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=${VPC_NAME}-igw}]" \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

# Attach Internet Gateway to VPC
aws ec2 attach-internet-gateway \
  --vpc-id ${VPC_ID} \
  --internet-gateway-id ${IGW_ID}

echo "Internet Gateway attached: ${IGW_ID}"

# Create Public Subnet
PUBLIC_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block ${PUBLIC_SUBNET_CIDR} \
  --availability-zone ${AZ_A} \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-a}]" \
  --query 'Subnet.SubnetId' \
  --output text)

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --map-public-ip-on-launch

echo "Public Subnet created: ${PUBLIC_SUBNET_ID}"

# Create Private Application Subnet
PRIVATE_APP_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block ${PRIVATE_APP_SUBNET_CIDR} \
  --availability-zone ${AZ_A} \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=private-app-subnet-a}]" \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Private App Subnet created: ${PRIVATE_APP_SUBNET_ID}"

# Create Private Database Subnet
PRIVATE_DB_SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block ${PRIVATE_DB_SUBNET_CIDR} \
  --availability-zone ${AZ_A} \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=private-db-subnet-a}]" \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Private DB Subnet created: ${PRIVATE_DB_SUBNET_ID}"
```

**Learning Points**:
- Public subnets have routes to Internet Gateway
- Private subnets use NAT Gateway for outbound internet
- Separating app and database subnets provides additional isolation

### Step 2: Configure Route Tables

Set up routing for public and private subnets:

```bash
# Create Route Table for Public Subnet
PUBLIC_RT_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]" \
  --query 'RouteTable.RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id ${PUBLIC_RT_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${IGW_ID}

# Associate with Public Subnet
aws ec2 associate-route-table \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --route-table-id ${PUBLIC_RT_ID}

echo "Public route table configured: ${PUBLIC_RT_ID}"

# Allocate Elastic IP for NAT Gateway
EIP_ALLOC_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --query 'AllocationId' \
  --output text)

# Create NAT Gateway in Public Subnet
NAT_GW_ID=$(aws ec2 create-nat-gateway \
  --subnet-id ${PUBLIC_SUBNET_ID} \
  --allocation-id ${EIP_ALLOC_ID} \
  --tag-specifications "ResourceType=nat-gateway,Tags=[{Key=Name,Value=${VPC_NAME}-nat}]" \
  --query 'NatGateway.NatGatewayId' \
  --output text)

echo "NAT Gateway created: ${NAT_GW_ID} (waiting for availability...)"

# Wait for NAT Gateway to become available
aws ec2 wait nat-gateway-available --nat-gateway-ids ${NAT_GW_ID}

# Create Route Table for Private Subnets
PRIVATE_RT_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=private-rt}]" \
  --query 'RouteTable.RouteTableId' \
  --output text)

# Add route to NAT Gateway
aws ec2 create-route \
  --route-table-id ${PRIVATE_RT_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id ${NAT_GW_ID}

# Associate with Private Subnets
aws ec2 associate-route-table \
  --subnet-id ${PRIVATE_APP_SUBNET_ID} \
  --route-table-id ${PRIVATE_RT_ID}

aws ec2 associate-route-table \
  --subnet-id ${PRIVATE_DB_SUBNET_ID} \
  --route-table-id ${PRIVATE_RT_ID}

echo "Private route table configured: ${PRIVATE_RT_ID}"
```

**Learning Points**:
- NAT Gateway allows private instances to access internet for updates
- NAT Gateway is placed in public subnet
- Private instances cannot receive inbound connections from internet

### Step 3: Create Security Groups with Least Privilege

Implement granular firewall rules:

```bash
# Security Group for Application Load Balancer
ALB_SG_ID=$(aws ec2 create-security-group \
  --group-name alb-security-group \
  --description "Security group for Application Load Balancer" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow HTTPS from internet
aws ec2 authorize-security-group-ingress \
  --group-id ${ALB_SG_ID} \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow HTTP (for redirect to HTTPS)
aws ec2 authorize-security-group-ingress \
  --group-id ${ALB_SG_ID} \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

echo "ALB Security Group created: ${ALB_SG_ID}"

# Security Group for Web Servers
WEB_SG_ID=$(aws ec2 create-security-group \
  --group-name web-server-security-group \
  --description "Security group for web servers" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow traffic only from ALB
aws ec2 authorize-security-group-ingress \
  --group-id ${WEB_SG_ID} \
  --protocol tcp \
  --port 80 \
  --source-group ${ALB_SG_ID}

echo "Web Server Security Group created: ${WEB_SG_ID}"

# Security Group for Database
DB_SG_ID=$(aws ec2 create-security-group \
  --group-name database-security-group \
  --description "Security group for RDS database" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow MySQL/PostgreSQL only from web servers
aws ec2 authorize-security-group-ingress \
  --group-id ${DB_SG_ID} \
  --protocol tcp \
  --port 3306 \
  --source-group ${WEB_SG_ID}

echo "Database Security Group created: ${DB_SG_ID}"

# Optional: Bastion host security group for admin access
BASTION_SG_ID=$(aws ec2 create-security-group \
  --group-name bastion-security-group \
  --description "Security group for bastion host" \
  --vpc-id ${VPC_ID} \
  --query 'GroupId' \
  --output text)

# Allow SSH from your IP only (replace with your IP)
YOUR_IP="$(curl -s https://checkip.amazonaws.com)/32"
aws ec2 authorize-security-group-ingress \
  --group-id ${BASTION_SG_ID} \
  --protocol tcp \
  --port 22 \
  --cidr ${YOUR_IP}

echo "Bastion Security Group created: ${BASTION_SG_ID}"

# Allow SSH from bastion to web servers
aws ec2 authorize-security-group-ingress \
  --group-id ${WEB_SG_ID} \
  --protocol tcp \
  --port 22 \
  --source-group ${BASTION_SG_ID}
```

**Security Best Practices**:
- ✅ Each layer has dedicated security group
- ✅ Only allow necessary ports
- ✅ Use source security groups instead of IP ranges
- ✅ Limit SSH access to bastion host only
- ✅ No direct SSH access to application servers from internet

### Step 4: Create KMS Key for Encryption

Set up encryption key management:

```bash
# Create KMS key for data encryption
KMS_KEY_ID=$(aws kms create-key \
  --description "Key for encrypting application data" \
  --query 'KeyMetadata.KeyId' \
  --output text)

# Create alias for easier reference
aws kms create-alias \
  --alias-name alias/app-data-key \
  --target-key-id ${KMS_KEY_ID}

echo "KMS Key created: ${KMS_KEY_ID}"

# Get KMS key ARN
KMS_KEY_ARN=$(aws kms describe-key \
  --key-id ${KMS_KEY_ID} \
  --query 'KeyMetadata.Arn' \
  --output text)

echo "KMS Key ARN: ${KMS_KEY_ARN}"
```

**Learning Points**:
- KMS keys are never exposed or downloadable
- Automatic key rotation can be enabled
- Fine-grained access control via key policies
- Integrated with most AWS services

### Step 5: Create IAM Roles with Least Privilege

Configure secure access for EC2 instances:

```bash
# Create trust policy for EC2
cat > ec2-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create IAM role for web servers
aws iam create-role \
  --role-name WebServerRole \
  --assume-role-policy-document file://ec2-trust-policy.json

# Create policy for web server permissions
cat > web-server-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "arn:aws:secretsmanager:${REGION}:*:secret:app/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt"
      ],
      "Resource": "${KMS_KEY_ARN}"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::app-assets-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:${REGION}:*:log-group:/aws/ec2/*"
    }
  ]
}
EOF

# Attach policy to role
aws iam put-role-policy \
  --role-name WebServerRole \
  --policy-name WebServerPermissions \
  --policy-document file://web-server-policy.json

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name WebServerInstanceProfile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name WebServerInstanceProfile \
  --role-name WebServerRole

echo "IAM role and instance profile created"
```

**IAM Best Practices**:
- ✅ Use roles instead of access keys
- ✅ Grant minimum required permissions
- ✅ Use resource-level permissions when possible
- ✅ Separate policies for different responsibilities
- ✅ Regular permission audits

### Step 6: Set Up AWS Secrets Manager

Store database credentials securely:

```bash
# Create database credentials in Secrets Manager
SECRET_VALUE=$(cat <<EOF
{
  "username": "appadmin",
  "password": "$(openssl rand -base64 32)",
  "engine": "mysql",
  "host": "database-placeholder.rds.amazonaws.com",
  "port": 3306,
  "dbname": "appdb"
}
EOF
)

SECRET_ARN=$(aws secretsmanager create-secret \
  --name app/database/credentials \
  --description "Database credentials for application" \
  --secret-string "${SECRET_VALUE}" \
  --kms-key-id ${KMS_KEY_ID} \
  --query 'ARN' \
  --output text)

echo "Secret created: ${SECRET_ARN}"

# Enable automatic rotation (30 days)
# Note: Requires Lambda function for rotation
# aws secretsmanager rotate-secret \
#   --secret-id app/database/credentials \
#   --rotation-lambda-arn <lambda-arn> \
#   --rotation-rules AutomaticallyAfterDays=30
```

**Learning Points**:
- Secrets are encrypted with KMS
- No hardcoded credentials in code or config files
- Automatic rotation improves security
- Fine-grained access control via IAM

### Step 7: Enable CloudTrail for Audit Logging

Track all API activity:

```bash
# Create S3 bucket for CloudTrail logs
CLOUDTRAIL_BUCKET="cloudtrail-logs-$(date +%s)"
aws s3 mb s3://${CLOUDTRAIL_BUCKET}

# Create bucket policy for CloudTrail
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

cat > cloudtrail-bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::${CLOUDTRAIL_BUCKET}"
    },
    {
      "Sid": "AWSCloudTrailWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::${CLOUDTRAIL_BUCKET}/AWSLogs/${ACCOUNT_ID}/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    }
  ]
}
EOF

aws s3api put-bucket-policy \
  --bucket ${CLOUDTRAIL_BUCKET} \
  --policy file://cloudtrail-bucket-policy.json

# Create CloudTrail
aws cloudtrail create-trail \
  --name security-audit-trail \
  --s3-bucket-name ${CLOUDTRAIL_BUCKET} \
  --is-multi-region-trail \
  --enable-log-file-validation

# Start logging
aws cloudtrail start-logging \
  --name security-audit-trail

echo "CloudTrail enabled and logging to s3://${CLOUDTRAIL_BUCKET}"
```

**Learning Points**:
- CloudTrail logs all API calls
- Multi-region trails capture activity across all regions
- Log file validation detects tampering
- Essential for compliance and security investigations

### Step 8: Enable VPC Flow Logs

Monitor network traffic:

```bash
# Create CloudWatch log group for VPC Flow Logs
aws logs create-log-group \
  --log-group-name /aws/vpc/flowlogs

# Create IAM role for VPC Flow Logs
cat > flowlogs-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "vpc-flow-logs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name VPCFlowLogsRole \
  --assume-role-policy-document file://flowlogs-trust-policy.json

# Create policy for Flow Logs
cat > flowlogs-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name VPCFlowLogsRole \
  --policy-name VPCFlowLogsPermissions \
  --policy-document file://flowlogs-policy.json

# Get role ARN
FLOWLOGS_ROLE_ARN=$(aws iam get-role \
  --role-name VPCFlowLogsRole \
  --query 'Role.Arn' \
  --output text)

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids ${VPC_ID} \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs \
  --deliver-logs-permission-arn ${FLOWLOGS_ROLE_ARN}

echo "VPC Flow Logs enabled"
```

**Use Cases for VPC Flow Logs**:
- Troubleshoot connectivity issues
- Detect unusual traffic patterns
- Analyze security group rules effectiveness
- Meet compliance requirements

### Step 9: Set Up Security Monitoring Alarms

Create alerts for security events:

```bash
# Create SNS topic for security alerts
SECURITY_TOPIC_ARN=$(aws sns create-topic \
  --name security-alerts \
  --query 'TopicArn' \
  --output text)

# Subscribe email to topic
aws sns subscribe \
  --topic-arn ${SECURITY_TOPIC_ARN} \
  --protocol email \
  --notification-endpoint security-team@example.com

echo "Please confirm SNS subscription via email"

# Create CloudWatch alarm for unauthorized API calls
aws cloudwatch put-metric-alarm \
  --alarm-name UnauthorizedAPICalls \
  --alarm-description "Alert on unauthorized API calls" \
  --metric-name UnauthorizedAPICalls \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions ${SECURITY_TOPIC_ARN}

# Create metric filter for unauthorized calls
aws logs put-metric-filter \
  --log-group-name CloudTrail/logs \
  --filter-name UnauthorizedAPICalls \
  --filter-pattern '{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }' \
  --metric-transformations \
    metricName=UnauthorizedAPICalls,metricNamespace=CloudTrailMetrics,metricValue=1

# Alarm for root account usage
aws cloudwatch put-metric-alarm \
  --alarm-name RootAccountUsage \
  --alarm-description "Alert when root account is used" \
  --metric-name RootAccountUsage \
  --namespace CloudTrailMetrics \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --alarm-actions ${SECURITY_TOPIC_ARN}

# Metric filter for root usage
aws logs put-metric-filter \
  --log-group-name CloudTrail/logs \
  --filter-name RootAccountUsage \
  --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
  --metric-transformations \
    metricName=RootAccountUsage,metricNamespace=CloudTrailMetrics,metricValue=1

echo "Security monitoring alarms created"
```

**Key Metrics to Monitor**:
- Unauthorized API calls
- Root account usage
- IAM policy changes
- Security group modifications
- Failed authentication attempts
- Large data transfers

### Step 10: Deploy Application with Security Best Practices

Deploy a sample secure application:

```bash
# Create user data script for web server
cat > user-data.sh <<'EOF'
#!/bin/bash
# Update system
yum update -y

# Install dependencies
yum install -y httpd mod_ssl aws-cli jq

# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Retrieve database credentials from Secrets Manager
SECRET_JSON=$(aws secretsmanager get-secret-value \
  --secret-id app/database/credentials \
  --region us-east-1 \
  --query SecretString \
  --output text)

DB_HOST=$(echo $SECRET_JSON | jq -r .host)
DB_USER=$(echo $SECRET_JSON | jq -r .username)
DB_PASS=$(echo $SECRET_JSON | jq -r .password)

# Configure application (example)
cat > /var/www/html/config.php <<PHPEOF
<?php
// Database configuration from Secrets Manager
define('DB_HOST', '$DB_HOST');
define('DB_USER', '$DB_USER');
define('DB_PASS', '$DB_PASS');
define('DB_NAME', 'appdb');

// Force HTTPS
if (empty(\$_SERVER['HTTPS']) || \$_SERVER['HTTPS'] === 'off') {
    header('Location: https://' . \$_SERVER['HTTP_HOST'] . \$_SERVER['REQUEST_URI']);
    exit;
}
?>
PHPEOF

# Start web server
systemctl start httpd
systemctl enable httpd

# Configure SSL/TLS (use ACM certificate in production)
# For demo, using self-signed cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/pki/tls/private/server.key \
  -out /etc/pki/tls/certs/server.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Configure Apache for HTTPS
cat > /etc/httpd/conf.d/ssl-custom.conf <<APACHEEOF
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/html
    
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/server.crt
    SSLCertificateKeyFile /etc/pki/tls/private/server.key
    
    # Modern SSL configuration
    SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
    SSLHonorCipherOrder on
    
    # Security headers
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</VirtualHost>

<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
APACHEEOF

systemctl restart httpd
EOF

# Launch EC2 instance with security configurations
# Get latest Amazon Linux 2 AMI
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text)

# Launch instance in private subnet
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id ${AMI_ID} \
  --instance-type t3.micro \
  --subnet-id ${PRIVATE_APP_SUBNET_ID} \
  --security-group-ids ${WEB_SG_ID} \
  --iam-instance-profile Name=WebServerInstanceProfile \
  --user-data file://user-data.sh \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=SecureWebServer},{Key=Environment,Value=Production}]" \
  --metadata-options "HttpTokens=required,HttpPutResponseHopLimit=1" \
  --monitoring Enabled=true \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "EC2 instance launched: ${INSTANCE_ID}"
```

**Security Features Implemented**:
- ✅ Instance in private subnet (no direct internet access)
- ✅ IAM role for secure AWS API access
- ✅ Secrets retrieved from Secrets Manager
- ✅ IMDSv2 enforced (HttpTokens=required)
- ✅ Detailed monitoring enabled
- ✅ TLS/SSL enforced
- ✅ Security headers configured
- ✅ HTTP to HTTPS redirect

## Security Validation Checklist

### Network Security
- [ ] VPC with proper CIDR ranges
- [ ] Public and private subnets configured
- [ ] Security groups follow least privilege
- [ ] NACLs provide defense in depth
- [ ] NAT Gateway for private subnet internet access
- [ ] No unnecessary ports exposed

### Access Management
- [ ] IAM roles used instead of access keys
- [ ] Policies follow least privilege principle
- [ ] MFA enabled for console access
- [ ] No root account usage
- [ ] Service-specific roles created

### Data Protection
- [ ] Data encrypted at rest (KMS)
- [ ] Data encrypted in transit (TLS/SSL)
- [ ] Encryption keys properly managed
- [ ] Secrets stored in Secrets Manager
- [ ] No hardcoded credentials

### Monitoring & Compliance
- [ ] CloudTrail enabled and logging
- [ ] VPC Flow Logs capturing traffic
- [ ] CloudWatch alarms for security events
- [ ] Log retention configured
- [ ] Security alerts routed to team

## Common Security Misconfigurations to Avoid

### ❌ Don't Do This:
1. **Open Security Groups**: 0.0.0.0/0 on SSH (port 22)
2. **Public Database Access**: RDS in public subnet
3. **Hardcoded Credentials**: Passwords in code or config files
4. **No Encryption**: Unencrypted S3 buckets or EBS volumes
5. **Overly Permissive IAM**: Administrator access for applications
6. **No Logging**: CloudTrail or VPC Flow Logs disabled
7. **Default Passwords**: Using default or weak passwords
8. **No Network Segmentation**: All resources in one subnet

### ✅ Do This Instead:
1. **Restricted Security Groups**: Allow only necessary IPs and ports
2. **Private Database**: Place RDS in private subnet
3. **Secrets Manager**: Store credentials securely
4. **Encryption Everywhere**: Use KMS for encryption at rest
5. **Least Privilege IAM**: Grant minimum required permissions
6. **Comprehensive Logging**: Enable all audit and monitoring logs
7. **Strong Credentials**: Use Secrets Manager to generate passwords
8. **Network Layers**: Separate subnets for web, app, and database tiers

## Cost Optimization for Security

Security doesn't have to be expensive:

1. **VPC Flow Logs**: Filter to capture only rejected traffic
2. **CloudTrail**: Single organizational trail instead of per-account
3. **KMS**: Use AWS managed keys for less sensitive data
4. **NAT Gateway**: Share across multiple private subnets
5. **CloudWatch Logs**: Set retention periods appropriately
6. **GuardDuty**: Enable only in production accounts

## Compliance Considerations

This architecture supports various compliance frameworks:

- **PCI DSS**: Network segmentation, encryption, access control, logging
- **HIPAA**: Data encryption, audit logs, access controls
- **SOC 2**: Security monitoring, change tracking, access management
- **GDPR**: Data protection, encryption, audit trails

## Cleanup

Remove all resources to avoid charges:

```bash
# Terminate EC2 instances
aws ec2 terminate-instances --instance-ids ${INSTANCE_ID}
aws ec2 wait instance-terminated --instance-ids ${INSTANCE_ID}

# Delete NAT Gateway
aws ec2 delete-nat-gateway --nat-gateway-id ${NAT_GW_ID}
sleep 60

# Release Elastic IP
aws ec2 release-address --allocation-id ${EIP_ALLOC_ID}

# Delete Security Groups
aws ec2 delete-security-group --group-id ${DB_SG_ID}
aws ec2 delete-security-group --group-id ${WEB_SG_ID}
aws ec2 delete-security-group --group-id ${ALB_SG_ID}
aws ec2 delete-security-group --group-id ${BASTION_SG_ID}

# Delete VPC resources
aws ec2 delete-subnet --subnet-id ${PUBLIC_SUBNET_ID}
aws ec2 delete-subnet --subnet-id ${PRIVATE_APP_SUBNET_ID}
aws ec2 delete-subnet --subnet-id ${PRIVATE_DB_SUBNET_ID}
aws ec2 delete-route-table --route-table-id ${PUBLIC_RT_ID}
aws ec2 delete-route-table --route-table-id ${PRIVATE_RT_ID}
aws ec2 detach-internet-gateway --vpc-id ${VPC_ID} --internet-gateway-id ${IGW_ID}
aws ec2 delete-internet-gateway --internet-gateway-id ${IGW_ID}
aws ec2 delete-vpc --vpc-id ${VPC_ID}

# Delete CloudTrail and logs
aws cloudtrail delete-trail --name security-audit-trail
aws s3 rm s3://${CLOUDTRAIL_BUCKET} --recursive
aws s3 rb s3://${CLOUDTRAIL_BUCKET}

# Delete Secrets Manager secret
aws secretsmanager delete-secret \
  --secret-id app/database/credentials \
  --force-delete-without-recovery

# Schedule KMS key deletion (minimum 7 days)
aws kms schedule-key-deletion \
  --key-id ${KMS_KEY_ID} \
  --pending-window-in-days 7

# Delete IAM resources
aws iam remove-role-from-instance-profile \
  --instance-profile-name WebServerInstanceProfile \
  --role-name WebServerRole
aws iam delete-instance-profile --instance-profile-name WebServerInstanceProfile
aws iam delete-role-policy --role-name WebServerRole --policy-name WebServerPermissions
aws iam delete-role --role-name WebServerRole
aws iam delete-role-policy --role-name VPCFlowLogsRole --policy-name VPCFlowLogsPermissions
aws iam delete-role --role-name VPCFlowLogsRole

# Delete CloudWatch resources
aws logs delete-log-group --log-group-name /aws/vpc/flowlogs
aws cloudwatch delete-alarms --alarm-names UnauthorizedAPICalls RootAccountUsage
aws sns delete-topic --topic-arn ${SECURITY_TOPIC_ARN}

# Clean up local files
rm -f ec2-trust-policy.json web-server-policy.json flowlogs-trust-policy.json \
      flowlogs-policy.json cloudtrail-bucket-policy.json user-data.sh
```

## Key Takeaways

✅ **Defense in Depth**: Multiple layers of security controls
✅ **Least Privilege**: Grant only necessary permissions
✅ **Encryption**: Protect data at rest and in transit
✅ **Monitoring**: Continuous visibility into security posture
✅ **Secrets Management**: Never hardcode credentials
✅ **Network Isolation**: Separate public and private resources
✅ **Audit Logging**: Track all access and changes

## Next Steps

- Implement AWS WAF for application-layer protection
- Enable AWS Config for compliance monitoring
- Set up GuardDuty for threat detection
- Explore Security Hub for centralized security view
- Study AWS Systems Manager for patch management
- Learn about AWS Organizations for multi-account security

## Additional Resources

- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [AWS Security Documentation](https://docs.aws.amazon.com/security/)
