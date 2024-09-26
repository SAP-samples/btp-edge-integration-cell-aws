# Amazon Elastic Kubernetes Service (EKS) Configuration Guide

This guide will walk you through setting up an Amazon Elastic Kubernetes Service (EKS) cluster with EBS and EFS storage classes and three node groups.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Configure IAM Role for Amazon EKS Cluster](#step-1-create-iam-role-for-amazon-eks-cluster)
3. [Create Amazon EKS Cluster](#step-2-create-an-amazon-elastic-kubernetes-service-eks-cluster)
4. [Create IAM OIDC provider](#step-3-create-an-iam-oidc-provider-for-your-cluster)
5. [Configure Communication Between Local Machine and Amazon EKS Cluster](#step-3-configure-your-computer-to-communicate-with-your-cluster)
6. [Create Nodes in Amazon EKS Cluster](#step-4-create-kubernetes-nodes-and-node-group-in-amazon-eks-cluster)

## Prerequisites

Before you begin, ensure you have the following:

- An AWS account
- Access to the AWS Management Console
- IAM permission to create roles and policies
- The `AWS CLI` installed and configured with your credentials
- `kubectl` installed on your local machine
- `eksctl` installed on your local machine.

## Step 1. Create IAM Role for Amazon EKS Cluster

Kubernetes clusters managed by Amazon EKS make calls to other AWS services on your behalf to manage the resources that you use with the service.

1. Copy the following contents to a file named `eks-cluster-role-trust-policy.json`

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "eks.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

2. Create the IAM Role by executing the below AWS CLI Command in your terminal. You could use your prefer name to replace the `myAmazonEKSClusterRole` in the command below.

   ```
   aws iam create-role \
       --role-name myAmazonEKSClusterRole \
       --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-3.png)

3. Attach the required Amazon EKS managed IAM policy to the role.

   ```
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
       --role-name myAmazonEKSClusterRole
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-4.png)

## Step 2. Create an Amazon Elastic Kubernetes Service (EKS) Cluster

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).

2. Navigate to the **EKS Dashboard** by searching for `EKS` in the AWS services search bar.

   ![Alt Text](/assets/aws/ha-mode/create-eks-1.png)

3. Click on **Add cluster** button and choose **Create** from the drop down menu.

   ![Alt Text](/assets/aws/ha-mode/create-eks-2.png)

4. Fill in the details of your EKS Cluster, then click **Next**

   - **Name:** Enter a Name for your cluster (e.g., `sap-pal-eic-cluster`)
   - **Cluster service role:** Choose the IAM role name created in the previous step.
   - **Kubernetes version:** Choose kubernetes version `1.29` or below.
   - Leave other fields as default.

   ![Alt Text](/assets/aws/ha-mode/create-eks-5.png)

   ![Alt Text](/assets/aws/ha-mode/create-eks-6.png)

5. In the **Specify networking** page, select the default VPC (for quick setup) or the VPC created in the previous step. Leave everything as default, then click **Next**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-7.png)

6. On the **Configure observability** page, choose **Next**.

7. On the **Select add-ons** page, choose **Next**. For more information on add-ons, see [Amazon EKS add-ons](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html).

8. On the **Configure selected add-ons** settings page, choose **Next**.

9. On the **Review and create** page, choose **Create**.

## Step 3. Create an IAM OIDC provider for your cluster

Run the following script to configure OIDC provider for your cluster.

```sh
    cluster_name=my-cluster
    oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
    echo $oidc_id
```

```sh
    eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

(If no output is returned, then you must create an IAM OIDC provider for your cluster by executing below script)

```sh
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

## Step 4. Configure Your Computer to Communicate With Your Cluster

In this section, you create a **kubeconfig** file for your EKS cluster. The settings in this file enable the kubectl CLI to communicate with your cluster.

1. Click on the EKS Cluster we just created, then switch to the **Access** tab.

   ![Alt Text](/assets/aws/ha-mode/create-eks-8.png)

2. Click **Create access entry** button. Then enter the IAM principle ARN, which you use to interact with aws cli on your local machine, in the IAM Principle search bar. Click **Next**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-9.png)

3. In the **Add access policy** screen, add these access policies to the access entry. Click **Next**

   - `AmazonEKSAdminPolicy`
   - `AmazonEKSClusterAdminPolicy`
   - `AmazonEKSEditPolicy`

   ![Alt Text](/assets/aws/ha-mode/create-eks-10.png)

4. Review the configuration for your access entry. If anything looks incorrect, choose **Previous** to go back through the steps and correct the error. If the configuration is correct, choose **Create**.

5. Once the cluster is created, create a file named `eks-access-policy.json` in your machine and copy the following content to it. Replace the sample ARN string under `Resource` section with the ARN string of your cluster. Cluster ARN string can be found on the cluster overview page.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": ["eks:*"],
         "Resource": [
           "arn:aws:eks:region-code:111122223333:cluster/sap-pal-eic-cluster"
         ]
       }
     ]
   }
   ```

6. Crete the following policy and attach it to your CLI user. Replace `cli-user` with your CLI user and `111122223333` with your AWS account ID.

   ```sh
       aws iam create-policy \
         --policy-name eks-access-policy \
         --policy-document file://eks-access-policy.json


       aws iam attach-user-policy \
         --user-name cli-user \
         --policy-arn arn:aws:iam::111122223333:policy/eks-access-policy
   ```

7. Create or update a `kubeconfig` file for your EKS cluster. Replace `region-code` with the AWS Region that you created your cluster in. Replace `my-cluster` with the name of your cluster.

   ```
   aws eks update-kubeconfig --region region-code --name my-cluster
   ```

   By default, the config file is created in `~/.kube` or the new cluster's configuration is added to an existing config file in `~/.kube`.

   ![Alt Text](/assets/aws/ha-mode/create-eks-11.png)

8. Test your configuration.

   ```
   kubectl get svc
   ```

   An example output is as follows.

   ![Alt Text](/assets/aws/ha-mode/create-eks-12.png)

## Step 5. Create Kubernetes Nodes and Node Group in Amazon EKS Cluster

In-order to deploy SAP Edge Integration Cell solution in the High Availability mode for production environment, we have to make sure there are **at least 3 worker nodes** in the EKS Cluster.

In this instruction, we will create **Amazon EC2 Linux managed node group** with 3 nodes.

1. Create a node IAM role and attach the required Amazon EKS IAM managed policy to it. Copy the following contents to a file named `node-role-trust-policy.json`.

   > [!NOTE]
   > The Amazon EKS node `kubelet` daemon makes calls to AWS APIs on your behalf. Nodes receive permissions for these API calls through an IAM instance profile and associated policies.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "ec2.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

2. Create the node IAM role with following `AWS CLI` command. You could use your preferred name name to replace the `myAmazonEKSNodeRole` in the command.

   ```
   aws iam create-role \
       --role-name myAmazonEKSNodeRole \
       --assume-role-policy-document file://"node-role-trust-policy.json"
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-1.png)

3. Attach the required managed IAM policies to the role with following `AWS CLI` command. Use your node role name to replace the `myAmazonEKSNodeRole` in the command.

   ```
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
       --role-name myAmazonEKSNodeRole
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
       --role-name myAmazonEKSNodeRole
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
       --role-name myAmazonEKSNodeRole
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-2.png)

4. Open the [Amazon EKS Console](https://console.aws.amazon.com/eks/home#/clusters), choose the name of the cluster that you created previously.

5. On your EKS cluster overview page, choose **Compute** tab, scroll down and choose **Add Node Group**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-3.png)

6. On the **Configure Node Group** page, fill out the form with following, then click **Next**.

   - **Name:** Enter a unique name for your managed node group (e.g., `eic-node-group`)
   - **Node IAM role name:** Choose node role that you created in a previous step (e.g., `sap-pal-eic-cluster-node-role`)

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-4.png)

7. On the **Set compute and scaling configuration** page, fill out the form with following, then click **Next**.

   - **Node group compute configuration**

     - **AMI type:** Choose `Amazon Linux 2 (AL2_x86_64)`
     - **Capacity type:** Choose `On-Demand`
     - **Instance types:** Please refer to [SAP Edge Integration Cell Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines) to find out minimum CPU & RAM requirement on worker nodes for your intended integration scenario. <br/>
       Example: **`m6i.2xlarge`** for quick setup, **`m6i.4xlarge`** for HA setup.
     - **Disk size:** **101GiB** for quick setup or **204GiB** for HA setup.

   - **Node group scaling configuration**
     - **Desired size:** `1` for quick setup or `3` for HA setup.
     - **Minimum size:** `1` for quick setup or `3` for HA setup.
     - **Maximum size:** `1` for quick setup or `4` for HA setup.

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-5.png)

   ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-6.png)

8. On the **Specify networking** page, accept the default values and choose **Next**.

9. On the **Review and create** page, review your managed node group configuration and choose **Create**.

10. After several minutes, the **Status** in the **Node Group configuration** section will change from **Creating** to **Active**. Don't continue to the next step until the status is **Active**.

    ![Alt Text](/assets/aws/ha-mode/create-eks-node-group-7.png)

## Conclusion

Your Amazon EKS cluster is now set up with three node groups. You could go ahead for the next step to configure an Amazon EBS and Amazon EFS storage class for your Amazon EKS cluster.

## References

- **SAP**

  - [SAP Edge Integration Cell - Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
  - [SAP Edge Integration Cell - Prepare Your Kubernetes Cluster](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
  - [SAP Edge Integration Cell - Prepare for Deployment on Amazon Elastic Kubernetes Service (EKS)](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-for-deployment-on-amazon-elastic-kubernetes-service-eks)

- **AWS**
  - [Getting started with Amazon EKS – AWS Management Console and AWS CLI - Prerequisites](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-prereqs)
  - [Getting started with Amazon EKS – AWS Management Console and AWS CLI - Create EKS Cluster](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-create-cluster)
  - [Getting started with Amazon EKS – AWS Management Console and AWS CLI - Create Nodes](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-launch-workers)

## Up Next

[**🔗 Quick setup: Configure storage**](/aws/high-availability-setup/step3-configure-storage-class.md) <br/>
[**🔗 HA setup: Configure storage**](/aws/high-availability-setup/step3-configure-storage-class.md)
