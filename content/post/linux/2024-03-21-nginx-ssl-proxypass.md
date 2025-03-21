---
title: "How to configure NGINX + Certbot for SSL proxy_pass"
summary: "How to configure NGINX + Certbot for SSL proxy_pass"
date: '2025-03-21T00:00:00+00:00'
draft: false
---
### Intro
This is a short step-by-step guide on how to configure Nginx and Certbot with Let's Encrypt certificates to secure an HTTP-only website running on port 8090 behind port 443 with SSL.
Adjust your-domain.com and port if needed. Tested on Ubuntu 24.04.

### 1. Install Nginx
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

### 2. Configure Nginx Reverse Proxy
Create a new configuration file:
```bash
sudo nano /etc/nginx/sites-available/your-domain.conf
```

Add this configuration (replace `your-domain.com` with your actual domain):
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (optional)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/your-domain.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### 3. Install Certbot and Get SSL Certificate
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com
```

Follow the interactive prompts to:
1. Enter your email (for security notices)
2. Agree to the terms of service
3. Choose whether to redirect HTTP to HTTPS (recommended)

### 4. Configure Firewall (if enabled)
Allow HTTP/HTTPS traffic:
```bash
sudo ufw allow 'Nginx Full'
sudo ufw status  # Verify the rules
```

### 5. Verify Configuration
Your final Nginx config should look like this (automatically updated by Certbot):
```nginx
server {
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8090;
        # ... rest of the proxy settings ...
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = your-domain.com) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name your-domain.com;
    return 404;
}
```

### 6. Verify SSL Configuration
Check your SSL setup:
```bash
sudo certbot renew --dry-run
```

Test your website:
- Visit `https://your-domain.com` in a browser
- Check SSL certificate validity
- Use [SSL Labs Test](https://www.ssllabs.com/ssltest/)
