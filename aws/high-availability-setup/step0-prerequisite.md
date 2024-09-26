## Pre-requisites

This guide will walk you through the prerequisites and setup process of configure required resources on AWS for the SAP Edge Integration Cell deployment

Before setting up your resources on AWS, ensure you have completed the following prerequisites:

1. **AWS Account**

   - An active AWS account is required. If you don't have one, [sign up here](https://aws.amazon.com/free/).

2. **AWS CLI**

   - Install the AWS CLI. Follow the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

3. **kubectl**

   - Install `kubectl`, the Kubernetes command-line tool. Instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

4. **eksctl**

   - Install `eksctl`, a command-line tool for Amazon EKS. Follow the installation guide [here](https://github.com/eksctl-io/eksctl?tab=readme-ov-file#installation).

5. **Required IAM permissions**

   - The AWS IAM security principal that you're using must have permissions to work with Amazon EKS IAM roles, service linked roles, AWS CloudFormation, a VPC, and related resources.
   - Make sure to assign below AWS managed policy to your AWS IAM security principal that you choose to interact with AWS CLI
     - `AmazonEC2FullAccess`
     - `AmazonElasticFileSystemFullAccess`
     - `AWSCloudFormationFullAccess`
     - `IAMFullAccess`

6. A DNS service such as [Amazon Route 53](https://aws.amazon.com/route53/), to map external IP of the Edge Node on EKS with a domain name.

## Up Next

[**🔗 Quick setup: Home**](../../1-quick-setup.md) <br/>
[**🔗 HA setup: Home**](../../2-high-availibility-setup.md)
