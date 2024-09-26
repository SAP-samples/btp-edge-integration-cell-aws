# Amazon Elastic Kubernetes Service (EKS) Cluster Storage Configuration Guide

This guide will walk you through setting up **storage classes** in Amazon EKS cluster by using **Amazon Elastic Block Store** and **Amazon Elastic File System**.

## Introduction

SAP Edge Integration Cell runs on compute clusters managed by the Kubernetes. In-order to successfully deploy the Edge Integration Cell runtime into Kubernetes, Edge Integration Cell requires Kubernetes cluster has two kinds of persistent storage with ability to dynamic provisioning:

- [Required] Persistence volume with **ReadWriteOnce (RWO)** access mode.
- [Optional] Persistence volume with **ReadWriteMany (RWX)** access mode. This is optional for Quick Setup.
- **ReadWriteOnce (RWO)**
  - Only one instance can mount the EBS volume at a time, allowing that instance to read and write data.
- **ReadWriteMany (RWX)**
  - Multiple instances can mount the file system simultaneously, allowing them to read and write to the file system concurrently.

To fulfill these storage requirements, we leverage **Amazon Elastic Block Store (EBS)** and **Amazon Elastic File System (EFS)** in our Amazon EKS (Elastic Kubernetes Service) cluster.

**[Required] Amazon Elastic Block Store (EBS)** volume are designed to provide high-performance block storage that can be attached to a **single EC2 instance** at a time in read-write mode. This nature makes Amazon EBS to be a good choice for the K8S cluster with storage access mode as ReadWriteOnce (RWO).

**[Optional] Amazon Elastic File System (EFS)** is a scalable, fully-managed file system that can be mounted by **multiple EC2 instances** or Kubernetes pods concurrently. This makes it ideal for shared storage scenarios, where multiple pods need read/write access to the same data. This nature makes Amazon EFS to be a good choice for the K8S cluster with storage access mode as ReadWriteMany (RWX).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create IAM Role for Amazon EBS CSI driver](#step-1-create-an-iam-role-for-amazon-ebs-csi-driver)
- [Step 2. Install the Amazon EBS CSI driver](#step-2-install-the-amazon-ebs-csi-driver-via-amazon-eks-add-on)
- [Step 3. Create Amazon EBS type storage class](#step-3-create-amazon-ebs-type-storage-class)
- [Step 4. Create an IAM role for Amazon EFS CSI driver](#step-4-create-an-iam-role-for-amazon-efs-csi-driver)
- [Step 5. Install the Amazon EFS CSI driver](#step-5-install-the-amazon-efs-csi-driver-via-amazon-eks-add-on)
- [Step 6. Create Amazon EFS file system for Amazon EKS cluster](#step-6-create-amazon-efs-file-system-for-amazon-eks-cluster)
- [Step 7. Create Amazon EFS type storage class](#step-7-create-amazon-efs-type-storage-class)

## Prerequisites

Before you begin, ensure you have the following:

- Finish instruction [Amazon Elastic Kubernetes Service (EKS) Configuration Guide](/aws/high-availability-mode-setup/step2-configure-eks.md)
- A running Kubernetes cluster on Amazon EKS
- `kubectl` installed and configured.
- `AWS CLI` installed and configured.
- IAM permissions for creating EBS and EFS resources.

## Step 1. Create an IAM role for Amazon EBS CSI driver

1. View your EKS cluster's OIDC provider URL by using below `AWS ClI` Command. Replace `my-cluster` with the name of the EKS cluster created in the previous step.

   ```
   aws eks describe-cluster \
       --name my-cluster \
       --query "cluster.identity.oidc.issuer" \
       --output text
   ```

   An example output is as follows.

   ```
   https://oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-1.png)

2. Copy the following contents to a file named `aws-ebs-csi-driver-trust-policy.json`. Replace `111122223333` with your AWS account ID. Replace `EXAMPLED539D4633E53DE1B71EXAMPLE` and `region-code` with the values returned in the previous step. If your cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace `arn:aws:` with `arn:aws-us-gov:`.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
         },
         "Action": "sts:AssumeRoleWithWebIdentity",
         "Condition": {
           "StringEquals": {
             "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
             "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
           }
         }
       }
     ]
   }
   ```

3. Create the IAM role for the EBS CSI Driver. You can change `AmazonEKS_EBS_CSI_DriverRole` to a different name. If you change it, make sure to change it in later steps.

   ```
   aws iam create-role \
       --role-name AmazonEKS_EBS_CSI_DriverRole \
       --assume-role-policy-document file://"aws-ebs-csi-driver-trust-policy.json"
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-2.png)

4. Attach the required AWS managed policy to the role with the following command. If your cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace `arn:aws:` with `arn:aws-us-gov:`

   ```
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
       --role-name AmazonEKS_EBS_CSI_DriverRole
   ```

## Step 2. Install the Amazon EBS CSI driver via Amazon EKS add-on

1. Go back to the AWS EkS console and select the EKS cluster we created previously. On your EKS cluster overview page, choose **Add-ons**, then click **Get more add-ons**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-3.png)

2. On the **Select add-ons** page, do the following:

   - In the **Amazon EKS-addons** section, select the **Amazon EBS CSI Driver** check box.
   - Choose **Next**

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-4.png)

3. On the **Configure selected add-ons settings** page, do the following:

   - Select the **Version** you'd like to use.
   - For **Select IAM role**, select the name of an IAM role that we created in the [Step 1](#step-1-create-an-amazon-ebs-csi-driver-iam-role)
   - Choose **Next**

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-5.png)

4. On the **Review and add** page, choose **Create**. After the add-on installation is complete, you see your installed add-on.

## Step 3. Create Amazon EBS type storage class

1. Copy the following contents to a file named `ebs-storageclass.yaml`. It will create a storage class called `ebs-sc` which will be used for the Persistent Volume Claim with `ReadWriteOnce` access mode.

   ```yaml
   ---
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: ebs-sc
     annotations:
       storageclass.kubernetes.io/is-default-class: "true"
   provisioner: ebs.csi.aws.com
   volumeBindingMode: WaitForFirstConsumer
   allowVolumeExpansion: true
   ```

2. Deploy the `ebs-sc` storage class by using the following `kubectl` command

   ```kubectl
   kubectl apply -f ebs-storageclass.yaml
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-6.png)

3. Execute the follow command to unset the storage class `gp2` as default.

   ```powershell
   kubectl patch storageclass gp2 -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "false"}}}'
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-7.png)

4. Now we've configure the Amazon EBS type storage class successfully, and allow EKS cluster able to dynamically provision persistent volumes for the access mode `ReadWriteOnce` by using the persistence volume claim we created.

## \*\*Up Next

If you are following the **Quick Setup** guide, you may proceed to the next step to create a key pair. EFS is optional for quick setup. <br/>
[**🔗 Quick setup: Configure key pair**](/aws/high-availability-setup/step5-configure-domain-name-key-pair.md)

## Step 4. Create an IAM role for Amazon EFS CSI driver

1. View your EKS cluster's OIDC provider URL by using below `AWS ClI` Command. Replace `my-cluster` with the name of the EKS cluster created in the previous step.

   ```
   aws eks describe-cluster \
       --name my-cluster \
       --query "cluster.identity.oidc.issuer" \
       --output text
   ```

   An example output is as follows.

   ```
   https://oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-1.png)

2. Copy the following contents to a file named `aws-efs-csi-driver-trust-policy.json`. Replace `111122223333` with your AWS account ID. Replace `EXAMPLED539D4633E53DE1B71EXAMPLE` and `region-code` with the values returned in the previous step. If your cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace `arn:aws:` with `arn:aws-us-gov:`.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
         },
         "Action": "sts:AssumeRoleWithWebIdentity",
         "Condition": {
           "StringLike": {
             "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:efs-csi-*",
             "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com"
           }
         }
       }
     ]
   }
   ```

3. Create the IAM role for the EFS CSI Driver. You can change `AmazonEKS_EFS_CSI_DriverRole` to a different name. If you change it, make sure to change it in later steps.

   ```PowerShell
   aws iam create-role \
       --role-name AmazonEKS_EFS_CSI_DriverRole \
       --assume-role-policy-document file://"aws-efs-csi-driver-trust-policy.json"
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-8.png)

4. Attach the required AWS managed policy to the role with the following command. If your cluster is in the AWS GovCloud (US-East) or AWS GovCloud (US-West) AWS Regions, then replace `arn:aws:` with `arn:aws-us-gov:`.

   ```PowerShell
   aws iam attach-role-policy \
       --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEFSCSIDriverPolicy \
       --role-name AmazonEKS_EFS_CSI_DriverRole
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-9.png)

5. **Now we've created IAM role for Amazon EFS CSI driver successfully.**

## Step 5. Install the Amazon EFS CSI driver via Amazon EKS add-on

1. Go back to the AWS EKS console and select the EKS cluster we created previously. On your EKS cluster overview page, choose **Add-ons**, then click **Get more add-ons**.

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-3.png)

2. On the **Select add-ons** page, do the following:

   - In the **Amazon EKS-addons** section, select the **Amazon EFS CSI Driver** check box.
   - Choose **Next**

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-10.png)

3. On the **Configure selected add-ons settings** page, do the following:

   - Select the **Version** you'd like to use.
   - For **Select IAM role**, select the name of an IAM role that we created in the [Step 4](#step-4-create-an-iam-role-for-amazon-efs-csi-driver)
   - Choose **Next**

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-11.png)

4. On the **Review and add** page, choose **Create**. After the add-on installation is complete, you see your installed add-on.

## Step 6. Create Amazon EFS file system for Amazon EKS cluster

Please follow the [AWS official step-by-step guide](https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/docs/efs-create-filesystem.md) to create a Amazon EFS file system for your EKS cluster.

## Step 7. Create Amazon EFS type storage class

1. Execute `AWS ClI` command below to retrieve the Amazon EFS file system ID that we created in the [Step 6. Create Amazon EFS file system for Amazon EKS cluster](#step-6-create-amazon-efs-file-system-for-amazon-eks-cluster)

   You could also find your Amazon EFS file system ID in the Amazon EFS console.

   ```PowerShell
   aws efs describe-file-systems --query "FileSystems[*].FileSystemId" --output text
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-12.png)

2. Copy the following contents to a file named `efs-storageclass.yaml`. Replace `fs-EXAMPLEEFSFILESYSID` with your Amazon EFS file system ID obtained in the previous step. It will create a storage class called `efs-sc` and referring to the Amazon EFS file system for us.

   ```YAML
   kind: StorageClass
   apiVersion: storage.k8s.io/v1
   metadata:
       name: efs-sc
   provisioner: efs.csi.aws.com
   volumeBindingMode: Immediate
   parameters:
       provisioningMode: efs-ap # The type of volume to be provisioned by Amazon EFS. Currently, only access point based provisioning is supported (efs-ap)
       fileSystemId: fs-EXAMPLEEFSFILESYSID
       directoryPerms: "700" # The directory permissions of the root directory created by the access point.
       gidRangeStart: "1000" # The starting range of the Posix group ID to be applied onto the root directory of the access point. The default value is 50000
       gidRangeEnd: "2000" # The ending range of the Posix group ID. The default value is 7000000
       basePath: "/dynamic_provisioning"
       subPathPattern: "${.PVC.namespace}/${.PVC.name}"
       ensureUniqueDirectory: "true"
       reuseAccessPoint: "false"
   allowVolumeExpansion: true
   ```

3. Deploy the `efs-sc` storage class by using the following `kubectl` command.

   ```kubectl
   kubectl apply -f efs-storageclass.yaml
   ```

   ![Alt Text](/assets/aws/ha-mode/create-eks-storage-class-13.png)

4. Now we've configure the Amazon EFS type storage class successfully. This storage class will enable Amazon EKS Cluster dynamically provision persistent volumes with the access mode `ReadWriteMany`, and allow volume expansion in general.

## Conclusion

Your Amazon EKS cluster is now set up Amazon EBS and Amazon EFS type storage class. You could go ahead for the next step.

## References

- **SAP**

  - [SAP Edge Integration Cell - Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
  - [Plan Your Setup of Edge Integration Cell](https://help.sap.com/docs/integration-suite/sap-integration-suite/plan-your-setup-of-edge-integration-cell)
  - [Prepare Your Kubernetes Cluster](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster#kubernetes-cluster-requirements)

- **AWS**
  - [Use Amazon EBS storage](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
  - [Use Amazon EFS storage](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)

## Up Next

[**🔗 Quick setup: Configure key pair**](/aws/high-availability-setup/step5-configure-domain-name-key-pair.md) <br/>
[**🔗 HA setup: Configure load balancer**](/aws/high-availability-setup/step4-configure-nlb-and-host-name.md)
