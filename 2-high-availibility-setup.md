## Deploy SAP Edge Integration Cell on AWS - High Availability Setup

![Alt Text](/assets/sap/ha-mode/sap-edge-integration-cell-aws.png)

To successfully deploy SAP Edge Integration Cell on AWS, you'll need to complete a series of steps to configure the necessary infrastructure on AWS and integrate it with your SAP Integration Suite. This guide will walk you through each step, ensuring that your deployment is production-ready, secure, and scalable.

**Overview:**

The deployment process consists of two primary stages:

1. **AWS Infrastructure Setup**

   This involves setting up the essential AWS services, such as Virtual Private Cloud (VPC), Kubernetes clusters on Amazon EKS, PostgreSQL on Amazon RDS, and Redis on Amazon ElastiCache. These services form the backbone of your Edge Integration Cell deployment, providing the necessary compute, storage, and networking capabilities.

2. **SAP Infrastructure Setup**

   After the AWS setup is complete, you will activate the Edge Integration Cell within your SAP Integration Suite, add the necessary edge nodes, and deploy the Edge Integration Cell solution to these nodes.

**Step-by-Step Instructions:**

1. AWS Infrastructure Setup

   - [Pre-requisites for AWS Setup](/aws/high-availability-setup/step0-prerequisite.md)

     Ensure that all pre-requisites are met, including SAP BTP account setup, service entitlements, and AWS environment preparation.

   - [Configuring VPC on AWS](/aws/high-availability-setup/step1-configure-vpc.md)

     Set up a Virtual Private Cloud (VPC) on AWS, with the necessary availability zones, subnets and security configurations to support your Edge Integration Cell deployment.

   - Kubernetes Cluster Setup on Amazon EKS

     - [Step 1: Configure Amazon EKS Cluster](/aws/high-availability-setup/step2-configure-eks.md)

       Create and configure an Amazon EKS cluster that meets the requirements for Edge Integration Cell deployment.

     - [Step 2: Configure Storage Class for Amazon EKS Cluster](/aws/high-availability-setup/step3-configure-storage-class.md)

       Set up storage classes that support dynamic provisioning for your Kubernetes workloads.

     - [Step 3: Install AWS Load Balancer Controller](/aws/high-availability-setup/step4-configure-nlb-and-host-name.md)

       Deploy and configure the AWS Load Balancer Controller to manage traffic across your Kubernetes nodes.

     - [Step 4: Generate Key Pair for NLB Endpoint](/aws/high-availability-setup/step5-configure-domain-name-key-pair.md)

       Secure the communication between your SAP Integration Suite and the EKS cluster by generating and applying a key pair.

   - [Configure PostgreSQL database on Amazon RDS](/aws/high-availability-setup/step6-configure-rds.md)

     Set up a PostgreSQL database on Amazon RDS, ensuring it meets the performance and security requirements for Edge Integration Cell.

   - [Configure Redis Data Store on Amazon ElastiCache](/aws/high-availability-setup/step7-configure-redis.md)

     Deploy a Redis instance on Amazon ElastiCache, which will serve as the data store for asynchronous messaging within the Edge Integration Cell.

2. **SAP Edge Integration Cell Setup**

   - [Activate Edge Integration Cell on SAP Integration Suite](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md)

     Enable the Edge Integration Cell service within your SAP Integration Suite, preparing it for deployment.

   - [Add Edge Node and Bootstrap Amazon EKS Cluster](/sap/high-availability-mode-setup/step2-add-edge-node.md)

     Register your Kubernetes cluster as an edge node within the SAP system and bootstrap the necessary configurations.

   - [Deploy Edge Integration Cell Solution](/sap/high-availability-mode-setup/step3-deploy-edge-integration-cell-solution.md)

     Deploy the SAP Edge Integration Cell solution on your edge node, completing the integration between your AWS infrastructure and SAP Integration Suite.

## Conclusion

By following these steps, you will successfully deploy the SAP Edge Integration Cell in a highly available and scalable environment on AWS, fully integrated with your SAP Integration Suite. This setup ensures that your data remains secure within your private landscape while leveraging the powerful capabilities of SAP's hybrid integration model.
