## Deploy SAP Edge Integration Cell on AWS - Quick Setup

![Alt Text](/assets/sap/qs-mode/sap-edge-integration-cell-aws-qs.png)

Quick setup mode gives easy-to-follow instructions for quickly deploying Edge Integration Cell on AWS. It includes a series of steps that are a simplified subset of our [high availability setup guide](/2-high-availibility-setup.md), providing a more cost-effective and faster way to deploy Edge Integration Cell to a secure AWS EKS environment.

To successfully deploy SAP Edge Integration Cell on AWS, you'll need to complete a series of steps to configure the necessary infrastructure on AWS and integrate it with your SAP Integration Suite. This guide will walk you through each step for a simplified setup. You may follow the steps that are marked **Required** for a simplest setup.

**Overview:**

The deployment process consists of two primary stages:

1. **AWS Infrastructure Setup**

   This involves setting up the essential AWS services, such as Virtual Private Cloud (VPC. **Optional** if you want to use the default VPC in your region), Kubernetes clusters on Amazon EKS. These services form the backbone of your Edge Integration Cell deployment, providing the necessary compute, storage, and networking capabilities.

2. **SAP Infrastructure Setup**

   After the AWS setup is complete, you will activate the Edge Integration Cell within your SAP Integration Suite, add the necessary edge nodes, and deploy the Edge Integration Cell solution to these nodes.

**Step-by-Step Instructions:**

1. AWS Infrastructure Setup

   - [**[Required]** Pre-requisites for AWS Setup](/aws/high-availability-setup/step0-prerequisite.md)

     Ensure that all pre-requisites are met, including SAP BTP account setup, service entitlements, and AWS environment preparation.

   - [**[Optional]** Configuring VPC on AWS](/aws/high-availability-setup/step1-configure-vpc.md)

     Set up a Virtual Private Cloud (VPC) on AWS, with the necessary availability zones, subnets and security configurations to support your Edge Integration Cell deployment.

   - Kubernetes Cluster Setup on Amazon EKS

     - [**[Required]** Step 1: Configure Amazon EKS Cluster](/aws/high-availability-setup/step2-configure-eks.md)

       Create and configure an Amazon EKS cluster that meets the requirements for Edge Integration Cell deployment.

     - [**[Required]** Step 2: Configure Storage Class for Amazon EKS Cluster](/aws/high-availability-setup/step3-configure-storage-class.md)

       Set up storage classes that support dynamic provisioning for your Kubernetes workloads.

   - [**[Required]** Generate Key Pair for NLB Endpoint](/aws/high-availability-setup/step5-configure-domain-name-key-pair.md)

     Secure the communication between your SAP Integration Suite and the EKS cluster by generating and applying a key pair.

2. **SAP Edge Integration Cell Setup**

   - [**[Required]** Activate Edge Integration Cell on SAP Integration Suite](/sap/high-availability-setup/step1-activate-edge-integration-cell.md)

     Enable the Edge Integration Cell service within your SAP Integration Suite, preparing it for deployment.

   - [**[Required]** Add Edge Node and Bootstrap Amazon EKS Cluster](/sap/high-availability-setup/step2-add-edge-node.md)

     Register your Kubernetes cluster as an edge node within the SAP system and bootstrap the necessary configurations.

   - [**[Required]** Deploy Edge Integration Cell Solution](/sap/high-availability-setup/step3-deploy-edge-integration-cell-solution.md)

     Deploy the SAP Edge Integration Cell solution on your edge node, completing the integration between your AWS infrastructure and SAP Integration Suite.

## Conclusion

By following these steps, you will successfully deploy the SAP Edge Integration Cell in a highly available and scalable environment on AWS, fully integrated with your SAP Integration Suite. This setup ensures that your data remains secure within your private landscape while leveraging the powerful capabilities of SAP's hybrid integration model.
