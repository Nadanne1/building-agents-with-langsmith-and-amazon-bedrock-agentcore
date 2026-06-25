---
title: "Cleanup"
weight: 90
---

The workshop provisions real AWS resources that can bill while they exist. Tear them down when you're done.

The Bedrock Knowledge Base uses OpenSearch Serverless behind the scenes. Confirm it is deleted after cleanup, because retained OpenSearch Serverless resources can continue billing even when idle.

## Delete Optional Hosted Deployment Resources

If you created optional hosted deployment AWS credentials, delete the access key before destroying the stack so CloudFormation can delete the IAM user cleanly:

```bash
uv run python scripts/create_deployment_user_key.py --delete-existing
```

If you completed the optional hosted deployment path in AWS LangSmith, delete that deployment in `https://aws.smith.langchain.com`:

1. Open **Deployments**.
2. Select your workshop deployment.
3. Delete the deployment.

Skip this section if you only completed the required local deploy-readiness path.

## Delete the AgentCore Gateway

The Gateway is created by `scripts/register_gateway.py`, not by CDK, so delete it separately.

List gateways:

```bash
aws bedrock-agentcore-control list-gateways
```

Delete the workshop Gateway:

```bash
aws bedrock-agentcore-control delete-gateway --gateway-identifier <gateway-id>
```

Use the Gateway ID for the `deepagents-tour-gateway` Gateway.

## Destroy the CDK Stack

Run this from the workshop project directory:

```bash
cdk destroy --force
```

This removes the CDK-managed resources:

- S3 bucket for KB data, agent files, and public docs
- Bedrock Knowledge Base resources managed by the stack
- Order Management and Issue Management Lambda functions
- Cognito User Pool and app client
- Gateway IAM role
- Attendee IAM policy
- Optional hosted deployment IAM user, if its access keys were deleted first

The S3 bucket uses `RemovalPolicy.DESTROY` with `auto_delete_objects=True`, so bucket contents are removed during stack deletion.

## Verify

Check for remaining workshop stacks:

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?contains(StackName, 'Tour')]"
```

Expected result:

```json
[]
```

Also check for remaining Gateway resources:

```bash
aws bedrock-agentcore-control list-gateways
```

Finally, confirm in the AWS console that no workshop OpenSearch Serverless collection remains for the Bedrock Knowledge Base.

Checkpoint: CDK stack destroyed, AgentCore Gateway deleted, optional hosted LangSmith deployment removed if you created one, optional deployment access key deleted if you created one, and no ongoing AWS charges from workshop resources.

Proceed to [Summary](/99-summary/).
