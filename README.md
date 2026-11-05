# CloudFormation Nested Stack Template : IAM Role with attached Managed and Inline policies

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

This repository contains a reusable nested CloudFormation template that creates an IAM role with conditionally attached managed policies for common AWS use cases.

## Overview

The template is designed to be invoked from a parent/root CloudFormation stack. It is modular and parameterized, and follows AWS best practices for least privilege, naming, and tagging. Upload the template to an S3 bucket so parent stacks can reference it by `TemplateURL`.

## Repository Layout

```text
cloudformation/
├── template.yaml        # IAM role with conditional managed policies
├── parameters.json      # Parameter values for deployment
└── stack-config.json    # Stack name plus template and parameter file paths
```

## Template Features

- ✅ Role naming with project prefix, environment, and region: `{ProjectName}-{RoleName}-{Environment}-{Region}`
- ✅ Optional CI suffix appended to the role name for unique identification in testing
- ✅ Configurable assume role principal (Lambda, EC2, ECS, Step Functions, Glue, CodeBuild, SSM, EventBridge, Data Pipeline, Bedrock AgentCore)
- ✅ Each policy is a separate `AWS::IAM::ManagedPolicy` with a fixed, descriptive name (for example `S3ReadOnly`, `SSMParameterRead`)
- ✅ Enable or disable each policy individually with a `true`/`false` parameter
- ✅ Resource ARN parameters for fine-grained scoping of every policy
- ✅ CloudFormation exports for cross-stack references

### Policy Categories

- **Data Pipeline:** S3 (read-only, write-only, read-write), Kinesis, DynamoDB (read-only, read-write), Glue, Athena
- **AI/ML Services:** Textract, Polly, Lex, Rekognition
- **Bedrock:** RAG access (knowledge base retrieve, embedding model invoke, OpenSearch Serverless) and AgentCore access
- **Parameter Store:** SSM Parameter Store read access under the project path
- **Compute & Orchestration:** Lambda invocation, Lambda basic execution, Step Functions, ECS, CodeBuild
- **Integration & Messaging:** SQS, SNS, SES, EventBridge, API Gateway logging
- **Security & Observability:** Secrets Manager, CloudWatch Logs and metrics, X-Ray, CloudTrail
- **Advanced:** Cross-account role assumption, KMS, resource tagging

When `EnableBedrockAgentCorePolicy` is `true`, the role also attaches these AWS managed policies:

- `AmazonBedrockFullAccess`
- `BedrockAgentCoreFullAccess`
- `CloudWatchLogsFullAccess`
- `AmazonEC2ContainerRegistryReadOnly`

## Parameters

### Basic Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | `cfn-nested` | Project name used as prefix for role naming and as the SSM path (`/{ProjectName}`) |
| `RoleName` | String | `iam-role` | Name of the IAM role |
| `Environment` | String | `devl` | Environment for tagging and naming (`devl`, `test`, `prod`) |
| `AssumeRolePrincipal` | String | `lambda.amazonaws.com` | AWS service principal that can assume this role |
| `CiSuffix` | String | `""` | Optional CI suffix for unique role identification |

### Feature Enable/Disable Flags

All flags accept `'true'` or `'false'` and default to `'false'`.

**Data Pipeline:**

- `EnableS3ReadOnlyPolicy` — S3 read-only access
- `EnableS3WritePolicy` — S3 write-only access
- `EnableS3ReadWritePolicy` — S3 read-write access
- `EnableKinesisPolicy` — Kinesis data streams
- `EnableDynamoDBReadOnlyPolicy` — DynamoDB read-only
- `EnableDynamoDBReadWritePolicy` — DynamoDB read-write
- `EnableGluePolicy` — AWS Glue ETL
- `EnableAthenaPolicy` — Amazon Athena queries

**AI/ML Services:**

- `EnableTextractPolicy` — Amazon Textract
- `EnablePollyPolicy` — Amazon Polly
- `EnableLexPolicy` — Amazon Lex
- `EnableRekognitionPolicy` — Amazon Rekognition
- `EnableBedrockRAGAccessPolicy` — Bedrock knowledge base retrieval and embedding model invocation
- `EnableBedrockAgentCorePolicy` — Bedrock AgentCore runtime, memory, events, and workload identity tokens

**SSM Parameter Store:**

- `EnableSSMParameterPolicy` — Read parameters under `/{ProjectName}` (`GetParameter`, `GetParameters`, `GetParametersByPath`)

**Compute & Orchestration:**

- `EnableLambdaPolicy` — Lambda function invocation
- `EnableLambdaBasicExecutionPolicy` — Lambda CloudWatch Logs
- `EnableStepFunctionsPolicy` — Step Functions
- `EnableECSPolicy` — ECS task management
- `EnableCodeBuildPolicy` — CodeBuild

**Integration & Messaging:**

- `EnableSQSPolicy` — SQS queue operations
- `EnableSNSPolicy` — SNS topic publish
- `EnableSESPolicy` — Amazon SES
- `EnableEventBridgePolicy` — EventBridge
- `EnableAPIGatewayPolicy` — API Gateway logging

**Security & Observability:**

- `EnableSecretsManagerPolicy` — Secrets Manager
- `EnableCloudWatchPolicy` — CloudWatch Logs and metrics
- `EnableXRayPolicy` — AWS X-Ray
- `EnableCloudTrailPolicy` — CloudTrail logs

**Cross-Account & Advanced:**

- `EnableCrossAccountAccess` — Cross-account role assumption (also requires `CrossAccountId`)
- `EnableKMSPolicy` — KMS key operations
- `EnableTagging` — Resource tagging

### Resource ARN Parameters

| Parameter | Default | Description |
| ----------- | --------- | ------------- |
| `S3BucketArn` | `arn:aws:s3:::test-bucket-us-east-1` | S3 bucket ARN(s) |
| `KinesisStreamArn` | `arn:aws:kinesis:*:*:stream/*` | Kinesis stream ARN pattern |
| `DynamoDBTableArn` | `arn:aws:dynamodb:*:*:table/*` | DynamoDB table ARN pattern |
| `GlueJobArn`, `GlueCatalogArn`, `GlueDatabaseArn`, `GlueTableArn` | Glue patterns | Glue resource ARN patterns |
| `AthenaWorkgroupArn`, `AthenaQueryResultsArn` | Athena patterns | Athena resource ARN patterns |
| `TextractPolicy`, `PollyPolicy`, `LexPolicy`, `RekognitionPolicy` | `Resource: '*'` | Not configurable; these services use `*` |
| `BedrockKnowledgeBaseArn` | `arn:aws:bedrock:*:*:knowledge-base/*` | Bedrock knowledge base ARN pattern |
| `BedrockEmbeddingModelArn` | Titan Embed Text v2 in `us-east-1` | Embedding model used by the knowledge base |
| `KnowledgeBaseCollectionArn` | `arn:aws:aoss:*:*:collection/*` | OpenSearch Serverless collection ARN pattern |
| `LambdaFunctionArn`, `LambdaBasicExecutionLogGroupArn` | Lambda patterns | Lambda resource ARN patterns |
| `StepFunctionsStateMachineArn`, `StepFunctionsExecutionArn` | Step Functions patterns | Step Functions resource ARN patterns |
| `ECSTaskArn`, `ECSTaskDefinitionArn`, `IAMRoleArn` | ECS and IAM patterns | ECS and PassRole target ARN patterns |
| `CodeBuildProjectArn` | `arn:aws:codebuild:*:*:project/*` | CodeBuild project ARN pattern |
| `SQSQueueArn`, `SNSTopicArn`, `EventBridgeArn` | Messaging patterns | SQS, SNS, and EventBridge ARN patterns |
| `SecretsManagerArn`, `CloudWatchLogsArn`, `KMSKeyArn` | Security patterns | Secrets Manager, CloudWatch Logs, and KMS ARN patterns |
| `CrossAccountRoleArn` | `arn:aws:iam::*:role/*` | Cross-account IAM role ARN pattern (required if `EnableCrossAccountAccess` is `true`) |

## Managed Policy Names

Each managed policy has a fixed `ManagedPolicyName`, for example `S3ReadOnly`, `KMS`, or `SSMParameterRead`. Because these names are literal, IAM requires them to be unique within the AWS account. Deploy only one stack from this template per account, or add a prefix before deploying a second copy.

## Outputs

- `RoleArn` — ARN of the created IAM role (exported as `{Environment}-{RoleName}-Arn`)
- `RoleName` — Name of the created IAM role (exported as `{Environment}-{RoleName}-Name`)
- `AssumeRoleCommand` — AWS CLI command to assume the role
- `CloudFormationStackId` — CloudFormation stack ID
- `DeploymentInfo` — Deployment summary including role, environment, principal, and assume role command

## Usage

### Deployment

Deploy with the parameter file:

```bash
aws cloudformation deploy \
  --stack-name my-lambda-role \
  --template-file cloudformation/template.yaml \
  --parameter-overrides file://cloudformation/parameters.json \
  --capabilities CAPABILITY_NAMED_IAM
```

Or pass parameters inline:

```bash
aws cloudformation create-stack \
  --stack-name my-lambda-role \
  --template-body file://cloudformation/template.yaml \
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

### Example: Data Pipeline Role (S3 + Glue + Athena)

```bash
aws cloudformation create-stack \
  --stack-name my-data-pipeline-role \
  --template-body file://cloudformation/template.yaml \
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

### Example: Role Reading SSM Parameters

```bash
aws cloudformation create-stack \
  --stack-name my-ssm-reader-role \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=config-reader \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=AssumeRolePrincipal,ParameterValue=lambda.amazonaws.com \
    ParameterKey=EnableSSMParameterPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### Example: CI Deployment with CI Suffix

```bash
aws cloudformation create-stack \
  --stack-name my-role-ci \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=lambda-executor \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID \
    ParameterKey=EnableLambdaPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### Stack Update

```bash
aws cloudformation update-stack \
  --stack-name my-lambda-role \
  --template-body file://cloudformation/template.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myproject \
    ParameterKey=RoleName,ParameterValue=lambda-executor \
    ParameterKey=Environment,ParameterValue=devl \
    ParameterKey=AssumeRolePrincipal,ParameterValue=lambda.amazonaws.com \
    ParameterKey=EnableCloudWatchPolicy,ParameterValue=true \
  --capabilities CAPABILITY_NAMED_IAM
```

### Validate

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
cfn-lint cloudformation/template.yaml
```

### Using as a Nested Stack

Upload the template, then reference it from a parent stack:

```bash
aws s3 cp cloudformation/template.yaml s3://your-cfn-bucket/templates/iam-role.yaml
```

```yaml
IAMRoleNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/iam-role.yaml
    Parameters:
      ProjectName: myproject
      RoleName: lambda-executor
      Environment: devl
      EnableSSMParameterPolicy: 'true'
    Tags:
      - Key: Environment
        Value: devl

Outputs:
  RoleArn:
    Value: !GetAtt IAMRoleNestedStack.Outputs.RoleArn
```

## Role Naming Convention

**Without CI suffix:**

```bash
{ProjectName}-{RoleName}-{Environment}-{Region}
```

Example: `myproject-lambda-executor-devl-us-east-1`

**With CI suffix:**

```bash
{ProjectName}-{RoleName}-{Environment}-{Region}-{CiSuffix}
```

Example: `myproject-lambda-executor-devl-us-east-1-pipeline-12345`

## Best Practices Implemented

- ✅ Principle of least privilege with conditional managed policies
- ✅ Parameterized role naming with project prefix, environment, and region
- ✅ Optional CI suffix support for unique role identification in testing
- ✅ Resource ARN parameters for fine-grained access control
- ✅ CloudFormation exports for cross-stack references
- ✅ Organized policy categories (data pipeline, AI/ML, Bedrock, Parameter Store, compute, messaging, security, observability)
- ✅ Metadata documentation within CloudFormation template

## License

MIT
