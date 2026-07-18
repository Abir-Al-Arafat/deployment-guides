# Step-by-Step Guide: Launching an AWS EC2 Instance

This document provides a comprehensive walkthrough for launching an Ubuntu EC2 instance on Amazon Web Services (AWS) to host backend services and dashboards.

---

## Prerequisites
* An active AWS Account.
* A local terminal environment supporting SSH (Linux, macOS, or Windows WSL/PowerShell).

---

## Step 1: Region Selection & Navigation
1. Log in to the **AWS Management Console**.
2. In the top right corner (next to your username), click the **Region dropdown** menu.
3. Select **Europe (Frankfurt) `eu-central-1`** (or your chosen region close to your target audience).
4. In the global search bar at the top, type **EC2** and select **EC2 Virtual Servers in the Cloud** to open the EC2 Dashboard.

---

## Step 2: Launching the Instance
1. On the EC2 Dashboard, click the orange **Launch instance** button.
2. Under **Name and tags**:
   * **Name:** Enter a descriptive identifier (e.g., `taxily-production-server`).

---

## Step 3: Application and OS Image (AMI)
1. Under **Application and OS Images (Amazon Machine Image)**, select **Ubuntu**.
2. In the **Amazon Machine Image (AMI)** dropdown, choose **Ubuntu Server 24.04 LTS** (64-bit x86). This provides a stable, modern foundation for Node.js/TypeScript environments.

---

## Step 4: Instance Type Selection
1. Locate the **Instance type** dropdown field.
2. If your account plan limits access to `t3.medium`, click the search icon (**Q**) or clear the "Free tier eligible" filter constraint.
3. Select **c7i-flex.large** (2 vCPUs, 4 GiB Memory) to satisfy the 4GB RAM architectural requirement, or choose **t3.small** (2 vCPUs, 2 GiB Memory) based on your tier structures.

---

## Step 5: Key Pair Generation
1. Under **Key pair (login)**, click **Create new key pair**.
2. Configure the overlay window exactly as follows:
   * **Key pair name:** `taxily`
   * **Key pair type:** `RSA`
   * **Private key file format:** `.pem` (For OpenSSH compatibility)
3. Click **Create key pair**. 
4. *Critical:* Save the downloaded `taxily.pem` file immediately to a secure local directory. AWS will not expose this key for download again.

---

## Step 6: Network and Security Settings
1. In the **Network settings** panel, click **Edit** (top right of the section) if you want customized controls, or use the quick-checkbox configuration.
2. Configure the **Firewall (security groups)** rules to explicitly allow the following traffic arrays:
   * **Rule 1 (SSH):** Port `22` | Source: `Anywhere (0.0.0.0/0)` or `My IP` (for restricted administrative access).
   * **Rule 2 (HTTP):** Port `80` | Source: `Anywhere (0.0.0.0/0)` (Required for standard web traffic and Nginx reverse proxy routing).
   * **Rule 3 (HTTPS):** Port `443` | Source: `Anywhere (0.0.0.0/0)` (Required for Let's Encrypt / Certbot SSL certificate layers).

---

## Step 7: Storage Architecture
1. Under **Configure storage**, the default root volume configures to standard parameters.
2. Modify the size parameter to **30 GiB** (the standard maximum threshold for foundational storage tiers) and verify the volume type is set to **gp3** for optimal IOPS performance.

---

## Step 8: Final Review & Launch
1. Verify all components in the right-hand **Summary** pane match your chosen configurations.
2. Click **Launch instance**.
3. Wait for the success banner to appear, then click **View all instances** at the bottom right to return to the active console matrix.
