# Domain and SSL Setup Guide for SAP Edge Integration Cell

## Introduction

When deploying SAP Edge Integration Cell, ensuring seamless access and optimal traffic management is crucial. To achieve this, a Load Balancer will be configured to expose the Edge Integration Cell endpoint, effectively distributing traffic across your Kubernetes (K8s) nodes and services. A key aspect of this setup is the registration of a custom hostname, which will be associated with the Load Balancer in the Domain Name System (DNS) of your domain.

In this guide, we will walk you through the process of creating and configuring a domain name specifically for the SAP Edge Integration Cell, utilizing Amazon Route 53 as the DNS system. This configuration will enable you to expose the Edge Integration Cell endpoints securely and efficiently.

The steps outlined in this guide will help you generate the necessary private key, create a Certificate Signing Request (CSR), obtain a signed certificate, and ultimately, create a Key Pair file that will be used during the deployment of the SAP Edge Integration Cell.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1. Generate a Private Key](#step-1-generate-a-private-key)
- [Step 2. Create a Certificate Signing Request (CSR)](#step-2-create-a-certificate-signing-request-csr)
- [Step 3. Sign the Certificate Signing Request to Obtain a Certificate](#step-3-sign-the-certificate-signing-request-to-obtain-a-certificate)
- [Step 4. Generate Key Pair file](#step-4-generate-key-pair-file)
- [Conclusion](#conclusion)

## Prerequisites

Before you begin, ensure you have the following:

- Purchase an **Domain Name** from **DNS provider**. In this guide we will use the domain name provided by Amazon Route53.
- `openssl` installed on your local machine.
  - [Download and install the OpenSSL](https://openssl-library.org/source/)
- `certbot client` installed on your local machine.
  - **Windows:** Download the latest version of the Certbot installer for Windows at https://dl.eff.org/certbot-beta-installer-win32.exe. Run the installer and follow the wizard. The installer will propose a default installation directory, C:\Program Files(x86)
  - **maxOS:** execute `brew install certbot` to install the certbot client

## Step 1. Generate a Private Key

1. Use OpenSSL to generate a private key in PEM format. Run the following command in your terminal.

   ```sh
   openssl genpkey -algorithm RSA -out my-key.key -aes256
   ```

   This command generates a 2048-bit RSA private key and saves it in the file `my-key.key`. The `-aes256` option encrypts the key with AES-256, prompting you to set a passphrase. If you prefer an unencrypted key, remove the `-aes256` option.

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-1.png)

## Step 2. Create a Certificate Signing Request (CSR)

1. Once you have the private key, generate a CSR using the following command.

   You will be prompted to enter information such as your country, state, organization, common name (domain name), etc. This information will be included in your certificate

   - **Country Name:** Two-letter country code (e.g., `US`).
   - **State or Province Name:** Full name of the state or province (e.g., `California`).
   - **Locality Name:** Name of the city or locality (e.g., `San Francisco`).
   - **Organization Name:** Full legal name of your organization.
   - **Organizational Unit Name:** This field is optional and can be left blank.
   - **Common Name:** Fully qualified domain name that you purchased (e.g., `*.saptisce.com` )
     - We purchased `saptisce.com` domain name from Amazon Route53. **You should enter your domain name here**.
   - **Email Address:** Contact email address (e.g., `admin@example.com`)
   - **A challenge password:** This field is optional and can be left blank.
   - **An optional company name:** This field is optional and can be left blank.

   ```sh
   openssl req -new -key my-key.key -out my-request.csr
   ```

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-2.png)

   By correctly filling out these fields, you will create a CSR that accurately represents your organization and the domain for which you are requesting a certificate

## Step 3. Sign the Certificate Signing Request to Obtain a Certificate

1. Sign the certificate signing request with a domain bought from Amazon Route53

   **Windows (console with administrative rights might be required)**

   - replace `*.example.com` with the host name that you bought.
   - replace `your.mail@example.com` with your email address.

   ```sh
   certbot certonly --manual --preferred-challenges dns --server "https://acme-v02.api.letsencrypt.org/directory" --domain "*.example.com" --email your.mail@example.com --csr my-request.csr --no-bootstrap --agree-tos
   ```

   **macOS**

   - replace `*.example.com` with the host name that you bought.
   - replace `your.mail@example.com` with your email address.

   ```sh
   sudo certbot certonly --manual --preferred-challenges dns --server "https://acme-v02.api.letsencrypt.org/directory" --domain "*.example.com" --email your.mail@example.com --csr my-request.cs --no-bootstrap --agree-tos
   ```

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-3.png)

2. Open the full certificate chain that has been created in the previous step.

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-4.png)

3. Open a new browser tab, go to https://letsencrypt.org/certs/isrgrootx1.pem, download the certificate and copy the content of the entire ISRG Root X1 Certificate.

   Don't forget to copy the entire content including `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`

4. Paste the content of the ISRG Root X1 Certificate to the end of the created certificate chain on your local machine that you have opened. Save it as a new file, e.g., `sap_edge_inetgration_cell_dnr.pem`.

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-5.png)

## Step 4. Generate Key Pair file

1. Let's generate a Key Pair with your **private key** and **signed certification** by using the command below

   ```sh
   openssl pkcs12 -inkey my-key.key -in sap_edge_inetgration_cell_dnr.pem -export -out eicdemo.pfx
   ```

   - replace `my-key.key` to the file name which you used to store your private key in the [Step 1. Generate a Private Key](#step-1-generate-a-private-key)
   - replace `sap_edge_inetgration_cell_dnr.pem` to the file name which you used to store the full certification chain in the [Sign the Certificate Signing Request to Obtain a Certificate](#step-3-sign-the-certificate-signing-request-to-obtain-a-certificate)
   - replace `eicdemo.pfx` to any your preferred name.

   ![Alt Text](/assets/aws/ha-mode/create-dns-record-6.png)

   By correctly execute the command, you will obtain a valid Key Pair file of your domain. Keep this file and we will need to use it when we handle the edge integration cell deployment later.

## Conclusion

By following this guide, you have successfully configured a custom domain name and generated the necessary key pairs and SSL certificates for your SAP Edge Integration Cell deployment. These steps are critical for ensuring secure, reliable access to your Edge Integration Cell endpoints and effective traffic management across your Kubernetes nodes.

Please keep below credentials / files / data handy, we will need those information later.

- **Key Pair file of your Domain**.

## Up Next

[**🔗 Quick setup: Activate EIC**](/sap/high-availability-mode-setup/step1-activate-edge-integration-cell.md) <br/>
[**🔗 HA setup: Configure RDS**](/aws/high-availability-setup/step6-configure-rds.md)
