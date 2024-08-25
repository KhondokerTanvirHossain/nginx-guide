# NGINX Installation and Configuration Guide

This guide provides step-by-step instructions to install NGINX, configure it, and add Certbot for SSL certificates on an Ubuntu system.

## Prerequisites

Ensure your system is up-to-date:

```sh
sudo apt update
```

### Step 1: Install Nginx

1. Install the prerequisites:

    ```bash
    sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
    ```

2. Import the official NGINX signing key:

    ```bash
    curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
    ```

3. Verify the key:

    ```bash
    gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
    ```

4. Set up the apt repository for stable NGINX packages:

    ```bash
    echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
    ```

5. Set up repository pinning:

    ```bash
    echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" | sudo tee /etc/apt/preferences.d/99nginx
    ```

6. Install NGINX Open Source:

    ```bash
    sudo apt update
    sudo apt install nginx
    ```

7. Start NGINX Open Source:

    ```bash
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

8. Verify that NGINX is running:

    ```bash
    curl -I 127.0.0.1
    ```

### Step 2: Configure NGINX

1. Add configuration file:

    ```bash
    sudo vim /etc/nginx/conf.d/example.conf
    ```

2. Add server configuration:

    ```nginx
    server {
        listen 80;
        server_name your_domain.com www.your_domain.com;

        location / {
            proxy_pass http://localhost:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    ```

3. Test the NGINX configuration:

    ```bash
    sudo nginx -t
    ```

4. Reload NGINX to apply the changes:

    ```bash
    sudo systemctl reload nginx
    ```

### Step 3: Install Certbot and Obtain SSL Certificates

1. Install Certbot:

    ```bash
    sudo apt install certbot python3-certbot-nginx
    ```

2. Stop Nginx

    ```bash
    sudo systemctl stop nginx
    ```

3. Obtain an SSL certificate:

    ```bash
    sudo certbot --nginx -d your_domain.com -d www.your_domain.com
    ```

4. Follow the prompts to complete the certificate installation.

5. Verify Certbot auto-renewal:

    ```bash
    sudo certbot renew --dry-run
    ```

6. Stop everything running on 80 81 and 443 port (optional)

    ```bash
    sudo kill -9 $(sudo lsof -t -i:80 -t -i:81 -t -i:443)
    ```

7. Start Nginx:

    ```bash
    sudo systemctl start nginx
    ```

## Remove Let's Encrypt Certificates from NGINX

To remove all Let's Encrypt certificates from NGINX, follow these steps:

1. **Stop Nginx:** First, stop the NGINX service to ensure no processes are using the certificates.

    ```bash
    sudo systemctl stop nginx
    ```

2. **Remove Let's Encrypt Certificates:** Delete the Let's Encrypt certificates and related files. These are typically stored in /etc/letsencrypt/live/, /etc/letsencrypt/archive/, and /etc/letsencrypt/renewal/.

    ```bash
    sudo rm -rf /etc/letsencrypt/live/*
    sudo rm -rf /etc/letsencrypt/archive/*
    sudo rm -rf /etc/letsencrypt/renewal/*
    ```

    If you know the domain name then manually delete the existing certificate files for sso.themaxlive.com from the Let's Encrypt directories.

    ```bash
    sudo rm -rf /etc/letsencrypt/live/example.domain.com
    sudo rm -rf /etc/letsencrypt/archive/example.domain.com
    sudo rm -rf /etc/letsencrypt/renewal/example.domain.com.conf
    ```

3. **Remove Let's Encrypt Configuration from NGINX:** Open your NGINX configuration files and remove any lines related to Let's Encrypt. These lines typically include ssl_certificate, ssl_certificate_key, and include /etc/letsencrypt/options-ssl-nginx.conf;.
You can use grep to find these lines:

    ```bash
    grep -r "letsencrypt" /etc/nginx/
    ```

    Edit the files to remove the Let's Encrypt configuration. For example:

    ```bash
    sudo nano /etc/nginx/sites-available/default
    sudo nano /etc/nginx/conf.d/example.conf  
    ```

4. **Test NGINX Configuration:** After removing the Let's Encrypt configuration, test the NGINX configuration to ensure there are no syntax errors.

    ```bash
    sudo nginx -t
    ```

5. **Start NGINX:** If the configuration test is successful, start the NGINX service.

    ```bash
    sudo systemctl start nginx
    ```
