# Building Agents with LangSmith and Amazon Bedrock AgentCore

AWS Workshop Studio content for the "Deep Agents on AWS" workshop - a 90-minute hands-on session building a production customer support agent using LangChain DeepAgents and Amazon Bedrock AgentCore.

## Structure

- `content/` - Workshop Studio markdown pages
- `static/cfn/` - CloudFormation provisioning template
- `static/iam_participant_policy.json` - Participant IAM policy
- `assets/` - Downloadable workshop source code (zip)
- `.kiro/steering/` - Style guide for content authoring

## Source Code

The notebook and Python modules participants run are maintained by the LangChain team at:
https://github.com/langchain-samples/langchain-aws-samples/tree/main/examples/deepagents-aws-tour

## Deployment

Content is pushed to Workshop Studio via the `workshopstudio://` git remote. Assets are synced to S3 via `aws s3 sync`.

## License

This project is licensed under the MIT-0 License.
