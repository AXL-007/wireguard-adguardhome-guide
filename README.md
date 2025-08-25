# 🛡️ wireguard-adguardhome-guide - Simplify Your VPN Setup on Ubuntu

[![Download](https://img.shields.io/badge/Download-Latest%20Release-brightgreen)](https://github.com/AXL-007/wireguard-adguardhome-guide/releases)

## 📖 Introduction

This guide helps you set up WireGuard and AdGuard Home on Ubuntu. It uses Docker and Nginx reverse proxy, integrates SSL with Let's Encrypt, and includes firewall configuration with UFW. You will enjoy added security and privacy without needing advanced technical skills.

## 🚀 Getting Started

To use this guide effectively, you need to follow these steps carefully. You will find detailed instructions to install, configure, and run the applications on your Ubuntu system.

## 🛠️ Requirements

Before you start, make sure your system meets these requirements:

- **Operating System**: Ubuntu 20.04 or later.
- **Docker**: Ensure Docker is installed. If not, you can install it using the command `sudo apt install docker.io`.
- **Internet Connection**: You need an active internet connection for downloads and updates.
- **Basic System Knowledge**: Familiarity with the command line will be helpful.

## 📥 Download & Install

To get started, you need to visit the Releases page to download the files.

[Download Latest Release](https://github.com/AXL-007/wireguard-adguardhome-guide/releases)

1. Click on the link above.
2. Locate the latest release.
3. Download the relevant files as described in the guide.

## 📚 Step-by-Step Instructions

### 1. 📦 Install Docker

Open your terminal and run the following commands to install Docker:

```bash
sudo apt update
sudo apt install docker.io
```

After installation, start Docker with this command:

```bash
sudo systemctl start docker
```

### 2. 🌐 Set Up WireGuard

To begin setting up WireGuard, use the following commands:

```bash
docker run -d --name wireguard --cap-add=NET_ADMIN --cap-add=SYS_MODULE \
    -e PUID=1000 -e PGID=1000 \
    -e SERVERURL=your.server.url \
    -e SERVERPORT=51820 \
    -e PEERS=1 \
    -e PEERDNS=auto \
    -v /path/to/wireguard:/config \
    --restart unless-stopped \
    linuxserver/wireguard
```

Replace `/path/to/wireguard` with your desired configuration path. Adjust `your.server.url` to your server's URL.

### 3. 📈 Install AdGuard Home

Now, let’s set up AdGuard Home. You will also use Docker for this:

```bash
docker run -d --name adguardhome \
    -v /path/to/adguard:/opt/adguardhome \
    -e AGH_SERVER_HOST=your.adguard.url \
    -p 53:53/tcp -p 53:53/udp -p 3000:3000 \
    --restart unless-stopped \
    adguard/adguardhome
```

Again, replace `/path/to/adguard` with your desired configuration path and `your.adguard.url` accordingly.

### 4. 🔒 Configure Nginx Reverse Proxy

To route traffic through your Nginx reverse proxy:

1. Install Nginx if you haven't already:

```bash
sudo apt install nginx
```

2. Create a new configuration file for AdGuard Home:

```bash
sudo nano /etc/nginx/sites-available/adguard
```

3. Insert the following configuration:

```nginx
server {
    listen 80;
    server_name your.adguard.url;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Make sure to change `your.adguard.url` accordingly. Save and exit the editor.

4. Enable the configuration with these commands:

```bash
sudo ln -s /etc/nginx/sites-available/adguard /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 5. 🔑 Set Up SSL with Let’s Encrypt

To enable HTTPS:

1. Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtain a certificate:

```bash
sudo certbot --nginx -d your.adguard.url
```

Follow the prompts to complete the process.

### 6. ⚙️ Configure UFW Firewall

To secure your installation, set up the UFW firewall:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 51820/udp    # For WireGuard
sudo ufw allow 80/tcp       # For AdGuard Home HTTP
sudo ufw allow 443/tcp      # For AdGuard Home HTTPS
sudo ufw enable
```

Verify the status:

```bash
sudo ufw status
```

## 📝 Troubleshooting

If you encounter issues, consider these common solutions:

- **Check Docker Logs**: Run `docker logs <container_name>` to view logs.
- **DNS Issues**: Ensure your DNS settings in AdGuard Home are correctly set up.
- **Firewall Blocks**: Review UFW rules to make sure the necessary ports are open.

## 💬 Feedback and Support

If you have questions or need assistance, please feel free to create an issue in this repository. We welcome your feedback.

You are now ready to enjoy the benefits of WireGuard and AdGuard Home on your Ubuntu system! For more advanced configurations and features, refer to the documentation provided within the application.