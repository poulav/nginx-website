# How to Serve a Load Balancer on a single server using Nginx and simple Python server

## Introduction

This project demonstrates the setup of a basic load balancer using Nginx and simple Python servers on a single server instance. It involves installing Nginx, configuring it as a load balancer, and running two Python servers on different ports to simulate backend servers.

## Prerequisites

1. A VPS server with Ubuntu 22.04 installed
2. SSH access to the VPS server
3. Basic understanding of Nginx and Python

## Steps

1. **Acquire Server and Set Up Nginx**

   a. Purchase a VPS server from DigitalOcean or another cloud provider.
   b. Create and associate SSH credentials to the server.
   c. SSH into the server using the SSH credentials.
   d. Install Nginx using the following command:
      ```bash
      sudo apt install nginx
      ```
   e. Verify Nginx installation by running the following command:
      ```bash
      systemctl status nginx
      ```
   f. (Optional) If you encounter issues like unable to start Nginx, try killing and restarting the service:
      ```bash
      sudo pkill nginx
      sudo nginx
      ```

2. **Setting up the web pages**

   a. Install git on the server:
      ```bash
      sudo apt install git
      ```
   b. Navigate to the `/var/www` directory:
      ```bash
      cd /var/www
      ```
   c. Clone the repository containing the necessary files:
      ```bash
      git clone https://github.com/poulav/nginx-website.git
      ```
   d. Alternatively, if you have your own HTML files, create two folders named `index.html` in two different directories inside `var/www`.
   e. In each of the folders with `index.html`, run the following command to start a Python server on a different port:
      ```bash
      sudo python3 -m http.server 3001
      ```
      Replace `3001` with a different port number for each server.

3. **Setting up the NGINX Server**

   a. Open the Nginx configuration file:
      ```bash
      sudo vim /etc/nginx/nginx.conf
      ```
   b. Replace the existing configuration with the following:
      ```nginx
      user nginx;
      worker_processes auto;

      error_log /var/log/nginx/error.log notice;
      pid /var/run/nginx.pid;

      events {
        worker_connections 1024;
      }

      http {
        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        # Upstream group of backend servers
        upstream backendserver {
          server 127.0.0.1:3001;
          server 127.0.0.1:3002;
        }

        # Load balancer configuration
        server {
          server_name ng.poulav.dev;

          location / {
            proxy_pass http://backendserver;
          }

          listen 443 ssl; # managed by Certbot
          ssl_certificate /etc/letsencrypt/live/ng.poulav.dev/fullchain.pem; # managed by Certbot
          ssl_certificate_key /etc/letsencrypt/live/ng.poulav.dev/privkey.pem; # managed by Certbot
          include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
          ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
        }

        # Server 1 configuration
        server {
          server_name s1.ng.poulav.dev;

          location / {
            proxy_pass https://<SERVER_IP>:3001; # Replace <SERVER_IP> with your server's IP address
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
          }

          listen 443 ssl; # managed by Certbot
          ssl_certificate /etc/letsencrypt/live/s1.ng.poulav.dev/fullchain.pem; # managed by Certbot
          ssl_certificate_key /etc/letsencrypt/live/s1.ng.poulav.dev/privkey.pem; # managed by Certbot
          include /etc/
