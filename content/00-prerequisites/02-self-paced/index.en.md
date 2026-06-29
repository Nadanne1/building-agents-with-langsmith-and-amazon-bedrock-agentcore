---
title: "Self-Paced Setup"
weight: 3
---

Running in your own AWS account requires provisioning the workshop infrastructure before starting the notebook.

## Requirements

- AWS account with credentials configured for `us-east-1`
- Python 3.12 and `uv`
- Bedrock model access enabled in `us-east-1` for:
  - `us.anthropic.claude-haiku-4-5-20251001-v1:0`
  - `us.anthropic.claude-sonnet-4-6`
  - `amazon.titan-embed-text-v2:0`
- LangSmith account on the AWS-region instance: `aws.smith.langchain.com`

Additional requirements if using the CDK deploy path:
- Node.js 18+ and AWS CDK CLI (`npm install -g aws-cdk`)

## Clone, Configure, Install

```bash
git clone https://github.com/langchain-samples/langchain-aws-samples.git
cd langchain-aws-samples/examples/deepagents-aws-tour
cp .env.example .env
uv sync --python 3.12
```

Run all workshop commands from `examples/deepagents-aws-tour`, the directory that contains `pyproject.toml`, `langgraph.json`, and the notebook.

Add your LangSmith key to `.env`:

```bash
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com
LANGSMITH_API_KEY=<your-aws-region-langsmith-key>
LANGSMITH_PROJECT=deepagents-aws-tour
```

Use `https://aws.api.smith.langchain.com`, not the UI URL. A key from the default `smith.langchain.com` instance will not authenticate against the AWS-region instance.

## Provision AWS Resources

Choose one of the two options below. Both create the same infrastructure: S3 bucket, Bedrock Knowledge Base, seed docs, OpenSearch Serverless, Lambda functions, Cognito auth, Gateway invoke role, and IAM roles.

### Option A: CloudFormation (simpler, no Node.js required)

Download the template from the [workshop GitHub repo](TODO-UPDATE-TEMPLATE-DOWNLOAD-URL), then deploy:

```bash
aws cloudformation create-stack \
  --stack-name workshop-provisioning \
  --template-body file://workshop-provisioning.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1

aws cloudformation wait stack-create-complete \
  --stack-name workshop-provisioning \
  --region us-east-1
```

The stack takes 5-10 minutes to create (OpenSearch Serverless collection is the slowest resource).

### Option B: CDK (from the LangChain source repo)

Requires Node.js 18+ and CDK CLI. Install the extra CDK dependencies:

```bash
uv sync --extra cdk --python 3.12
cdk bootstrap aws://$(aws sts get-caller-identity --query Account --output text)/us-east-1
cdk deploy
```

## Register the Gateway

After either deploy option completes, create or reuse the AgentCore Gateway, register the Lambda MCP targets, and write stack outputs to `.env`:

```bash
export TOUR_STACK_NAME=workshop-provisioning
uv run python scripts/register_gateway.py --write-env .env
```

If you used CDK (Option B), replace `workshop-provisioning` with your CDK stack name.

The Knowledge Base ingestion job can take a few minutes. If the first KB query returns no matching docs, wait and retry.

## Attach the Attendee Policy

If you used CloudFormation (Option A), the SageMaker execution role is already configured with the necessary permissions. If you are running the notebook locally or from a different IAM identity, attach the participant policy from `static/iam_participant_policy.json` to your identity.

If you used CDK (Option B), the stack outputs `AttendeePolicyArn`. Attach it to the IAM identity that will run the notebook:

```bash
aws iam attach-user-policy --user-name <your-user> --policy-arn <AttendeePolicyArn>
```

## Verify

```bash
aws sts get-caller-identity
uv run python --version
uv run python -c "from dotenv import load_dotenv; import os; load_dotenv('.env'); print('region:', os.getenv('AWS_REGION')); print('kb:', bool(os.getenv('BEDROCK_KB_ID'))); print('bucket:', bool(os.getenv('AGENT_FILES_BUCKET'))); print('gateway:', bool(os.getenv('GATEWAY_URL'))); print('langsmith:', os.getenv('LANGSMITH_ENDPOINT'))"
```

Checkpoint: infrastructure deployed, Gateway registered, `.env` populated, Bedrock model access enabled, and AWS credentials work in `us-east-1`.

Proceed to [Fundamentals](/10-fundamentals/).
