# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **CloudFormation template repository** that provides a reusable nested stack template for creating an IAM role with conditionally attached managed policies for common AWS use cases. The template is designed to be referenced by parent CloudFormation stacks.

**Key characteristics:**

- Nested CloudFormation template (referenced via `TemplateURL`)
- One IAM role plus one `AWS::IAM::ManagedPolicy` per capability, each enabled by a `true`/`false` parameter
- Parameterized role naming with project, role, environment, region, and optional CI suffix
- Automated semantic versioning and releases
- AWS OIDC authentication for CI/CD deployments

## Project Structure

```text
cloudformation/
├── template.yaml                  # IAM role with conditional managed policies
├── parameters.json                # Parameter values for deployment
└── stack-config.json              # Stack name plus template and parameter file paths

.github/workflows/
├── ci.yaml                        # CI build workflow (validate and deploy the template)
├── release.yaml                   # Semantic release on push to main
├── create-branch.yaml             # Auto-create feature branches from issues
├── claude.yaml                    # Claude Code workflow
├── claude-code-review.yaml        # Claude Code review workflow
├── notify.yaml                    # Slack notification workflow
└── setup-environments.yaml        # AWS environment setup

scripts/plugins/
├── release.config.js              # Semantic-release configuration
└── (other release plugins)        # Custom commit analysis, notes generation

.claude/
├── settings.json                  # Claude Code workspace settings
└── settings.local.json            # Local overrides

.devcontainer/
└── devcontainer.json              # Dev container setup (Node.js 20)

package.json                       # Dependencies: semantic-release, commitizen
README.md                          # Template documentation and usage examples
```

## Development Commands

### Install dependencies

```bash
npm ci
```

### Trigger semantic release (usually automatic on main)

```bash
npm run release
```

### Commit with conventional commit format

```bash
npx cz commit
```

Select `feat`, `fix`, or `chore` type. Only `feat` and `fix` trigger releases.

## Key Architecture Concepts

### Nested Stack Pattern

The template is referenced from parent/root CloudFormation stacks via `TemplateURL`. Parents retrieve its outputs with `!GetAtt NestedStack.Outputs.OutputKey`. The template exports `RoleArn` and `RoleName` for cross-stack references.

### Role Naming Convention

```bash
{ProjectName}-{RoleName}-{Environment}-{Region}[-{CiSuffix}]
```

Example: `myproject-lambda-executor-devl-us-east-1`

The `CiSuffix` is added only when it is non-empty (`HasCiSuffix` condition). This keeps role names unique across environments and CI runs.

### Managed Policy Naming Convention

Each managed policy has a fixed literal `ManagedPolicyName` (for example `S3ReadOnly`, `KMS`, `SSMParameterRead`). IAM requires managed policy names to be unique per account, so only one stack from this template can exist per account unless names are changed to include a prefix. Policy names are capped at 128 characters.

### Enable-Flag Pattern

Every capability follows the same four-part pattern. Keep to it when adding or changing policies:

1. **Parameter** `Enable<Name>Policy`: `Type: String`, `Default: 'false'`, `AllowedValues: ['true', 'false']`
2. **Condition** `Create<Name>Policy`: `!Equals [!Ref Enable<Name>Policy, 'true']`
3. **Resource** `<Name>Policy`: `AWS::IAM::ManagedPolicy` with `Condition`, `ManagedPolicyName`, `Roles: [!Ref IAMRole]`, and a `PolicyDocument`
4. **Metadata group**: a `ParameterGroups` entry under `AWS::CloudFormation::Interface` listing the new parameter, so it shows under a labelled group in the console

Resource ARNs come from `*Arn` parameters with wildcard defaults, or from `!Sub` with `${AWS::Partition}`, `${AWS::Region}`, `${AWS::AccountId}`, and `${ProjectName}`.

### Current Policy Groups

- **Data Pipeline:** S3 (read-only, write, read-write), Kinesis, DynamoDB (read-only, read-write), Glue, Athena
- **AI/ML Services:** Textract, Polly, Lex, Rekognition
- **Bedrock Knowledge Base:** RAG access (`BedrockAccessPolicy`)
- **Bedrock Agent Core Policies:** `EnableBedrockAgentCorePolicy`. When enabled, the role also gets the AWS managed policies `AmazonBedrockFullAccess`, `BedrockAgentCoreFullAccess`, `CloudWatchLogsFullAccess`, and `AmazonEC2ContainerRegistryReadOnly` through `ManagedPolicyArns`
- **SSM Parameter Store:** `SSMParameterRead`, read access to `/{ProjectName}` and `/{ProjectName}/*`
- **Compute & Orchestration:** Lambda, Lambda basic execution, Step Functions, ECS, CodeBuild
- **Integration & Messaging:** SQS, SNS, SES, EventBridge, API Gateway logging
- **Security & Observability:** Secrets Manager, CloudWatch Logs, X-Ray, CloudTrail
- **Cross-Account & Advanced:** Cross-account assume role, KMS, tagging

## Key Files to Understand

### `cloudformation/template.yaml`

**Purpose:** Creates an IAM role (`IAMRole`) and attaches conditional managed policies.

**Key inputs:**

- `ProjectName` (default `cfn-nested`): Name prefix, also the SSM parameter path
- `RoleName` (default `iam-role`): Role name component
- `Environment` (`devl`, `test`, `prod`): Environment label for naming and tags
- `AssumeRolePrincipal`: Service allowed to assume the role
- `CiSuffix`: Optional suffix for unique CI/CD deployments
- `Enable*Policy` flags and `*Arn` resource pattern parameters

**Key outputs:**

- `RoleArn`, `RoleName`: Exported as `{Environment}-{RoleName}-Arn` and `{Environment}-{RoleName}-Name`
- `AssumeRoleCommand`, `CloudFormationStackId`, `DeploymentInfo`

**Behavior notes:**

- Template is `CAPABILITY_NAMED_IAM` because it creates named IAM resources
- Parameter groups in `Metadata` must list every `Enable*` parameter, or it will not appear in the console grouping

### `cloudformation/parameters.json`

Parameter values for deployment. Update it when adding parameters that need a non-default value.

### `cloudformation/stack-config.json`

Stack name plus the template and parameter file paths (`templateFile`, `parameterFile`).

### `.github/workflows/ci.yaml`

**Triggered on:**

- Manual `workflow_dispatch`
- Pushes to `feature/**` and `bug/**` branches
- Pull requests to `main`

**Path filter:** Runs only when `cloudformation/**` changes (and on changes to `ci.yaml` for pushes).

**Config loading:** The `load-config` job reads `stack-config.json` to get the stack name, template file, and parameter file.

**Environment setup:**

- Reads config from GitHub environment variables: `AWS_REGION`, `AWS_ACCOUNT_ID`, `OIDC_ROLE_NAME`, `CFN_TEMPLATES_S3_BUCKET`
- Uses AWS OIDC for keyless authentication via `aws-actions/configure-aws-credentials`
- Requires GitHub environment `ci` with OIDC trust configured

### `.github/workflows/release.yaml`

**Triggered:** On push to main

**Release rules:**

- `feat:` → MINOR bump (0.1.0 → 0.2.0)
- `fix:` → PATCH bump (0.1.0 → 0.1.1)
- `BREAKING CHANGE:` → MAJOR bump (0.1.0 → 1.0.0)
- Other commits → no release

## Testing & Validation

Run these before committing template changes:

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
cfn-lint cloudformation/template.yaml
```

**Manual stack deployment:**

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-lambda-role \
  --parameter-overrides file://cloudformation/parameters.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

The CI workflow (`ci.yaml`) runs on pull requests that change `cloudformation/**`.

## AWS Credentials & Environment Variables

**GitHub environment variables required in `ci` environment:**

- `AWS_REGION`: CloudFormation deployment region
- `AWS_ACCOUNT_ID`: AWS account to deploy into
- `OIDC_ROLE_NAME`: IAM role name for OIDC trust (uses `arn:aws:iam::{ACCOUNT_ID}:role/{ROLE_NAME}`)
- `CFN_TEMPLATES_S3_BUCKET`: S3 bucket where templates are stored

**OIDC setup:** The CI workflow uses AWS OIDC for keyless auth. The GitHub OIDC provider must trust the specified role.

## Conventional Commits & Release Flow

This repo enforces conventional commits to drive semantic versioning:

```bash
npx cz commit
```

Commit types:

- `feat: add support for X` → triggers MINOR release
- `fix: correct behavior of Y` → triggers PATCH release
- `chore: update deps` → no release
- `docs: clarify README` → no release

Only commits to `main` trigger releases. Feature branches use this format but releases happen on merge to main.

## When Modifying the Template

### Adding a New Managed Policy

1. Add the `Enable<Name>Policy` parameter under `Parameters`, following the enable-flag pattern
2. Add the `Create<Name>Policy` condition under `Conditions`
3. Add the `<Name>Policy` resource under `Resources` with `Condition`, `ManagedPolicyName`, `Roles`, and `PolicyDocument`
4. Add the parameter to a `ParameterGroups` entry in `Metadata`
5. Add the parameter to `cloudformation/parameters.json` if it needs a non-default value
6. Update the matching section in `README.md`
7. Validate with `aws cloudformation validate-template` and `cfn-lint`

### Workflow

1. **Edit** `cloudformation/template.yaml`
2. **Update** `cloudformation/parameters.json` if new parameters need values
3. **Validate** locally with `aws cloudformation validate-template` and `cfn-lint`
4. **Create a PR** with a conventional commit message (e.g., `feat: add SSM parameter read policy`)
5. **CI validates and deploys** automatically on pull requests
6. **Merge to main** → release workflow creates the version tag and GitHub release

## Dev Container

Pre-configured with:

- Node.js 20
- GitHub Copilot extension

Use via VS Code: `code --remote-container-url <repo-url>`

## Current Branch

Main branch is the release branch. Feature work branches from here and merges back via PR. Branch naming follows: `{type}/CFN-{issue-number}-{slug}` (e.g., `feature/CFN-42-add-encryption`).
