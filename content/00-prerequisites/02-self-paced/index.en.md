---
title: "Self-Paced Setup"
weight: 3
---


Running on your own account requires provisioning infrastructure before starting the notebook.

## Requirements

- AWS account with credentials configured for **us-east-1**
- Python 3.11+ and [uv](https://docs.astral.sh/uv/getting-started/installation/)
- Node.js 18+ and AWS CDK CLI (`npm install -g aws-cdk`)
- Bedrock model access enabled for Claude Haiku 4.5, Claude Sonnet 4.6, and Titan Embeddings v2

## Clone and Install

```bash
git clone https://github.com/srimanthtangedipalli-eng/deepagents-aws-tour.git
cd deepagents-aws-tour
uv sync
uv sync --extra cdk
source .venv/bin/activate
```

## Provision AWS Resources

The CDK stack deploys the S3 bucket, Knowledge Base, Lambdas, Cognito, and IAM policy in one command:

```bash
cdk bootstrap aws://$(aws sts get-caller-identity --query Account --output text)/us-east-1
cdk deploy
```

After deploy, register the Gateway and write outputs to `.env`:

```bash
uv run python scripts/register_gateway.py --write-env .env
```

## Configure LangSmith

```bash
cp .env.example .env
```

Add your LangSmith key (from [aws.smith.langchain.com](https://aws.smith.langchain.com/)):

```bash
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com
LANGSMITH_API_KEY=<your-key>
LANGSMITH_PROJECT=deepagents-aws-tour
```


Use the API endpoint `https://aws.api.smith.langchain.com`, not the UI URL. A key from the default `smith.langchain.com` will not authenticate against the AWS-region instance.


## Attach the Attendee Policy

The CDK stack outputs an `AttendeePolicyArn`. Attach it to the IAM identity running the notebook:

```bash
aws iam attach-user-policy --user-name <your-user> --policy-arn <AttendeePolicyArn>
```

## Verify

```bash
aws configure get region
python --version
aws sts get-caller-identity
```


**Checkpoint:** CDK stack deployed, Gateway registered, `.env` populated with all required values, and AWS credentials working in us-east-1.


Proceed to [Fundamentals](/10-fundamentals/).
