# Amazon Relational Database Service (RDS) for PostgreSQL Configuration Guide

## Introduction

SAP Edge Integration Cell requires external services for managing persistence and policies. In this case, a relational database system is required by SAP Edge Integration Cell.

For the DEV / UAT environments, an internal PostgreSQL database will be deployed as built-in Edge Integration Cell components.

For the PROD environment, the built-in PostgreSQL is not highly available and scalable. You will need to configure an external PostgreSQL DB cluster for the PROD env.

On the AWS side, we could setup an PostgreSQL DB cluster on the Amazon Relational Database Service (RDS).

The requirements below on the external PostgreSQL database must be meet:

- PostgreSQL version **12** and **15 (preferred)** are currently supported as external database by SAP Edge Integration Cell.

- **Minimum CPU / Memory requirements: 1 CPU / 2 GiB**

- PostgreSQL server must have TLS/SSL enabled.

- Server certificate must use a Subject Alternative Name (SAN) extension (type DNS Name) to be used as server identity.

- PostgreSQL connectivity and user information is required during solution deployment.

- Required user privileges: GRANT ALL on schema

For more information about the requirements on PostgreSQL database, please referring to the following SAP official documents

- [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)
- [PostgreSQL Database Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)

In this instruction, we will walk you through the steps of create PostgreSQL Database on Amazon Relational Database Service (RDS) for SAP Edge Integration Cell.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create an Amazon RDS for PostgreSQL Database Instance / Cluster](#step-1-create-an-amazon-rds-for-postgresql-database-instance--cluster)
- [Step 2. Configure the Security Group of Amazon RDS for PostgreSQL DB Instance / Cluster](#step-2-configure-the-security-group-of-amazon-rds-for-postgresql-db)
- [Step 3. Create Database in Amazon RDS for PostgreSQL](#step-3-create-database-in-amazon-rds-for-postgresql)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

Before you begin, ensure you have the following:

- Already finished instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md)
- Already finished instruction [Amazon Elastic Kubernetes Service (EKS) Configuration Guide](/aws/high-availability-mode-setup/step2-configure-eks.md)
- Download and install pgAdmin in your local machine.
  - Download: https://www.pgadmin.org/download/
- IAM permissions to create and manage RDS instances.

## Step 1. Create an Amazon RDS for PostgreSQL Database Instance / Cluster

1. Navigate to the Amazon RDS console, click **Create database** button to create a new Amazon RDS instance.

   ![Alt Text](/assets/aws/ha-mode/create-rds-1.png)

2. In the **Create database** page, do following:

   - **Choose a database creation method:** Choose `Standard create`.

   - **Engine options:**

     - Choose `PostgreSQL`.
     - **Engine Version:** Choose `PostgreSQL 15.4-R3`.

   - **Templates:**

     - For production usage, please choose `Production`, otherwise `Dev/Test`.

   - **Availability and durability:**

     - For production usage, please choose `Multi-AZ DB Cluster`, otherwise `Single DB instance`.

   - **Settings:**

     - **DB instance identifier：** Enter an unique, read-friendly DB instance name.
     - **Master username:** Choose `postgres`.
     - **Credentials management:** Choose `Self managed` and check `Auto generate password`.

   - **Instance configuration:**

     - Choose `Standard classes (includes m classes)` and make sure the instance have **minimum 1 CPU** and **2GB RAM**.

   - **Storage:**

     - Leave everything as **default**.

   - **Connectivity:**
     - **Compute resource:** Choose `Don’t connect to an EC2 compute resource`.
     - **Network type:** Choose `IPv4`.
     - **Virtual private cloud (VPC):** Choose the VPC we created in the instruction [Amazon VPC Configuration Guide](/aws/high-availability-mode-setup/step1-configure-vpc.md)
     - **Public access:** Choose `Yes`.
     - **VPC security group (firewall)** Choose `Create new`.
     - **New VPC security group name:** Enter an unique, read-friendly security group name.

   ![Alt Text](/assets/aws/ha-mode/create-rds-2.png)

   ![Alt Text](/assets/aws/ha-mode/create-rds-3.png)

   ![Alt Text](/assets/aws/ha-mode/create-rds-4.png)

   ![Alt Text](/assets/aws/ha-mode/create-rds-5.png)

   ![Alt Text](/assets/aws/ha-mode/create-rds-6.png)

   **Leave other filed as default**, then review the configurations and click on `Create database` to create your Amazon RDS for PostgreSQL DB Cluster.

3. You will be redirect back to the Amazon RDS console. Click **View Credential details** button to see your Amazon RDS cluster password. **Note down the master password, we will need it later.**

   ![Alt Text](/assets/aws/ha-mode/create-rds-7.png)

## Step 2. Configure the Security Group of Amazon RDS for PostgreSQL DB

1. Navigate to the Amazon VPC dashboard. Choose **Security Group** from the left menu, search the security group name of Amazon RDS which we gave in the previous step.

   ![Alt Text](/assets/aws/ha-mode/create-rds-8.png)

2. Let's enable the communication between Amazon EKS cluster and Amazon RDS instance by editing the inbound rules. Click `Edit inbound rules` button.

   ![Alt Text](/assets/aws/ha-mode/create-rds-9.png)

3. Let's add new inbound rules, click **Add rule** and do the following:

   - **Type:** Choose `PostgreSQL`.
   - **Source:** Choose `Custom`.
   - **CIDR blocks**
     - Search for the default Security Group of your Amazon EKS Cluster.
     - The default security group name of EKS should be in the format of `eks-cluster-sg-<your clustername>-<aws_account_id>`

   ![Alt Text](/assets/aws/ha-mode/create-rds-10.png)

   Then click Save rules to the inbound rule.

## Step 3. Create Database in Amazon RDS for PostgreSQL

1. Navigate to the Amazon RDS console. Choose the Amazon RDS DB cluster that we created, switch to the **Connectivity & security** tab.

2. Copy and save the **Endpoint** of your Amazon RDS for PostgreSQL DB cluster.

   - DB Instance
     ![Alt Text](/assets/aws/ha-mode/create-rds-11.png)

   - DB Cluster
     ![Alt Text](/assets/aws/ha-mode/create-rds-11.png)

3. Open pgAdmin in your local machine. **Right click** on the **Servers**, choose **Register**, and then choose **Server**.

   ![Alt Text](/assets/aws/ha-mode/create-rds-12.png)

4. In the Register - Server screen, do the following:

   - **General:**

     - **Name:** Enter your Amazon RDS DB cluster name here.

   - **Connection:**
     - **Host name/address:**
       - DB Instance:
         - Enter your Amazon RDS DB cluster endpoint here.
       - DB Cluster:
         - Enter the Writer endpoint here.
     - **Port:** Enter `5432`.
     - **Username:** Enter `postgres`
     - **Password:** Enter the master password your copied from Amazon RDS Console.

   ![Alt Text](/assets/aws/ha-mode/create-rds-13.png)

   ![Alt Text](/assets/aws/ha-mode/create-rds-14.png)

   Then click **Save** button to create the connection.

5. Expand the registered DB on the left menu in pgAdmin. Right click on the DB then choose Create -> Database to create a new database for SAP Edge Integration Cell deployment.

   ![Alt Text](/assets/aws/ha-mode/create-rds-16.png)

6. In the Create - Database screen, enter the name of the new database, then click **Save** to create the new Schema. The **SAP Edge Integration Cell will use this database to create required tables, and store some required entries during the deployment**.

   ![Alt Text](/assets/aws/ha-mode/create-rds-17.png)

7. Please download your Amazon RDS for PostgreSQL database instance/cluster by visited the URL below. Keep the download file in your local, we will use it later.

   - Download Amazon RDS Certification bundle
   - **Please download the certification bundle based the AWS region where your RDS instance/cluster located.**
   - https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesDownload

## Conclusion

Now you have successfully configure a new Amazon RDS for PostgreSQL Database instance/cluster in your AWS account successfully.

Please keep below credentials / files / data handy, we will need those information later.

- **Amazon RDS DB instance / cluster endpoint**.
- **Amazon RDS DB instance / cluster master user name**.
- **Amazon RDS DB instance / cluster master user password**.
- **Amazon RDS DB instance / cluster database name**.
- **Amazon RDS DB instance / cluster SSL certification bundle**.

## References

- **SAP**

  - [PostgreSQL Database Requirements](https://help.sap.com/docs/integration-suite/sap-integration-suite/prepare-your-kubernetes-cluster)
  - [Sizing Guidelines](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)
  - [Prerequisites for installing SAP Integration Suite Edge Integration Cell](https://me.sap.com/notes/3247839)

- **AWS**
  - [Amazon Relational Database Service User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
  - [Download certificate bundles for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html#UsingWithRDS.SSL.CertificatesDownload)

## Up Next

[**🔗 HA setup: Configure Redis ➡**](/aws/high-availability-setup/step7-configure-redis.md)
