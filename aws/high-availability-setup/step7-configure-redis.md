# Amazon ElasticCache with Redis OSS Configuration Guide

## Introduction

SAP Edge Integration Cell requires external services for managing persistence and policies. In this case, an in-memory data store used for caching is required by SAP Edge Integration Cell.

For the DEV / UAT environments, Redis services will be deployed as built-in Edge Integration Cell components.

For the PROD environment, built-in Redis data store is not highly available and scalable. You will need to configure an external Redis cluster for the PROD env.

On the AWS side, there are two AWS managed data store services could be used as external Redis data store for the SAP Edge Integration Cell:

- **Amazon ElastiCache for Redis OSS** (**preferred**)
- **Amazon MemoryDB for Redis OSS** (**alternative**)

The requirements below on the external Redis cluster of your choice must be meet:

- **Redis** versions **6.x**, **7.x** are currently supported as external data store by SAP Edge Integration Cell.
- **Minimum CPU / Memory requirements: 1 CPU / 1 GiB**
- Redis server must have **TLS/SSL enabled**.
- Server certificate must use a Subject Alternative Name (SAN) extension (type DNS Name) to be used as server identity.
- Redis connectivity and user (**with full permission**) information is required during solution deployment.

For more information about the requirements on Redis, please referring to the following SAP official documents

- [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)
- [Redis Data Store Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster#redis-data-store-requirements)

In this instruction, we will walk you through the steps of set up a Redis cluster on Amazon ElastiCache which will meet the requirements above.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create a Redis Cluster](#step-1-create-a-redis-cluster)
- [Step 2. Create a Redis User](#step-2-create-a-redis-user)
- [Step 3. Create a User Group](#step-3-create-a-user-group)
- [Step 4. Modify the Redis Cluster to Use the User](#step-4-modify-the-redis-cluster-to-use-the-user)
- [Step 5. Obtain the TLS certificates of Amazon ElastiCache](#step-5-obtain-the-tls-certificates-of-amazon-elasticache)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

Before you begin, ensure you have the following:

- Already finished instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md)
- IAM permissions to create and manage Amazon ElastiCache instance.

## Step 1. Create a Redis Cluster

1. Log in to the AWS Management Console. Navigate to the **ElastiCache** service. Click on **Get started** then choose **Redis OSS** to start a new Redis cluster.

   ![Alt Text](/assets/aws/ha-mode/create-redis-1.png)

2. In the **Cluster settings** page, do the following and then click **Create** to create your Redis cluster on Amazon ElastiCache. Wait for the status change to **Available**, then proceed with next step.

   - **Configuration**:

     - **Deployment option:** Choose **Design your own cache**.
     - **Creation method:** Choose **Easy create**.
     - **Configuration:** Choose **Demo**.
       - We choose **Demo** for demoing and cost saving purpose.
       - Please referring to [SAP Notes 3247839](https://me.sap.com/notes/3247839) and [SAP Edge Integration Cell Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines) to choose the Redis Cluster with minimum requirement on CPU and RAM.

   - **Cluster info:**

     - **Name:** Enter an unique, read-friendly name to your Amazon ElastiCache Redis instance.

   - **Connectivity:**
     - **Network type:** Choose **IPv4**.
     - **Name:** Enter an unique, read-friendly subnet group name.
     - **VPC ID:** Enter the VPC ID we created in the instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md)

   ![Alt Text](/assets/aws/ha-mode/create-redis-2.png)

   ![Alt Text](/assets/aws/ha-mode/create-redis-3.png)

## Step 2. Create a Redis User

1. In the Amazon ElastiCache console, click **User management** on the left menu.

   ![Alt Text](/assets/aws/ha-mode/create-redis-4.png)

2. Click **Create user** button to create a new Redis user.

3. In the **Create user** page, do the following, then click **Create** to create new Redis user.

   - **User settings:**

     - **User ID:** Enter an unique user ID.
     - **User name:** Enter an unique, read-friendly user name.

   - **User authentication settings:**

     - **Authentication mode:** Choose **Password(s)**.
     - **Password 1:** Set a strong password.
     - **Password 2 - optional:** Leave as empty.

   - **Access string:**
     - **Access string:** Enter `on ~* +@all` to grant full permission.

   ![Alt Text](/assets/aws/ha-mode/create-redis-5.png)

   ![Alt Text](/assets/aws/ha-mode/create-redis-6.png)

## Step 3. Create a User Group

1. In the Amazon ElastiCache console, click **User group management** on the left menu.

   ![Alt Text](/assets/aws/ha-mode/create-redis-8.png)

2. Click **Create user group** button to create a new Redis user group.

3. In the **Create user group** page, do the following, then click **Create** button to create new Redis user group. **Wait until the User Group status change to Active**.

   - **User group settings:**
     - **User group ID:** Enter an unique, read-friendly user group name.
     - **Selected users:** Choose the Redis user we just created.

   ![Alt Text](/assets/aws/ha-mode/create-redis-9.png)

## Step 4. Modify the Redis Cluster to Use the User

1. In the Amazon ElastiCache console, select your Redis cluster, then click **Modify**

   ![Alt Text](/assets/aws/ha-mode/create-redis-7.png)

2. Scroll down to the **Security** tab. Change **Access control** to **User group access control list**. Then Choose the user group we just created in the previous step.

   ![Alt Text](/assets/aws/ha-mode/create-redis-10.png)

3. Click **Preview changes** button, then click **Modify** button to apply the changes to your Redis cluster.

   ![Alt Text](/assets/aws/ha-mode/create-redis-11.png)

## Step 5. Obtain the TLS certificates of Amazon ElastiCache

1. Go to the **Amazon EC2 Dashboard**, choose **Launch instance** to create a new EC2 instance in your AWS account. **Make sure to put the EC2 instance in the same Amazon VPC which your Amazon ElastiCache Redis cluster located.**

   ![Alt Text](/assets/aws/ha-mode/create-redis-12.png)

2. In the Launch an instance page, do the following:

   - **Application and OS Images (Amazon Machine Image):**

     - Choose **Quick Start**.
     - Choose **Amazon Linux**.
     - **Amazon Machine Image (AMI)**
       - Choose **Amazon Linux 2023 AMI**

   - **Instance type:** Choose **t2.micro**.

   - **Key pair (login):**

     - Click **Create new key pair** to generate a new key pair file.
     - Once you click **Create key pair** button, the generated key pair file will be download to you local machine automatically.
     -

   - **Network settings:**
     - **VPC - required:** Choose the VPC we created in the instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md)
     - **Subnet:** Choose one of the **public subnet** inside of your VPC.
     - **Firewall (security groups):** Choose **Create security group**.

   ![Alt Text](/assets/aws/ha-mode/create-redis-13.jpeg)

   ![Alt Text](/assets/aws/ha-mode/create-redis-14.png)

   ![Alt Text](/assets/aws/ha-mode/create-redis-15.png)

   Leave rest of the fields as default, then click Launch instance button to create new EC2 instance.

3. Once your EC2 instance has been created and instance status become **Running**, open **Security** tab, then click on the **Security group**.

   ![Alt Text](/assets/aws/ha-mode/create-redis-17.png)

4. Click **Edit inbound rules** then choose **Add rule** to create a inbound rule to allow you connect to EC2 instance from your local machine.

   ![Alt Text](/assets/aws/ha-mode/create-redis-18.png)

5. In the Edit inbound rules screen, do the following:

   - **Type:** Choose **Custom TCP**.
   - **Port Range:** Enter **22**.
   - **Source:** Choose **My IP**.
   - Choose **Save rules**.

   ![Alt Text](/assets/aws/ha-mode/create-redis-19.png)

6. Open the security group of your Amazon ElastiCache Redis instance. Let's create a new inbound rules to allow the connection from your EC2 to Amazon ElastiCache. Do the following:

   - **Type:** Choose **Custom TCP**.
   - **Port Range:** Enter **6379**.
   - **Source:** Choose **Custom**.
   - **CIDR blocks:** Choose the security group of your EC2 instance.
   - Choose **Save rules**.

   ![Alt Text](/assets/aws/ha-mode/create-redis-20.png)

7. Now Go back to your EC2 instance details page. Click **Connect** to see the connection details of your EC2 instance.

   ![Alt Text](/assets/aws/ha-mode/create-redis-16.png)

8. In the **Connect to instance** page, switch to **SSH Client** tab, following the instruction to connect to your EC2 instance in your terminal.

   ![Alt Text](/assets/aws/ha-mode/create-redis-21.png)

9. Enter command below in your terminal.

   - Replace `EXAMPLE_KEYPAIR.pem` with your actual key pair file name.
   - Replace `EXAMPLE_PUBLIC_DNS` with the connection details you get in previous step.
   - Replace `REGION_CODE` with the actual region code where your EC2 and VPC located.

   ```ssh
   ssh -i "EXAMPLE_KEYPAIR.pem" ec2-user@EXAMPLE_PUBLIC_DNS.REGION_CODE.compute.amazonaws.com
   ```

   ![Alt Text](/assets/aws/ha-mode/create-redis-22.png)

10. Once you connect to the EC2 instance via SSH, execute the command below to obtain the SSL/TLS certificate of your Redis cluster in Amazon ElastiCache.

    - replace `<cluster-endpoint>:port` with your Redis cluster endpoint.

    ```ssh
    openssl s_client -connect <cluster-endpoint>:port -showcerts
    ```

11. Execute the command above will output the SSL certificate information. Look for the `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` lines, and copy the certificate content between these lines. Save those information to a text file named `redis-cert.pem`.

    ![Alt Text](/assets/aws/ha-mode/create-redis-24.png)

## Conclusion

So far we have been successfully created the Redis Cluster in the Amazon ElastiCache service. Congratulation!

Please keep below credentials / files / data handy, we will need those information later.

- **Redis Cluster Connection Endpoint**
- **Redis Cluster User Name**
- **Redis Cluster User Password**
- **SSL/TLS Certificate of Redis Cluster**

## References

- **SAP**

  - [Redis Data Store Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
  - [Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
  - [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)

- **AWS**
  - [Amazon ElastiCache (Redis OSS) User Guide](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)

## Up Next

[**🔗 HA setup: Activate EIC ➡**](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md)
