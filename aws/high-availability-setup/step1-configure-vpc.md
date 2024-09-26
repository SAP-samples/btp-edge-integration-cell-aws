# Amazon VPC Configuration Guide

This guide will walk you through the process of setting up an Amazon Virtual Private Cloud (VPC) using the AWS Management Console.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Configure AWS VPC & Subnets](#step-1-create-a-vpc)
3. [Configure Public Subnets](#step-2-configure-public-subnets)

## Prerequisites

Before you begin, ensure you have the following:

- An AWS account
- Access to the AWS Management Console
- IAM permissions to create VPC resources

## Configure AWS VPC & Subnets

### Step 1: Create a VPC

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).

2. Navigate to the **VPC Dashboard** by searching for `VPC` in the AWS services search bar.

   ![Alt Text](/assets/aws/ha-mode/search-vpc.png)

3. Click on the **Create VPC** button.

   ![Alt Text](/assets/aws/ha-mode/create-vpc-1.png)

4. Fill in the details of your VPC and then Click **Create VPC**:

   - **Resources to create:** Choose `VPC and more`
   - **Name tag:** Enter a name for your VPC (e.g., `sap-pal-eic`).
   - **IPv4 CIDR block:** Enter a CIDR block (e.g., `10.0.0.0/16`).
   - **IPv6 CIDR block:** Leave this as `No IPv6 CIDR block` unless you need IPv6.
   - **Tenancy:** Choose `Default`.
   - **Number of Availability Zones (AZs):** `3` `[2 for quick setup]`
   - **Number of public subnets**: `3` `[2 for quick setup]`
   - **Number of private subnets**: `3` `[2 for quick setup]`
   - **NAT gateways ($):** Choose `1 per AZ`
   - **VPC endpoints:** Choose `None`

   ![Alt Text](/assets/aws/ha-mode/create-vpc-2.png)

   ![Alt Text](/assets/aws/ha-mode/create-vpc-3.png)

5. Wait for couple of seconds, then you will see the VPC and all related resources will be created.

   ![Alt Text](/assets/aws/ha-mode/create-vpc-4.png)

### Step 2: Configure Public Subnets

1. Click `Redirect` icon for the first **Public Subnet** displaying in the Resource map tab. It will open a new tab and redirect you to the Manage Subnet screen.

   ![Alt Text](/assets/aws/ha-mode/config-subnet-1.png)

2. In the new screen, click **Actions** button and choose **Edit subnet settings** from drop down menu.

   ![Alt Text](/assets/aws/ha-mode/config-subnet-2.png)

3. Under the **Auto-assign IP settings** section, check **Enable auto-assign public IPv4 address** and click **Save** to apply the changes.

   ![Alt Text](/assets/aws/ha-mode/config-subnet-3.png)

4. **Apply the same process to all of the public subnet in your created VPC**.

### Conclusion

You have successfully configured an AWS VPC with public and private subnets. You can now Amazon EKS Cluster within this VPC and configure them according to your requirements.

For more information, refer to the [AWS VPC documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).

### Up Next

[**🔗 Quick setup: Configure EKS ➡**](/aws/high-availability-setup/step2-configure-eks.md) <br/>
[**🔗 HA setup: Configure EKS ➡**](/aws/high-availability-setup/step2-configure-eks.md)
