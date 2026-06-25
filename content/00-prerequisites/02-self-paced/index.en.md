---
title: "Self-Paced Setup"
weight: 3
---

Running in your own AWS account requires provisioning the workshop infrastructure before starting the notebook.

## Requirements

- AWS account with credentials configured for `us-east-1`
- Python 3.12 and `uv`
- Node.js 18+ and AWS CDK CLI (`npm install -g aws-cdk`)
- Bedrock model access enabled in `us-east-1` for:
  - `us.anthropic.claude-haiku-4-5-20251001-v1:0`
  - `us.anthropic.claude-sonnet-4-6`
  - `amazon.titan-embed-text-v2:0`
- LangSmith account on the AWS-region instance: `aws.smith.langchain.com`

## Clone, Configure, Install

```bash
git clone https://github.com/langchain-samples/langchain-aws-samples.git
cd langchain-aws-samples/examples/deepagents-aws-tour
cp .env.example .env
uv sync --extra cdk --python 3.12
```

Run all workshop commands from `examples/deepagents-aws-tour`, the directory that contains `pyproject.toml`, `langgraph.json`, and `cdk_preprovision.py`.

Add your LangSmith key to `.env`:

```bash
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com
LANGSMITH_API_KEY=<your-aws-region-langsmith-key>
LANGSMITH_PROJECT=deepagents-aws-tour
```

Use `https://aws.api.smith.langchain.com`, not the UI URL. A key from the default `smith.langchain.com` instance will not authenticate against the AWS-region instance.

## Provision AWS Resources

The CDK stack creates the S3 bucket, Bedrock Knowledge Base, seed docs, Lambdas, Cognito auth, Gateway invoke role, and attendee IAM policy.

```bash
cdk bootstrap aws://$(aws sts get-caller-identity --query Account --output text)/us-east-1
cdk deploy
```

After deploy, create or reuse the AgentCore Gateway, register the Lambda MCP targets, and write stack outputs to `.env`:

```bash
uv run python scripts/register_gateway.py --write-env .env
```

The Knowledge Base ingestion job can take a few minutes. If the first KB query returns no matching docs, wait and retry.

## Attach the Attendee Policy

The CDK stack outputs `AttendeePolicyArn`. Attach it to the IAM identity that will run the notebook: user, role, or SSO permission set.

For an IAM user:

```bash
aws iam attach-user-policy --user-name <your-user> --policy-arn <AttendeePolicyArn>
```

## Verify

```bash
aws sts get-caller-identity
uv run python --version
uv run python -c "from dotenv import load_dotenv; import os; load_dotenv('.env'); print('region:', os.getenv('AWS_REGION')); print('kb:', bool(os.getenv('BEDROCK_KB_ID'))); print('bucket:', bool(os.getenv('AGENT_FILES_BUCKET'))); print('gateway:', bool(os.getenv('GATEWAY_URL'))); print('langsmith:', os.getenv('LANGSMITH_ENDPOINT'))"
```

Checkpoint: CDK stack deployed, Gateway registered, `.env` populated, Bedrock model access enabled, attendee identity has the emitted policy, and AWS credentials work in `us-east-1`.

Proceed to [Fundamentals](/10-fundamentals/).
