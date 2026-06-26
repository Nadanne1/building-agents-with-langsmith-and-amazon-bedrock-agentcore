---
title: "At an AWS Event"
weight: 2
---


Your AWS account has been pre-provisioned with all required infrastructure: SageMaker Studio, Bedrock Knowledge Base, Lambda functions, Cognito auth, S3 bucket, and IAM permissions.

## Access Your AWS Account

1. Navigate to the Workshop Studio event URL provided by your instructor
2. Sign in and open the AWS Console from the event dashboard
3. Confirm you are in **us-east-1** (N. Virginia)

## Open SageMaker Studio

1. In the AWS Console, search for **SageMaker AI** and open it
2. Select **Studio** in the left navigation
3. Open SageMaker Studio with the `workshop-user` profile
4. In the top-left panel, select **JupyterLab**
5. Click **Run** to start the space
6. Once the space is running, click **Open** to launch JupyterLab

## Get the Workshop Code

1. Open a terminal: **File -> New -> Terminal**
2. Download and extract the workshop source code:

```bash
cd ~
git clone https://github.com/langchain-samples/langchain-aws-samples.git
cp -r langchain-aws-samples/examples/deepagents-aws-tour .
cd deepagents-aws-tour
```

4. Install dependencies and register the Jupyter kernel:

```bash
pip install uv
uv sync
source .venv/bin/activate
python -m ipykernel install --user --name deepagents --display-name "DeepAgents"
```

## Open the Notebook

1. In the JupyterLab file browser, navigate to `deepagents-aws-tour/`
2. Open `deepagents_aws_tour.ipynb`
3. When prompted for a kernel (or via the kernel selector in the top-right), choose **"DeepAgents"**


You must select the **"DeepAgents"** kernel, not the default "Python 3". The default kernel uses the SageMaker system Python which doesn't have the workshop dependencies.

## Create a LangSmith Account and API Key

You must use the AWS-region LangSmith instance: `https://aws.smith.langchain.com`. Do not use the default `smith.langchain.com` instance. The API endpoint for `.env` is `https://aws.api.smith.langchain.com`.

## Activate Workshop Access

1. Go to `https://aws.smith.langchain.com`.
2. Sign up or log in.
3. Create a new organization for this workshop. Use a recognizable name, such as `AWS Workshop Chicago June 2026 - Your Name`.
4. Open the new organization settings/details page and copy the organization ID.
5. Go to the workshop access portal: `https://d3jdipm6o3zr7o.cloudfront.net`.
6. Paste your LangSmith organization ID.
7. Enter the workshop access code provided by staff.
8. Click **Activate**.
9. Wait until the page says **Access activated** or **Access already active**.

If the portal says **Needs help**, share the activation reference shown on the page with workshop staff.

## Create an API Key

After workshop access is active:

1. Open `https://aws.smith.langchain.com`.
2. Select the organization you activated for this workshop.
3. Create or select a workspace for this workshop.
4. Navigate to **Settings -> API Keys -> Create API key**.
5. Copy the key.

## Configure `.env`

Create `.env` once:

```bash
cp .env.example .env
```

Add your LangSmith key to `.env`:

```bash
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com
LANGSMITH_API_KEY=<paste-your-aws-region-langsmith-key>
LANGSMITH_PROJECT=deepagents-aws-tour
```

Use `https://aws.api.smith.langchain.com`, not the UI URL. A key from the default `smith.langchain.com` instance will not authenticate against the AWS-region instance.

## Register the Gateway

Copy the stack name from the Workshop Studio event dashboard, replace the placeholder, and run the Gateway registration script:

```bash
export TOUR_STACK_NAME=<paste-stack-name-from-dashboard>
uv run python scripts/register_gateway.py --write-env .env
```

This creates or reuses the AgentCore Gateway, registers the Lambda MCP targets, fetches the Cognito client secret without printing it, and writes these values into `.env`:

- `BEDROCK_KB_ID`
- `AGENT_FILES_BUCKET`
- `PUBLIC_SUPPORT_DOC_KEY`
- `GATEWAY_URL`
- `COGNITO_TOKEN_URL`
- `COGNITO_CLIENT_ID`
- `COGNITO_CLIENT_SECRET`

Do not run `cp .env.example .env` again after this step, or you will overwrite the stack outputs.

## Verify

```bash
aws sts get-caller-identity
uv run python --version
uv run python -c "from dotenv import load_dotenv; import os; load_dotenv('.env'); print('region:', os.getenv('AWS_REGION')); print('kb:', bool(os.getenv('BEDROCK_KB_ID'))); print('bucket:', bool(os.getenv('AGENT_FILES_BUCKET'))); print('gateway:', bool(os.getenv('GATEWAY_URL'))); print('langsmith:', os.getenv('LANGSMITH_ENDPOINT'))"
```

Checkpoint: workshop code is available in SageMaker Studio or Workshop Studio, dependencies are installed, LangSmith workshop access is active, `.env` has your LangSmith key and stack outputs, Gateway registration has completed, and AWS credentials work in `us-east-1`.

Proceed to [Fundamentals](/10-fundamentals/).
