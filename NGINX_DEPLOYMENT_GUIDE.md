# Guide: Nginx Reverse Proxy Deployment for React/Next.js

This document details the configuration and deployment strategy for hosting multiple services (API and Dashboard) on a single EC2 instance using Nginx as a reverse proxy.

## 1. Nginx Configuration Strategy
Nginx acts as a high-performance reverse proxy that sits in front of your Node.js applications (running on ports 3000 and 5000). It handles incoming traffic, SSL termination, and forwards requests to the appropriate local service.

### Directory Structure
Nginx manages configurations in two primary directories:
* `/etc/nginx/sites-available/`: The repository for all server block configurations.
* `/etc/nginx/sites-enabled/`: Contains symbolic links to active configurations. Nginx only loads files present in this directory.

## 2. Implemented Configurations

### Backend (API)
* **File:** `/etc/nginx/sites-available/backend`
* **Target:** `api.domain.com`
* **Port:** 5000
* **Features:** SSL-enabled via Certbot, handles HTTPS redirects, and proxies requests to the Node.js backend.

### Frontend (Dashboard)
* **File:** `/etc/nginx/sites-available/dashboard`
* **Target:** `dashboard.domain.com`
* **Port:** 3000
* **Features:** Proxies traffic from the domain to the Next.js frontend application.

## 3. Deployment Workflow

To deploy or modify these services, follow this standardized workflow:

### A. Locating Configurations
If you need to view or modify an existing configuration:
1. Navigate to: `ls /etc/nginx/sites-available/`
2. View content: `sudo cat /etc/nginx/sites-available/[filename]`

### B. Adding/Updating a Service
1. **Define the Server Block:** Create a new file in `sites-available` following the standard proxy_pass template.
2. **Enable the Service:** `sudo ln -s /etc/nginx/sites-available/[filename] /etc/nginx/sites-enabled/`
3. **Validate Syntax:** Always run `sudo nginx -t` to check for errors before restarting.
4. **Apply Changes:** `sudo systemctl restart nginx`

### C. Securing with SSL
Use Certbot to automate SSL certificate issuance and Nginx configuration updates:
`sudo certbot --nginx -d [your_domain_name]`

## 4. Troubleshooting
* **Verify Proxy Target:** Check if your PM2 processes are running on the ports specified in your Nginx config: `pm2 list`
* **Check Logs:** If traffic isn't reaching your app, check the Nginx error logs: `tail -f /var/log/nginx/error.log`
