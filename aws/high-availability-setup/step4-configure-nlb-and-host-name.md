# Amazon Elastic Kubernetes Service (EKS) Cluster Network Load Balancer Configuration Guide

## Introduction

When deploying SAP Edge Integration Cell on Amazon Elastic Kubernetes Service (EKS), ensuring that your application traffic is efficiently managed and routed is critical for maintaining high availability and performance. To achieve this, a Network Load Balancer (NLB) must be configured to handle traffic distribution across the Amazon EKS cluster. The NLB will enable secure, scalable, and fault-tolerant routing of incoming requests to the appropriate Kubernetes nodes and services within your EKS environment.

This guide will walk you through the steps required to create and configure a Network Load Balancer specifically for your Amazon EKS cluster as part of the SAP Edge Integration Cell deployment. From setting up necessary IAM roles and installing the AWS Load Balancer Controller to correctly tagging your VPC subnets, each step is designed to ensure that your load balancer is properly integrated with your EKS cluster and fully operational.

By following this guide, you will establish a robust network infrastructure that supports the high demands of SAP Edge Integration Cell, ensuring that your deployment is both resilient and secure.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create IAM Role for AWS Load Balancer Controller](#step-1-create-iam-role-for-aws-load-balancer-controller-optional)
- [Step 2. Install AWS Load Balancer Controller](#step-2-install-aws-load-balancer-controller)
- [Step 3. Tagging public subnet & private subnet properly in Amazon VPC](#step-3-tagging-public-subnet--private-subnet-properly-in-amazon-vpc)
- [Conclusion](#conclusion)

## Prerequisites

Before you begin, ensure you have the following:

- Finish instruction [Amazon Elastic Kubernetes Service (EKS) Cluster Storage Configuration Guide](/aws/high-availability-mode-setup/step3-configure-storage-class.md)
- A running Kubernetes cluster on Amazon EKS
- `AWS CLI` installed and configured
- `kubectl` installed and configured to interact with your EKS cluster
- [Helm](https://helm.sh/) installed and configured
- [eksctl](https://eksctl.io/) installed and configured to interact with your EKS cluster
- IAM permissions to create and manage AWS resources

## Step 1. Create IAM Role for AWS Load Balancer Controller (Optional)

> - Note
> - You only need to create an IAM Role for the AWS Load Balancer Controller one per AWS account.
> - Check if `AmazonEKSLoadBalancerControllerRole` exists in the IAM Console.
> - If this role exists, skip to [Step 2. Install AWS Load Balancer Controller](#step-2-install-aws-load-balancer-controller)

1. Download an IAM policy for the AWS Load Balancer Controller that allows it to make calls to AWS APIs on your behalf.

   ```curl
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-1.png)

2. Create an IAM policy using the policy downloaded in the previous step.

   ```PowerShell
   aws iam create-policy \
       --policy-name AWSLoadBalancerControllerIAMPolicy \
       --policy-document file://iam_policy.json
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-2.png)

3. Create IAM Open ID Connect (OIDC) provider and associate it with you Amazon EKS Cluster using `eksctl` command showing below.

   - Replace `region-code` with your AWS region hosts yout Amazon EKS cluster
   - Replace `my-cluster` with the name of your Amazon EKS cluster

   ```PowerShell
   eksctl utils associate-iam-oidc-provider \
        --region=region-code \
        --cluster=my-cluster \
        --approve
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-3.png)

4. Create IAM Role using `eksctl` command showing below.

   - Replace `my-cluster` with the name of your Amazon EKS cluster.
   - Replace `111122223333` with your AWS account ID, and then run the command.
   - If your Amazon EKS cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace `arn:aws:` with `arn:aws-us-gov:`.

   ```PowerShell
   eksctl create iamserviceaccount \
       --cluster=my-cluster \
       --namespace=kube-system \
       --name=aws-load-balancer-controller \
       --role-name AmazonEKSLoadBalancerControllerRole \
       --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \
       --approve
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-4.png)

## Step 2. Install AWS Load Balancer Controller

In this step, we will install AWS Load Balancer Controller on your Amazon EKS cluster using [Helm](https://helm.sh/). Make sure you already install Helm in your local machine properly before start.

1. Add the `eks-charts` Helm chart repository. AWS maintains [this repository](https://github.com/aws/eks-charts) on GitHub.

   ```helm
   helm repo add eks https://aws.github.io/eks-charts
   ```

2. Update your local repo to make sure that you have the most recent charts.

   ```helm
   helm repo update eks
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-5.png)

3. Install the AWS Load Balancer Controller.

   - Replace `my-cluster` with the name of your Amazon EKS cluster.
   - In the following command, `aws-load-balancer-controller` is the Kubernetes service account that you created in a previous step.

   ```helm
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
       -n kube-system \
       --set clusterName=my-cluster \
       --set serviceAccount.create=false \
       --set serviceAccount.name=aws-load-balancer-controller
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-6.png)

4. Verify that the controller is installed.

   ```kubectl
   kubectl get deployment -n kube-system aws-load-balancer-controller
   ```

   An example output is as follows.

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-7.png)

## Step 3. Tagging public subnet & private subnet properly in Amazon VPC

In this step, we will add some necessary tag to the Public & Private Subnet in your Amazon VPC which your Amazon EKS cluster located in. With those tags, Amazon EKS cluster and AWS Load Balancer will know which subnets could be used for internal load balancers, and which subnets could be used for external load balancers.

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).

2. Navigate to the VPC that we created in the previous instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md) in the **VPC Dashboard**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-nlb-8.png)

3. In the **Resource Map** tab you could find out all of the **Public** & **Private subnets** belongs to your VPC. **Tagging all subnets** with following:

   - **Public Subnet**
     - **Key** - `kubernetes.io/role/elb`
     - **Value** - `1`
   - **Private Subnet**
     - **Key** - `kubernetes.io/role/internal-elb`
     - **Value** - `1`

   ![Public Subnet Tagging](/assets/aws/ha-mode/create-eks-nlb-9.png)

   ![Private Subnet Tagging](/assets/aws/ha-mode/create-eks-nlb-10.png)

## Conclusion

By completing this guide, you have successfully configured a Network Load Balancer (NLB) for your Amazon EKS cluster, enabling efficient traffic management and robust load balancing for your SAP Edge Integration Cell deployment. These configurations ensure that your application can handle high volumes of traffic securely and reliably, maintaining optimal performance across your Kubernetes environment.

## Up Next

[**🔗 HA setup: Configure key pair ➡**](/aws/high-availability-setup/step5-configure-domain-name-key-pair.md)
