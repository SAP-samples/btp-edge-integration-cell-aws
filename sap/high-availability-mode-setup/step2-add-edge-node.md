# Deploy SAP Edge Integration Cell on Amazon Elastic Kubernetes Service (EKS) - Add an Edge Node

## Introduction

In the previous instruction, we've activate the Edge Integration Cell in the SAP Integration Suite, and grant user the access to the Edge Lifecycle Management UI. Well done!

Now it is the time for us to add the Edge Node in the SAP Integration Suite - Edge Integration Cell. The Edge Node is an abstraction of the hosting Kubernetes cluster, which will grant us the ability to deploy and initiate Edge Lifecycle Management.

Once the Edge Node has been added, the next step is to run the Edge Lifecycle Management Bridge to establish the connection between the Edge Lifecycle Management and the Edge Node using the Cloud Connector.

In this instruction, we will walk you through the steps of adding Edge Node into your Edge Integration Cell in SAP Integration Suite, and the steps of establish the connection between the Edge Lifecycle Management and the Edge Node.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Create technical user (P-User) in SAP BTP and Assign Role Collection](#step-1-create-technical-user-p-user-in-sap-btp-and-assign-role-collection)
- [Step 2. Create a technical user in SAP Repositories Management](#step-2-create-a-technical-user-in-sap-repositories-management)
- [Step 3. Obtain the latest Kubernetes configuration file of your Amazon EKS cluster](#step-3-obtain-the-latest-kubernetes-configuration-file-of-your-amazon-eks-cluster)
- [Step 4. Configure SSO for Logging and Monitoring Group for SAP Edge Integration Cell](#step-4-configure-sso-for-logging-and-monitoring-group-for-sap-edge-integration-cell)
- [Step 5. Add edge node in the SAP Edge Lifecycle Management UI](#step-5-add-edge-node-in-the-sap-edge-lifecycle-management-ui)
- [Step 6. Bootstrapping Kubernetes cluster on the Edge Node](#step-6-bootstrapping-kubernetes-cluster-on-the-edge-node)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

- You've finished the previous instruction [Deploy SAP Edge Integration Cell on Amazon Elastic Kubernetes Service (EKS) - Activate Edge Integration Cell](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md)
- **You've finished all required AWS instructions, and have a Amazon EKS configured handy**.
- You've the access to the Edge Lifecycle Management UI

## Step 1. Create technical user (P-User) in SAP BTP and Assign Role Collection

1. Please follow the SAP official instruction below to create a technical user.

   > Important
   >
   > Please **note down** your technical user **P-number** and **password**.

   [Creating a Technical User (P-User) Account](https://help.sap.com/docs/EDGE_LIFECYCLE_MANAGEMENT/9d5719aae5aa4d479083253ba79c23f9/edcd1a455afb4cb0b6b1b3d148256468.html)

2. Go back to the SAP BTP Subaccount. Go to **Security** -> **Users**, then click **Create** button to register your SAP BTP technical user on your BTP subaccount.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-1.png)

3. In the **Create User** pop-up screen, enter your **technical user email**, then click **Create** to add technical user to your SAP BTP subaccount.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-2.png)

4. Go to the user details page, and then assign role collections below to your technical user.

   - **Cloud Connector Administrator**
   - **Role Collection created in the previous step** of [Activate Edge Integration Cell](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md)

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-3.png)

## Step 2. Create a technical user in SAP Repositories Management

1. Please follow the SAP official instruction below to create a technical user in SAP Repositories Management.

   > Important
   >
   > Please **note down** your technical user **username** and **basic auth string**.

   [Managing Technical Users in Repository-Based Shipment Channel](https://help.sap.com/docs/RBSC/0a64be17478d4f5ba45d14ab62b0d74c/7e83dfc309834942b441fc2106c5b7f5.html?version=Cloud)

## Step 3. Obtain the latest Kubernetes configuration file of your Amazon EKS cluster

1. Let's first execute command below to install the **Kubernetes Metrics Server** to your Amazon EKS cluster.

   ```ssh
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-9.png)

2. Execute command below to obtain the updated kubeconfig file of EKS cluster.

   - Replace `region-code` with the AWS Region that you created your cluster in.
   - Replace `my-cluster` with the name of your cluster.

   ```ssh
   aws eks update-kubeconfig --region region-code --name my-cluster
   ```

   Once you executed the command successfully, it will output the directory where the kubeconfig file located on your local machine.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-4.png)

## \*\*Up Next

If you are following **Quick Setup** guide, you may skip monitoring and logging configuration and proceed to creating an edge node. <br/>
[**🔗 Quick setup: Add edge node in ELM UI ➡**](/sap/high-availability-mode-setup/step2-add-edge-node.md#step-5-add-edge-node-in-the-sap-edge-lifecycle-management-ui) <br/>

## Step 4. Configure SSO for Logging and Monitoring Group for SAP Edge Integration Cell

1. Open your **SAP BTP subaccount administration console** for Identity Authentication. Choose **User & Authorization** -> **Groups**, then click **+ Create** button to create a new user group.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-10.png)

2. In the **Create Group** pop-up screen, enter an unique name and display name for this new user group.

   > Important
   >
   > Please **note down** the **group name** your give, we will need it later.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-11.png)

3. Go to the group details page. Assign the user into this new group who will need access to see the log file of the SAP Edge Integration Cell.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-12.png)

4. Expand the **Applications & Resources** menu, choose **Applications**. Then click **+ Create** button to create a new application.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-13.png)

5. In the **Create Application** pop-up screen, do the following, then click **+ Create** button to create the new application.

   - **Display Name:** Enter a read-friendly name.
   - **Type:** Choose **Other SAP cloud solution**.
   - **Protocol Type:** Choose **OpenID Connect**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-14.png)

6. Once your application has been created, please **note down** the **application ID** of your application.

   > Important
   >
   > **Please note down the application ID from the browser URL!!**

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-15.png)

7. Now let's create a new client secret of your application. In the application details page, click **Client Authentication**, then click **Add** button under the **Secrets section**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-16.png)

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-17.png)

8. In the **Add Secret** pop-up screen, enter a proper description of the new secret. **Leave everything as default**, and click **Save** to create new client secret.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-18.png)

9. Note done the **Client ID** and **Client Secret**. We will need it in the following step.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-19.png)

10. Click **Attributes** in the application details page, then click **Add** button to add one more assertion attribute for the single sign-on.

    ![Alt Text](/assets/sap/ha-mode/add-edge-node-20.png)

11. In the add new attributes page, do the following and then click Save to create new assertion attribute for the single sign-on.

    - **Name:** Enter **groups**.
    - **Source:** Choose **Identity Directory**
    - **Value:** Choose **Groups**

    ![Alt Text](/assets/sap/ha-mode/add-edge-node-21.png)

12. Click **OpenID Connect Configuration** in the application details page. Give a **Name** to your configuration then click **Save** button to save the changes.

    ![Alt Text](/assets/sap/ha-mode/add-edge-node-32.png)

## Step 5. Add edge node in the SAP Edge Lifecycle Management UI

1. On the SAP Integration Suite homepage, choose the URL for accessing Edge Lifecycle Management UI.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-5.png)

2. Choose **Add Edge Node** to start the configuration wizard.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-6.png)

3. In the **Prerequisite Validation Procedures** screen, check all the checkboxes then click **Continue**.

4. In the **Provide Edge Node Details** screen, do following then click **Step 2**.

   - **Edge Node Name:** Enter a read-friendly name to your edge node.
   - If you are following **Quick setup** guide, **DO NOT** check the High Availability Mode.
   - If you are following **HA Setup** guide, Check **High Availability Mode**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-7.png)

5. In the **Provide SAP Credentials** screen, do following:

   - **SAP Business Technology Platform**

     - **User ID:** Enter your SAP BTP technical user P-number.
     - **Email:** Enter your SAP BTP technical email address.
     - **Password:** Enter your SAP BTP technical user password.
     - click **Test Connection**

   - **Repository-Based Shipment Channel (Container Registry)**
     - **Username:** Enter your technical user name in SAP Repositories Management.
     - **Password:** Enter your technical user password in SAP Repositories Management.
     - click **Test Connection**

   If all the connection testing pass, click **Save** and then click **Step 3**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-8.png)

6. In the **Enable Monitoring and Logging** screen, do following:

   - If you are following **Quick Setup** guide, **DO NOT** check Deploy the Monitoring and Logging components on the Edge Node check box.
   - If you are following **HA Setup** guide, Check **Deploy the Monitoring and Logging components on the Edge Node** check box.
   - **Single Sign-On Configuration**
     - **FQDN for Identity Authentication**: Enter the **host name** of your **SAP BTP subaccount administration console**.
     - **Group Name:** Enter the **user group** name that you created in [previous step](#step-4-configure-sso-for-logging-and-monitoring-group-for-sap-edge-integration-cell).
     - **Application ID:** Enter the **applicationID** that you obtained in [previous step](#step-4-configure-sso-for-logging-and-monitoring-group-for-sap-edge-integration-cell).
     - **Client ID:** Enter the Client ID of your application that created in the [previous step](#step-4-configure-sso-for-logging-and-monitoring-group-for-sap-edge-integration-cell).
     - **Client Secret:** Enter the Client Secret of your application that created in the [previous step](#step-4-configure-sso-for-logging-and-monitoring-group-for-sap-edge-integration-cell).

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-22.png)

7. In the **Enable Local Container Registry** screen, click **Step 5** directly.

8. In the **Provide HTTP Proxy Details** screen, click **Review** directly. Review your edge node configuration details in the next page, then click **Add Edge Node**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-23.png)

9. Now you are redirected to the Edge Nodes tab where the newly added Edge Node is selected and you can view its details. The status of the Edge Node is **Not Initialized**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-24.png)

## Step 6. Bootstrapping Kubernetes cluster on the Edge Node

1. Execute command below to obtain an updated kubeconfig file of your Amazon EKS cluster.

   - Replace `region-code` with the AWS Region that you created your cluster in.
   - Replace `my-cluster` with the name of your cluster.

   ```ssh
   aws eks update-kubeconfig --region region-code --name my-cluster
   ```

   Once you executed the command successfully, it will output the directory where the kubeconfig file located on your local machine.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-4.png)

2. Go back to your Edge Node details tab. Under the **Setup Cloud Connector** section, upload your EKS cluster kubeconfig file by clicking the **Upload** button, and setup a strong password. Then click **Download Bootstrapping File** button.

   > Important
   >
   > - Multiple Amazon EKS cluster ARN will be store in your kubeconfig file.
   > - Use drop down in the **Context** field to select the correct Amazon EKS Cluster.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-25.png)

3. Still under the Set Up Cloud Connector tab, click on the tab based on your local machine to download the **Edge Lifecycle Management Bridge**.

   > Important
   >
   > - Make sure to put both Edge Lifecycle Management Bridge executable and Bootstrapping File under the same directory in your local machine.

4. Open your terminal and go to the directory you used to store both Edge Lifecycle Management Bridge executable and Bootstrapping File. Execute the command to execute the Edge Lifecycle Management Bridge executable.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-26.png)

5. You are prompted to enter the **Context Password** you defined previously.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-27.png)

6. You will be ask to choose the storage class then, please **choose** the **EBS type storage class** that we created previously.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-28.png)

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-29.png)

7. Follow the instructions display in the terminal to finish the Edge Lifecycle Management. Once it finished, you should see the Status of your Edge Node will change to **Available**.

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-30.png)

   ![Alt Text](/assets/sap/ha-mode/add-edge-node-31.png)

## Conclusion

After running the Edge Lifecycle Management Bridge successfully, you have accomplished the following:

- You see the new Edge Node displayed in Edge Lifecycle Management UI.

- Additional Kubernetes resources are deployed automatically before the status of the Edge Node shows **Available**.

## References

- **SAP**
  - [Add an Edge Node](https://help.sap.com/docs/integration-suite/sap-integration-suite/add-edge-node)
  - [Run the Edge Lifecycle Management Bridge](https://help.sap.com/docs/integration-suite/sap-integration-suite/run-edge-lifecycle-management-bridge)

## Up Next

[**🔗 Quick setup: Deploy EIC ➡**](/sap/high-availability-mode-setup/step3-deploy-edge-integration-cell-solution.md) <br/>
[**🔗 HA setup: Deploy EIC ➡**](/sap/high-availability-mode-setup/step3-deploy-edge-integration-cell-solution.md)
