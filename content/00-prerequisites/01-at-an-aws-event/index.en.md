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
wget https://ws-assets-us-east-1.s3.amazonaws.com/9b8a1f16-ad5c-415d-8640-8a773e835fce/deepagents-aws-tour.zip
unzip deepagents-aws-tour.zip -d deepagents-aws-tour
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


You must use `https://aws.smith.langchain.com/` - not the default `smith.langchain.com`. The API endpoint is `https://aws.api.smith.langchain.com` (note the `api` subdomain).


1. Go to [aws.smith.langchain.com](https://aws.smith.langchain.com/) and sign up
2. Create a workspace
3. Navigate to **Settings → API Keys → Create API key**
4. Copy the key

## Configure `.env`

```bash
cp .env.example .env
```

Add your LangSmith key to `.env`:

```bash
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://aws.api.smith.langchain.com
LANGSMITH_API_KEY=<paste-your-key>
LANGSMITH_PROJECT=deepagents-aws-tour
```

Then set the stack name and run the gateway registration script to populate the remaining values. Copy the **StackName** value from the Workshop Studio event dashboard and replace the placeholder:

```bash
export TOUR_STACK_NAME=<paste-stack-name-from-dashboard>
uv run python scripts/register_gateway.py --write-env .env
```

This creates the AgentCore Gateway, registers the Lambda MCP targets, and writes `BEDROCK_KB_ID`, `AGENT_FILES_BUCKET`, `GATEWAY_URL`, `COGNITO_TOKEN_URL`, `COGNITO_CLIENT_ID`, and `COGNITO_CLIENT_SECRET` into your `.env`.

## Verify

```bash
python --version
aws sts get-caller-identity
```


**Checkpoint:** You should have the workshop code extracted in SageMaker Studio, dependencies installed, `.env` configured with your LangSmith key and all stack outputs populated by the gateway script.


Proceed to [Fundamentals](/10-fundamentals/).
