---
title: "Cleanup"
weight: 90
---


The workshop provisions real AWS resources that bill while they exist. Tear everything down when you're done.


The OpenSearch Serverless collection behind the Bedrock Knowledge Base bills continuously even when idle. Don't leave it running.


## Destroy the CDK Stack

```bash
cdk destroy --force
```

This removes:
- S3 bucket (KB data + agent files + public docs)
- Bedrock Knowledge Base + OpenSearch Serverless collection
- Order Management and Issue Management Lambda functions
- Cognito User Pool and app client
- Gateway IAM role
- Attendee IAM policy

The stack uses `RemovalPolicy.DESTROY` with `auto_delete_objects`, so bucket contents are removed cleanly.

## Delete the AgentCore Gateway

The Gateway is created by `register_gateway.py`, not CDK, so delete it separately:

```bash
aws bedrock-agentcore-control delete-gateway --gateway-identifier <gw-id>
```

## Delete the LangSmith Deployment

1. Go to [aws.smith.langchain.com](https://aws.smith.langchain.com/)
2. Navigate to **Deployments** → your deployment → **Delete**

## Verify

```bash
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?contains(StackName, 'Tour')]"
```

Should return an empty list.


**Checkpoint:** CDK stack destroyed, Gateway deleted, LangSmith deployment removed. No ongoing AWS charges from workshop resources.

