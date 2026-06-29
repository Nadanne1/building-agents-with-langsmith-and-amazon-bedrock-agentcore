---
title: "Cleanup"
weight: 90
---

## At an AWS Event

If you ran this workshop at an AWS-hosted event using a Workshop Studio provisioned account, no cleanup is required. The sandbox account and all resources in it are automatically reclaimed when the event ends.

## Self-Paced (Your Own Account)

If you ran this workshop in your own AWS account, follow these steps to avoid ongoing charges.

### Estimated Costs

If you complete the workshop in a single session (~90 minutes) and delete all resources immediately after, the estimated cost is approximately **$5-10 USD**. The primary cost drivers are:

| Service | Cost Driver | Estimate |
|---------|-------------|----------|
| OpenSearch Serverless | Minimum 2 OCUs while collection exists (~$0.24/hr each) | ~$4-7 for 90 min |
| Amazon Bedrock (Claude) | Token usage across notebook cells | ~$1-3 |
| SageMaker Studio | ml.t3.medium instance (~$0.05/hr) | ~$0.10 |
| Lambda, Cognito, S3 | Minimal usage | < $0.01 |
| AgentCore (Code Interpreter, Browser) | Per-session charges | ~$0.50 |

The OpenSearch Serverless collection is the most expensive resource and bills continuously while it exists. Delete resources promptly after completing the workshop.

Pricing links:
- [Amazon OpenSearch Serverless Pricing](https://aws.amazon.com/opensearch-service/pricing/)
- [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Amazon SageMaker Pricing](https://aws.amazon.com/sagemaker/pricing/)
- [Amazon Bedrock AgentCore Pricing](https://aws.amazon.com/bedrock/agentcore/pricing/)

### Step 1: Delete the AgentCore Gateway

The Gateway was created by `register_gateway.py` and is not part of the CloudFormation stack. Delete it manually:

```bash
GATEWAY_ID=$(aws bedrock-agentcore-control list-gateways --region us-east-1 --query "items[?name=='deepagents-tour-gateway'].gatewayId" --output text)

# Delete targets first
for TARGET in $(aws bedrock-agentcore-control list-gateway-targets --gateway-identifier $GATEWAY_ID --region us-east-1 --query "items[*].targetId" --output text); do
  aws bedrock-agentcore-control delete-gateway-target --gateway-identifier $GATEWAY_ID --target-id $TARGET --region us-east-1
done

# Wait for targets to delete, then delete gateway
sleep 10
aws bedrock-agentcore-control delete-gateway --gateway-identifier $GATEWAY_ID --region us-east-1
```

### Step 2: Delete the CloudFormation Stack

This removes all provisioned resources: SageMaker Studio, Knowledge Base, OpenSearch Serverless collection, S3 bucket, Lambda functions, Cognito, and IAM roles.

If you used the Workshop Studio provisioned account, the stack name is `workshop-provisioning`. If you deployed with CDK in the self-paced path, use `cdk destroy` instead or find your stack name with `aws cloudformation list-stacks`.

```bash
aws cloudformation delete-stack --stack-name workshop-provisioning --region us-east-1
```

The OpenSearch Serverless collection takes 5-10 minutes to delete. Monitor progress:

```bash
aws cloudformation wait stack-delete-complete --stack-name workshop-provisioning --region us-east-1
```

### Step 3: Delete LangSmith Deployment (if created)

If you deployed to LangSmith Deployment in Part 7:

1. Go to [aws.smith.langchain.com](https://aws.smith.langchain.com/)
2. Navigate to **Deployments**
3. Find your deployment and delete it

### Resources Created by This Workshop

| Resource | Service | Deleted By |
|----------|---------|------------|
| SageMaker Studio Domain + Space | SageMaker | CFN stack delete |
| OpenSearch Serverless collection | OpenSearch | CFN stack delete |
| Bedrock Knowledge Base | Bedrock | CFN stack delete |
| S3 bucket (KB docs + agent files) | S3 | CFN stack delete |
| Order Management Lambda | Lambda | CFN stack delete |
| Issue Management Lambda | Lambda | CFN stack delete |
| Cognito User Pool | Cognito | CFN stack delete |
| IAM roles (SageMaker, KB, Gateway) | IAM | CFN stack delete |
| AgentCore Gateway + targets | AgentCore | Manual (Step 1) |
| LangSmith Deployment (optional) | LangSmith | Manual (Step 3) |
