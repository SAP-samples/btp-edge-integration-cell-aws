# Deploy SAP Edge Integration Cell on Amazon Elastic Kubernetes Service (EKS) - Activate Edge Integration Cell

## Introduction

SAP Edge Integration Cell is an optional hybrid runtime offered as part of SAP Integration Suite. Before starting the actual deployment, you need to activate the Edge Integration Cell option in SAP Integration Suite.

In this instruction we will walk you the process of activate the Edge Integration Cell option, and get access to the Edge Lifecycle Management UI in SAP Integration Suite step-by-step.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Activate Edge Integration Cell](#step-1-activate-edge-integration-cell)
- [Step 2. Create a Role Collection to access the Edge Lifecycle Management UI](#step-2-create-a-role-collection-to-access-the-edge-lifecycle-management-ui)
- [Conclusion](#conclusion)
- [References](#references)

## Prerequisites

- You've subscribed to **SAP Integration Suite (Standard Edition or Premium Edition)** in your SAP Business Technology Platform subaccount.
- You've assigned the **Integration_Provisioner** role collection to the BTP user that you are using.
- You've activated the **Cloud Integration** capability in the SAP Integration Suite subscription.
- You've assigned the **edge_integration_cell** service plan of **Integration Suite** to the your SAP BTP subaccount.

## Step 1. Activate Edge Integration Cell

1. To activate Edge Integration Cell, on the SAP Integration Suite home page, choose **Settings > Runtime** from the left navigation pane.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-1.png)

2. Choose **Activate**. Once the Edge Integration Cell is activated, the **URL** for accessing **Edge Lifecycle Management user interface** is displayed. You can use this user interface to perform all lifecycle management operations of Edge Integration Cell.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-2.png)

   ![Alt Text](/assets/sap/ha-mode/activate-eic-3.png)

## Step 2. Create a Role Collection to access the Edge Lifecycle Management UI

1. Open the SAP BTP cockpit, choose **Security > Role Collections** on the left menu, then click **Create** button to create a new Role Collection.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-4.png)

2. In the **Create Role Collection** pop-up screen, enter an unique, read-friendly role collection name and description.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-5.png)

3. Once the role collection created, click **Edit** button and use the search option to find the **EdgeLMAccess** role.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-6.png)

   ![Alt Text](/assets/sap/ha-mode/activate-eic-7.png)

4. Go back to SAP BTP cockpit, choose **Security > User**, then assign the new role collection to the user who need access to the Edge Integration Cell Lifecycle Management UI.

5. Open the Edge Integration Cell Lifecycle Management UI then you should see the UI as below.

   ![Alt Text](/assets/sap/ha-mode/activate-eic-8.png)

## Conclusion

Now you have successfully activate the SAP Edge Integration Cell runtime in your SAP Integration Suite subscription.

## References

- **SAP**
  - [SAP Help Portal - Activate Edge Integration Cell](https://help.sap.com/docs/integration-suite/sap-integration-suite/activate-edge-integration-cell)
  - [SAP Help Portal - Subscribing and Configuring Initial Access to SAP Integration Suite](https://help.sap.com/docs/integration-suite/sap-integration-suite/subscribing-to-integration-suite)
  - [SAP Help Portal - Activating and Managing Capabilities](https://help.sap.com/docs/integration-suite/sap-integration-suite/activating-and-managing-capabilities)

## Up Next

[**🔗 Quick setup: Add edge node ➡**](/sap/high-availability-mode-setup/step2-add-edge-node.md) <br/>
[**🔗 HA setup: Add edge node ➡**](/sap/high-availability-mode-setup/step2-add-edge-node.md)
