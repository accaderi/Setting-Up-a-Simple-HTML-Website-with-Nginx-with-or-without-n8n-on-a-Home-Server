<h1 align="center"><strong>Setting Up a Simple HTML Website with Nginx (with or without n8n) on a Home Server</strong></h1>

<!-- <p align="center">
  <img src="screenshot.jpg" alt="Screenshot">
</p> -->

<p align="center">
  <a href="https://youtu.be/I8EesgZEyIg">
    <img src="https://img.youtube.com/vi/I8EesgZEyIg/0.jpg" alt="Youtube Video">
  </a>
</p>

<p align="center">
  <a href="https://youtu.be/I8EesgZEyIg">Hosting n8n on Home Server</a>
</p>

This guide walks you through the steps to set up a simple HTML website on your home server using Nginx. You can use this guide to either:
- Set up a basic HTML site alone, or
- Set up a website alongside n8n for automation.

The steps will help you create, configure, and serve the website on your home server.

## 1. Create a Directory for Your Website

First, create a directory that will hold the files for your website (e.g., `your-domain.com`):

```bash
sudo mkdir -p /var/www/html/your-domain.com
```

Create the Index File
Next, create an index.html file in the new directory to serve as the homepage:

```bash
sudo nano /var/www/html/your-domain.com/index.html
```

Add the following sample HTML content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to My Website</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: linear-gradient(135deg, #6e8efb, #a777e3);
            color: white;
            text-align: center;
        }

        .container {
            background: rgba(255, 255, 255, 0.1);
            padding: 2rem;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
            max-width: 80%;
        }

        h1 {
            font-size: 3rem;
            margin-bottom: 1rem;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        p {
            font-size: 1.2rem;
            line-height: 1.6;
            margin: 1rem 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello & Welcome!</h1>
        <p>This website is proudly hosted on my home server.</p>
    </div>
</body>
</html>
```
Save and exit the editor (Ctrl + X, then press Y to confirm).

## 3. Configure Nginx
### 3.1 Forward Port 80 on Your Router
To make your website accessible from outside your home network, forward port 80 on your router:

Log into your router's web interface.
Navigate to **WAN > Virtual Server / Port Forwarding**.
Create a new port forwarding rule:
- **Service Name**: Your preferred name
- **Protocol**: TCP
- **External Port**: 80
- **Internal Port**: Same as the external port (80)
- **Internal IP Address**: The internal IP of your home server (use ifconfig to check)
- **Source IP**: Empty

### 3.2 Modify Nginx Configuration
Edit the Nginx configuration to serve your website and n8n:

```bash
sudo nano /etc/nginx/sites-available/n8n.conf
```
Add the following server blocks:

```nginx
server {
    listen 80;
    server_name your-site.com;

    root /var/www/html/your-site.com;  # Adjust this path as needed
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    server_name your-site.com;

    root /var/www/html/your-site.com;  # Adjust this path as needed
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# The following server block is for n8n. If n8n is not installed on your server, remove this block.
server {
    listen 443 ssl;
    server_name your-n8n-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-n8n-domain.com/fullchain.pem;  # Update with your n8n certificate path
    ssl_certificate_key /etc/letsencrypt/live/your-n8n-domain.com/privkey.pem;  # Update with your n8n key path

    location / {
        proxy_pass http://localhost:5678;  # n8n runs on port 5678 by default
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;  # Ensure the Host header is set correctly
        proxy_cache_bypass $http_upgrade;  # Bypass cache for upgrades
    }
}

```

### 3.3 Test Nginx Configuration

Test the Nginx configuration for syntax errors:

```bash
sudo nginx -t
```
### 3.4 Restart Nginx
Restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```
## 4. Obtain SSL Certificate
To secure your website with SSL, use Certbot to obtain and install the certificate for your domain:

```bash
sudo certbot --nginx -d your-domain.com
```

This will automatically configure SSL for your site.

Now, your website should be accessible via http://your-domain.com and https://your-domain.com, while n8n is accessible through https://your-n8n-domain.com.

**Note:** If you do not require HTTP (port 80) for your website and only want to serve it over HTTPS (port 443), you can remove the `server { listen 80; ...}` part from the Nginx configuration.