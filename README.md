# Deploy SAP Integration Suite - Edge Integration Cell on Amazon Web Services

[![REUSE status](https://api.reuse.software/badge/github.com/SAP-samples/btp-edge-integration-cell-aws)](https://api.reuse.software/info/github.com/SAP-samples/btp-edge-integration-cell-aws)

## Description

SAP Edge Integration Cell is a hybrid integration runtime offered as part of SAP Integration Suite, which enables you to manage APIs and run integration scenarios within your private landscape.

This repository provides comprehensive instructions for deploying SAP Edge Integration Cell on Amazon Web Services (AWS). By following the guidelines provided, you can configure the necessary AWS resources and deploy SAP Edge Integration Cell.

![EIC Architecture HA](/assets/sap/ha-mode/sap-edge-integration-cell-aws-ha.png)

For more information on Edge Integration Cell, refer to [SAP Help page](https://help.sap.com/docs/integration-suite/sap-integration-suite/what-is-sap-integration-suite-edge-integration-cell).

## Requirements

Following infrastructure are required to deploy Edge Integration Cell on AWS.

- **SAP Prerequisites**

  - SAP BTP Global Account & Subaccount.
  - SAP Integration Suite (Standard Edition or Premium Edition) Service Entitlement & Subscription.
  - (Optional) SAP Integration Suite - Edge Integration Cell service plan entitlement.

- **External Prerequisites**

  - (**Required**) [Amazon Virtual Private Cloud (VPC)](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

    - At least 2 Availability Zones are needed for a quick start (a default VPC is sufficient), or 3 Availability Zones with both public and private subnets for high availability.

  - (**Required**) Kubernetes cluster on [Amazon Elastic Kubernetes Service (EKS)](https://docs.aws.amazon.com/eks/)

    - **Kubernetes** **1.26**, **1.27**, **1.28**, **1.29** are currently supported by Edge Integration Cell.

    - **Minimum 8 CPU, 32 GiB memory, and 101GiB persistent volumes are required for each Kubernetes node**.

    - A kubeconfig with cluster admin privileges is required for initializing the edge node, enabling CRUD operations on namespaces and CRDs.

  - (**Optional; Required for HA**) A PostgreSQL database on [Amazon Relational Database Service (RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)

    - **PostgreSQL** **12** and **15 (preferred)** are currently supported by Edge Integration Cell.

    - **Minimum 1 CPU, 2GiB memory is required for the PostgreSQL database server**.

  - (**Optional; Required for HA**) A Redis data store on [Amazon ElastiCache (Redis OSS)](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)

    - **Redis** **6.x** and **7.x** are currently supported by Edge Integration Cell.

    - **Minimum 1 CPU, 1 GiB memory is required for the Redis server**.

  - (**Optional; Required for HA**) [A Load Balancer integrated with Amazon EKS cluster](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)

    - A load balancer integrated with your Kubernetes infrastructure to expose Edge Integration Cell endpoints and load balance traffic across K8s nodes and services.

For more information, please refer to [SAP Notes 3247839](https://me.sap.com/notes/3247839)

### Required Infrastructure Sizing Guidelines

Understand the key components of Edge Integration Cell, the factors that influence its performance, and the methods to calculate the size for the hardware resources required.

For detailed sizing guidelines, please refer to [Sizing Guide for Edge Integration Cell](https://help.sap.com/docs/integration-suite/sap-integration-suite/sizing-guidelines)

## Deploy SAP Edge Integration Cell on AWS

Ready to get started? Follow the steps for either a quick setup or a high availability setup below.

[**🔗 Quick Setup: Simplified steps to deploy EIC on AWS**](/1-quick-setup.md)

[**🔗 High Availability Setup: Comprehensive guide for robust EIC deployment**](/2-high-availibility-setup.md)

## Known Issues

No known issues.

## How to obtain support

[Create an issue](https://github.com/SAP-samples/btp-edge-integration-cell-aws/issues) in this repository if you find a bug or have questions about the content.

For additional support, [ask a question in SAP Community](https://answers.sap.com/questions/ask.html).

## Contributing

If you wish to contribute code, offer fixes or improvements, please send a pull request. Due to legal reasons, contributors will be asked to accept a DCO when they create the first pull request to this project. This happens in an automated fashion during the submission process. SAP uses [the standard DCO text of the Linux Foundation](https://developercertificate.org/).

## License

Copyright (c) 2024 SAP SE or an SAP affiliate company. All rights reserved. This project is licensed under the Apache Software License, version 2.0 except as noted otherwise in the [LICENSE](LICENSE) file.
