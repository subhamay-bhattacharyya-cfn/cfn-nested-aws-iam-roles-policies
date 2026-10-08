# CloudFormation Nested AWS Templates Repository

<!-- Row 1: Status - Most Important -->
[![Release](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template)&nbsp;[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazon&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cloudformation-template)](https://github.com/subhamay-bhattacharyya-cfn/cloudformation-template/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/f55f73ac88992d4bd5c9835ee5fd70b6/raw/aws-vpc-cloudformation-fundamentals.json)](https://gist.github.com/subhamay-bhattacharyya/f55f73ac88992d4bd5c9835ee5fd70b6)

This repository contains reusable nested CloudFormation templates for deploying AWS resources with security best practices. Currently includes IAM roles with conditional inline policies and S3 buckets with optional policy enforcement.

## Overview

This is a collection of **nested stack templates** designed to be invoked from a parent/root CloudFormation stack. Templates are modular, parameterized, and follow AWS best practices for security, naming conventions, and resource management. Templates should be uploaded to an S3 bucket for reference by parent stacks.

## Template Files

### CloudFormation Templates

#### IAM Role Templates

- **`cloudformation/iam-role-inline-policies/template.yaml`** — Parameterized IAM role with conditional inline policies for common AWS use cases (data pipelines, compute, ML services, messaging, security, cross-account access)

#### S3 Bucket Templates

- **`templates/s3-bucket.yaml`** — Nested template for S3 bucket creation (versioning, public access blocking)
- **`templates/s3-bucket-policy.yaml`** — Optional nested template for S3 bucket policy (encryption enforcement, secure transport)

### Parameter Files

- **`cloudformation/iam-role-inline-policies/parameters.json`** — Parameter values for IAM role deployment
- **`cloudformation/iam-role-inline-policies/stack-config.json`** — Stack configuration
- **`parameters/parameters.json`** — Parameter values for S3 bucket deployment

## Template Features

### IAM Role Template (iam-role-inline-policies/template.yaml)

- ✅ Parameterized role naming with project prefix, environment, and region
- ✅ Optional CI suffix support for unique role identification
- ✅ Configurable assume role principal (Lambda, EC2, ECS, Step Functions, Glue, CodeBuild, SSM, EventBridge, DataPipeline)
- ✅ **Data Pipeline Policies:** S3 (read-only, write-only, read-write), Kinesis, DynamoDB (read-only, read-write), Glue, Athena
- ✅ **AI/ML Services:** Textract, Polly, Lex, Rekognition
- ✅ **Compute & Orchestration:** Lambda, Lambda Basic Execution, Step Functions, ECS, CodeBuild
- ✅ **Integration & Messaging:** SQS, SNS, SES, EventBridge, API Gateway
- ✅ **Security & Observability:** Secrets Manager, CloudWatch Logs, X-Ray, CloudTrail
- ✅ **Advanced Features:** KMS key operations, cross-account access, resource tagging
- ✅ Conditional inline policies (enable/disable individual policies via parameters)
- ✅ CloudFormation exports for cross-stack references

### S3 Bucket Template (s3-bucket.yaml)

- ✅ Versioning enabled by default
- ✅ Public Access Blocking
- ✅ Smart bucket naming (project prefix, account ID, environment, region)
- ✅ Optional CI suffix support

### S3 Bucket Policy Template (s3-bucket-policy.yaml)

- ✅ Encryption enforcement on uploads
- ✅ Secure transport enforcement (HTTPS only)
- ✅ Optional/conditional policy rules

## Parameters

### IAM Role Parameters (iam-role-inline-policies/template.yaml)

#### Basic Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | `cfn-nested` | Project name used as prefix for role naming |
| `RoleName` | String | `iam-role` | Name of the IAM role |
| `Environment` | String | `devl` | Deployment environment (devl, test, prod) |
| `AssumeRolePrincipal` | String | `lambda.amazonaws.com` | AWS service principal that can assume this role |
| `CiSuffix` | String | `""` | Optional CI suffix for unique role identification |

#### Feature Enable/Disable Flags

**Data Pipeline Policies:**

- `EnableS3ReadOnlyPolicy` (default: false) — S3 read-only access
- `EnableS3WritePolicy` (default: false) — S3 write-only access
- `EnableS3ReadWritePolicy` (default: false) — S3 read-write access
- `EnableKinesisPolicy` (default: false) — Kinesis data streams
- `EnableDynamoDBReadOnlyPolicy` (default: false) — DynamoDB read-only
- `EnableDynamoDBReadWritePolicy` (default: false) — DynamoDB read-write
- `EnableGluePolicy` (default: false) — AWS Glue ETL
- `EnableAthenaPolicy` (default: false) — Amazon Athena queries

**AI/ML Services:**

- `EnableTextractPolicy` (default: false) — Amazon Textract
- `EnablePollyPolicy` (default: false) — Amazon Polly
- `EnableLexPolicy` (default: false) — Amazon Lex
- `EnableRekognitionPolicy` (default: false) — Amazon Rekognition
- `EnableBedrockKnowledgeBasePolicy` (default: false) — Amazon Bedrock knowledge base Retrieve and RetrieveAndGenerate

**Compute & Orchestration:**

- `EnableLambdaPolicy` (default: false) — Lambda function invocation
- `EnableLambdaBasicExecutionPolicy` (default: false) — Lambda CloudWatch Logs
- `EnableStepFunctionsPolicy` (default: false) — Step Functions
- `EnableECSPolicy` (default: false) — ECS task management
- `EnableCodeBuildPolicy` (default: false) — CodeBuild

**Integration & Messaging:**

- `EnableSQSPolicy` (default: false) — SQS queue operations
- `EnableSNSPolicy` (default: false) — SNS topic publish
- `EnableSESPolicy` (default: false) — Amazon SES
- `EnableEventBridgePolicy` (default: false) — EventBridge
- `EnableAPIGatewayPolicy` (default: false) — API Gateway logging

**Security & Observability:**

- `EnableSecretsManagerPolicy` (default: false) — Secrets Manager
- `EnableCloudWatchPolicy` (default: false) — CloudWatch Logs and metrics
- `EnableXRayPolicy` (default: false) — AWS X-Ray
- `EnableCloudTrailPolicy` (default: false) — CloudTrail logs

**Advanced Features:**

- `EnableCrossAccountAccess` (default: false) — Cross-account role assumption
- `EnableKMSPolicy` (default: false) — KMS key operations
- `EnableTagging` (default: false) — Resource tagging

#### Resource ARN Parameters

| Parameter | Default | Description |
| ----------- | --------- | ------------- |
| `S3BucketArn` | `arn:aws:s3:::test-bucket-us-east-1` | S3 bucket ARN |
| `KinesisStreamArn` | `arn:aws:kinesis:*:*:stream/*` | Kinesis stream ARN pattern |
| `DynamoDBTableArn` | `arn:aws:dynamodb:*:*:table/*` | DynamoDB table ARN pattern |
| `GlueJobArn`, `GlueCatalogArn`, `GlueDatabaseArn`, `GlueTableArn` | Glue defaults | Glue resource ARN patterns |
| `AthenaWorkgroupArn`, `AthenaQueryResultsArn` | Athena defaults | Athena resource ARN patterns |
| `LambdaFunctionArn`, `LambdaBasicExecutionLogGroupArn` | Lambda defaults | Lambda resource ARN patterns |
| `StepFunctionsStateMachineArn`, `StepFunctionsExecutionArn` | Step Functions defaults | Step Functions resource ARN patterns |
| `ECSTaskArn`, `ECSTaskDefinitionArn` | ECS defaults | ECS resource ARN patterns |
| `SQSQueueArn`, `SNSTopicArn` | MQ defaults | SQS/SNS resource ARN patterns |
| `SecretsManagerArn`, `CloudWatchLogsArn`, `KMSKeyArn` | Default patterns | Security/Observability resource ARN patterns |
| `CrossAccountRoleArn` | `arn:aws:iam::*:role/*` | Cross-account IAM role ARN pattern |
| `BedrockKnowledgeBaseArn` | `arn:aws:bedrock:*:*:knowledge-base/*` | Bedrock knowledge base ARN pattern |

### S3 Bucket Parameters

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | — | Project name to use as bucket prefix (required) |
| `BucketBaseName` | String | `cfn-bucket` | Base name for S3 bucket |
| `environment` | String | `devl` | Deployment environment (devl, stag, prod) |
| `CiSuffix` | String | `""` | Optional CI suffix to append to bucket name |

### S3 Bucket Policy Parameters

**Generic (Standalone) Mode:**

| Parameter | Type | Description |
| ----------- | ------ | ------------- |
| `BucketName` | String | Direct bucket name (use for any bucket) |

**Integrated Mode (with bucket template):**

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | — | Project name (must match bucket template) |
| `BucketBaseName` | String | `cfn-bucket` | Base name (must match bucket template) |
| `environment` | String | `devl` | Environment (must match bucket template) |
| `CiSuffix` | String | `""` | CI suffix (must match bucket template) |

**Usage:** If `BucketName` is provided (non-empty), it takes precedence. Otherwise, the bucket name is constructed from ProjectName/BucketBaseName/environment/CiSuffix.

## Outputs

### IAM Role Template Outputs

- `RoleArn` — ARN of the created IAM role (exported for cross-stack reference)
- `RoleName` — Name of the created IAM role (exported for cross-stack reference)
- `AssumeRoleCommand` — AWS CLI command to assume the role
- `CloudFormationStackId` — CloudFormation stack ID
- `DeploymentInfo` — Deployment summary including role, environment, principal, and assume role command

### S3 Bucket Template Outputs

- `S3BucketName` — S3 bucket name
- `S3BucketArn` — S3 bucket ARN

### S3 Bucket Policy Template Outputs

- `BucketName` — Bucket name with policy applied
- `PolicyStatus` — Policy application status (Applied)

## Usage

### IAM Role Deployment

#### Basic Deployment (Lambda Execution Role with CloudWatch Logs)

```bash
aws cloudformation create-stack \
  --stack-name my-lambda-role \
  --template-body file://cloudformation/iam-role-inline-policies/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=lambda-executor \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=AssumeRolePrincipal,ParameterValue=lambda.amazonaws.com \
    ParameterKey=EnableLambdaPolicy,ParameterValue=true \
    ParameterKey=EnableLambdaBasicExecutionPolicy,ParameterValue=true \
    ParameterKey=EnableCloudWatchPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

#### Data Pipeline Role (S3 + Glue + Athena Access)

```bash
aws cloudformation create-stack \
  --stack-name my-data-pipeline-role \
  --template-body file://cloudformation/iam-role-inline-policies/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=data-pipeline \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=AssumeRolePrincipal,ParameterValue=glue.amazonaws.com \
    ParameterKey=EnableS3ReadWritePolicy,ParameterValue=true \
    ParameterKey=EnableGluePolicy,ParameterValue=true \
    ParameterKey=EnableAthenaPolicy,ParameterValue=true \
    ParameterKey=S3BucketArn,ParameterValue=arn:aws:s3:::my-data-bucket \
  --capabilities CAPABILITY_NAMED_IAM
```

#### With CI Suffix (for CI/CD Deployments)

```bash
aws cloudformation create-stack \
  --stack-name my-role-ci \
  --template-body file://cloudformation/iam-role-inline-policies/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=lambda-executor \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID \
    ParameterKey=EnableLambdaPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

#### Stack Update

```bash
aws cloudformation update-stack \
  --stack-name my-lambda-role \
  --template-body file://cloudformation/iam-role-inline-policies/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=lambda-executor \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=AssumeRolePrincipal,ParameterValue=lambda.amazonaws.com \
    ParameterKey=EnableCloudWatchPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### S3 Bucket Deployment

### 1. Upload Templates to S3

```bash
aws s3 cp templates/s3-bucket.yaml s3://your-cfn-bucket/templates/s3-bucket.yaml
aws s3 cp templates/s3-bucket-policy.yaml s3://your-cfn-bucket/templates/s3-bucket-policy.yaml
```

### 2. Reference from Parent Stack

In your parent/root CloudFormation template:

```yaml
S3BucketNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/s3-bucket.yaml
    Parameters:
      ProjectName: !Ref ProjectName
      BucketBaseName: cfn-bucket
      environment: !Ref Environment
      CiSuffix: !Ref CiSuffix
    Tags:
      - Key: Environment
        Value: !Ref Environment

S3PolicyNestedStack:
  Type: AWS::CloudFormation::Stack
  DependsOn: S3BucketNestedStack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/s3-bucket-policy.yaml
    Parameters:
      BucketName: !GetAtt S3BucketNestedStack.Outputs.S3BucketName
      ProjectName: ""
      BucketBaseName: cfn-bucket
      environment: !Ref Environment
      CiSuffix: !Ref CiSuffix
    Tags:
      - Key: Environment
        Value: !Ref Environment

Outputs:
  BucketName:
    Value: !GetAtt S3BucketNestedStack.Outputs.S3BucketName
  BucketArn:
    Value: !GetAtt S3BucketNestedStack.Outputs.S3BucketArn
```

### 3. Deploy Using AWS CLI

#### Option A: Deploy Bucket Only

```bash
# Development (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/dev.json

# Staging (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-stag \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/staging.json

# Production (without CI prefix)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-prod \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/prod.json
```

#### Option B: Deploy Bucket + Policy (Recommended)

```bash
# Deploy bucket first
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev \
  --template-body file://templates/s3-bucket.yaml \
  --parameters file://parameters/dev.json

# Wait for bucket to be created
aws cloudformation wait stack-create-complete --stack-name cfn-s3-bucket-dev

# Get the bucket name from stack outputs
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name cfn-s3-bucket-dev \
  --query 'Stacks[0].Outputs[?OutputKey==`S3BucketName`].OutputValue' \
  --output text)

#### Using Integrated Mode (with bucket template parameters)

```bash
aws cloudformation create-stack \
  --stack-name cfn-s3-policy-dev \
  --template-body file://templates/s3-bucket-policy.yaml \
  --parameters file://parameters/policy-dev.json
```

#### Using Generic Mode (standalone with direct bucket name)

```bash
aws cloudformation create-stack \
  --stack-name cfn-s3-policy-dev \
  --template-body file://templates/s3-bucket-policy.yaml \
  --parameters \
    ParameterKey=BucketName,ParameterValue=my-existing-bucket \
    ParameterKey=ProjectName,ParameterValue=""
```

#### Option C: Deploy with CI Suffix

```bash
# Development with CI suffix (e.g., for GitLab CI)
aws cloudformation create-stack \
  --stack-name cfn-s3-bucket-dev-ci \
  --template-body file://templates/s3-bucket.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=BucketBaseName,ParameterValue=cfn-bucket \
    ParameterKey=environment,ParameterValue=devl \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID
```

## Bucket Naming Convention

The templates generate bucket names using the following pattern:

**Without CI Suffix:**

```bash
{ProjectName}-{BucketBaseName}-{AccountId}-{Environment}-{Region}
```

Example: `myproject-cfn-bucket-123456789012-devl-us-east-1`

**With CI Suffix:**

```bash
{ProjectName}-{BucketBaseName}-{AccountId}-{Environment}-{Region}-{CiSuffix}
```

Example: `myproject-cfn-bucket-123456789012-devl-us-east-1-pipeline-12345`

## Best Practices Implemented

### IAM Role Best Practices

- ✅ Principle of least privilege with conditional inline policies
- ✅ Parameterized role naming with project prefix, environment, and region
- ✅ Optional CI suffix support for unique role identification in testing
- ✅ Resource ARN parameters for fine-grained access control
- ✅ CloudFormation exports for cross-stack references
- ✅ Organized policy categories (data pipeline, AI/ML, compute, messaging, security, observability)
- ✅ Metadata documentation within CloudFormation template

### S3 Bucket Best Practices

- ✅ Versioning enabled by default
- ✅ Public access blocked by default
- ✅ Smart bucket naming with project prefix, account ID, environment, and region
- ✅ Optional CI suffix support for unique deployments
- ✅ Optional policy enforcement (encryption and secure transport)
- ✅ Export values for cross-stack references

## License

MIT
