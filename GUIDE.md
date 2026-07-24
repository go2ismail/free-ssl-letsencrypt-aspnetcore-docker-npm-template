# 🚀 Nginx Proxy Manager (NPM) with Let's Encrypt SSL — Complete Guide

> **Target Audience:** .NET Developers who are NOT familiar with Linux/Docker commands
> **Goal:** Replace manual nginx config files with Nginx Proxy Manager GUI, and auto-provision free SSL certificates from Let's Encrypt

---

## 📋 Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Clone This Repository](#2-clone-this-repository)
3. [Stop Existing Nginx / Apache (If Any)](#3-stop-existing-nginx--apache-if-any)
4. [Start Nginx Proxy Manager](#4-start-nginx-proxy-manager)
5. [Access the NPM Admin Panel](#5-access-the-npm-admin-panel)
6. [Add Your First Proxy Host (Example)](#6-add-your-first-proxy-host-example)
7. [Obtain Free SSL Certificate from Let's Encrypt](#7-obtain-free-ssl-certificate-from-lets-encrypt)
8. [Seamless Integration with Existing Containers](#8-seamless-integration-with-existing-containers)
9. [Verification](#9-verification)
10. [Useful Docker Commands](#10-useful-docker-commands)
11. [Troubleshooting](#11-troubleshooting)

---

## 1️⃣ Prerequisites

Before starting, make sure you have these ready:

### ✅ 1.1 Docker & Docker Compose Installed

Check if Docker is installed:

```bash
docker --version
```

Expected output (example):
```
Docker version 24.0.7, build afdd53b
```

If not installed, run:

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group (so you don't need 'sudo' for docker commands)
sudo usermod -aG docker $USER

# Apply group change (or logout & login again)
newgrp docker

# Verify
docker --version
```

### ✅ 1.2 Docker Compose Installed

Docker Compose is usually included with Docker Desktop. For Linux, install it separately:

```bash
docker compose version
```

Expected output:
```
Docker Compose version v2.x.x
```

If not installed:

```bash
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

### ✅ 1.3 Domain Names Pointing to Your Server

You need at least one domain (e.g., `app1.yourdomain.com`) pointing to your server's public IP address.

To verify DNS is set correctly:

```bash
# Replace with your domain
nslookup app1.yourdomain.com
```

Or use `dig`:

```bash
dig +short app1.yourdomain.com
```

The result should show your server's public IP address.

### ✅ 1.4 Ports 80 and 443 Must Be Free

Your VPS must have **ports 80 and 443** available (not used by another program).

Check if ports 80 and 443 are in use:

```bash
sudo ss -tulpn | grep -E ':(80|443)\s'
```

If there's output showing nginx or apache listening on those ports, you need to stop them (see Step 3).

### ✅ 1.5 Open Ports in Firewall

If your VPS has a firewall (like UFW), allow these ports:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 81/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

Check firewall status:

```bash
sudo ufw status
```

---

## 2️⃣ Clone This Repository

SSH into your VPS, then clone this repo:

```bash
# SSH into your VPS first (from your local machine)
ssh your_user@your_server_ip
```

Then run:

```bash
# Clone the repository
git clone https://github.com/go2ismail/free-ssl-letsencrypt-aspnetcore-docker-npm-template.git

# Enter the directory
cd free-ssl-letsencrypt-aspnetcore-docker-npm-template
```

List files to confirm:

```bash
ls -la
```

You should see:
```
docker-compose-npm.yml
GUIDE.md
```

---

## 3️⃣ Stop Existing Nginx / Apache (If Any)

If you already have nginx, Apache, or any other web server running on ports 80/443, stop them.

### Stop Nginx:

```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
sudo systemctl status nginx   # Should show "inactive (dead)"
```

### Stop Apache:

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
sudo systemctl status apache2   # Should show "inactive (dead)"
```

### If You Also Have Certbot or Let's Encrypt Manual:

Remove certbot auto-renew cron jobs (optional, but recommended to avoid conflicts):

```bash
sudo certbot delete
```

> **Note:** Nginx Proxy Manager handles SSL certificates itself, so you don't need certbot anymore.

---

## 4️⃣ Start Nginx Proxy Manager

Run the following command from the repository directory:

```bash
docker compose -f docker-compose-npm.yml up -d
```

What this command does:
- `docker compose` — Docker Compose tool
- `-f docker-compose-npm.yml` — Use this specific compose file
- `up` — Create and start containers
- `-d` — Run in detached mode (background)

### Check if NPM is running:

```bash
docker ps
```

You should see something like:

```
CONTAINER ID   IMAGE                           STATUS         PORTS                                                                                             NAMES
abc123def456   jc21/nginx-proxy-manager:latest   Up 2 minutes   0.0.0.0:80->80/tcp, 0.0.0.0:81->81/tcp, 0.0.0.0:443->443/tcp   nginx-proxy-manager
```

### Check logs (if something is wrong):

```bash
docker logs nginx-proxy-manager
```

---

## 5️⃣ Access the NPM Admin Panel

Open your browser and go to:

```
http://YOUR_SERVER_IP:81
```

> Replace `YOUR_SERVER_IP` with your VPS's actual IP address (e.g., `http://123.45.67.89:81`)

### Default Login Credentials:

| Field    | Value                 |
|----------|-----------------------|
| Email    | `admin@example.com`   |
| Password | `changeme`            |

### On First Login:

1. Enter the default credentials
2. You will be **forced to change your email and password** immediately
3. Set a new email and a strong password
4. Save

---

## 6️⃣ Add Your First Proxy Host (Example)

### Scenario:

You have an existing ASP.NET Core Docker container running on port 5000:

```
http://YOUR_SERVER_IP:5000
```

Now you want to access it via a domain with SSL:

```
https://app1.yourdomain.com
```

### Steps in NPM Admin Panel:

1. Click **"Proxy Hosts"** in the left sidebar
2. Click the **"Add Proxy Host"** button (blue button, top-right)

   ![Add Proxy Host button](https://i.imgur.com/placeholder.png)

3. Fill in the form:

   | Tab/Field           | Value                           |
   |---------------------|---------------------------------|
   | **Domain Names**    | `app1.yourdomain.com`           |
   | **Scheme**          | `http`                          |
   | **Forward Hostname / IP** | Your server's IP address (e.g., `123.45.67.89`) |
   | **Forward Port**    | `5000` (or `8000`, `8080`, etc.) |
   | **Cache Assets**    | ❌ Unchecked                    |
   | **Block Common Exploits** | ✅ Checked               |
   | **Websocket Support** | ✅ Checked (if your app uses SignalR/WebSockets) |

   > **IMPORTANT:** If you've connected your existing container to the `npm_network` (see Step 8), you can use the **container name** instead of IP address. For example: `aspnet-app-1` instead of `123.45.67.89`.

4. Click **"Save"**

### Repeat for Other Apps:

Create additional proxy hosts for your other apps:

| Your App | Current Access        | New Domain                    |
|----------|-----------------------|-------------------------------|
| App 1    | `http://IP:5000`      | `https://app1.yourdomain.com` |
| App 2    | `http://IP:8000`      | `https://app2.yourdomain.com` |
| App 3    | `http://IP:8080`      | `https://app3.yourdomain.com` |

---

## 7️⃣ Obtain Free SSL Certificate from Let's Encrypt

After saving the proxy host, you can add SSL:

1. Click the **three dots** (⋮) next to the proxy host you just created
2. Click **"Edit"**
3. Go to the **"SSL"** tab
4. Select **"Request a new SSL Certificate"**
5. Choose:
   - **"Force SSL"** ✅ — automatically redirect HTTP to HTTPS
   - **"HTTP-01 Challenge"** (default)
   - Enter your **email address** (for Let's Encrypt renewal notices)
   - Check **"I agree to the Let's Encrypt Terms of Service"**
6. Click **"Save"**

### What happens next:

- NPM will contact Let's Encrypt
- It will create a temporary file on port 80 for domain verification
- If DNS is correctly pointed to your server, the certificate will be issued
- You'll see a green "SSL" badge next to your proxy host

### Result:

Your ASP.NET Core app is now accessible at:

```
https://app1.yourdomain.com
```

And HTTP will automatically redirect to HTTPS.

---

## 8️⃣ Seamless Integration with Existing Containers

Your existing ASP.NET Core Docker containers currently run on:

```
http://YOUR_SERVER_IP:5000
http://YOUR_SERVER_IP:8000
http://YOUR_SERVER_IP:8080
```

There are **two ways** NPM can forward traffic to them:

### Option A: Use Host IP (Simplest — Works Immediately)

In the NPM Proxy Host form, set:

| Field              | Value                    |
|--------------------|--------------------------|
| Forward Hostname   | Your server's IP (e.g., `123.45.67.89`) |
| Forward Port       | `5000` (or 8000, 8080)   |

**Pros:**
- Works immediately without changing anything
- No need to modify existing containers

**Cons:**
- Traffic goes through the host network stack (slightly slower)

### Option B: Use Internal Docker Network (Better Performance)

Connect your existing containers to `npm_network` so NPM can access them directly via container names.

## How to Connect Existing Containers to npm_network

### Step 8.1: Find Your Existing Container Names

```bash
docker ps
```

Example output:
```
CONTAINER ID   IMAGE                              PORTS                    NAMES
xyz789abc      mcr.microsoft.com/dotnet/aspnet    0.0.0.0:5000->80/tcp    my-aspnet-app-1
def456ghi      mcr.microsoft.com/dotnet/aspnet    0.0.0.0:8000->80/tcp    my-aspnet-app-2
jkl012mno      mcr.microsoft.com/dotnet/aspnet    0.0.0.0:8080->80/tcp    my-aspnet-app-3
```

### Step 8.2: Connect Each Container to npm_network

```bash
docker network connect npm_network my-aspnet-app-1
docker network connect npm_network my-aspnet-app-2
docker network connect npm_network my-aspnet-app-3
```

### Step 8.3: Update NPM Proxy Hosts

Now edit each proxy host and change:

| Field            | Before                 | After              |
|------------------|------------------------|--------------------|
| Forward Hostname | `123.45.67.89`         | `my-aspnet-app-1`  |
| Forward Port     | `5000`                 | `80`               |

> **Why port 80 now?** Because internally, the ASP.NET Core container listens on port 80. The port mapping `5000:80` only applies to external access via the host IP.

### Step 8.4: (Optional) Make It Permanent

If your containers are defined in a `docker-compose.yml`, add the network to their compose file:

```yaml
# In your ASP.NET Core project's docker-compose.yml
services:
  my-aspnet-app-1:
    # ... your existing config ...
    networks:
      - npm_network

networks:
  npm_network:
    external: true
    name: npm_network
```

Then recreate the containers:

```bash
docker compose down
docker compose up -d
```

---

## 9️⃣ Verification

### 9.1 Check if NPM is running:

```bash
docker ps | grep nginx-proxy-manager
```

### 9.2 Check the network:

```bash
docker network inspect npm_network
```

You should see all containers connected to this network listed under "Containers".

### 9.3 Test SSL Certificate:

Using curl:

```bash
curl -I https://app1.yourdomain.com
```

Expected output:
```
HTTP/2 200
...
```

Or visit `https://app1.yourdomain.com` in your browser. You should see a padlock icon.

### 9.4 Check Certificate Details:

```bash
echo | openssl s_client -connect app1.yourdomain.com:443 -servername app1.yourdomain.com 2>/dev/null | openssl x509 -noout -dates
```

---

## 🔧 Useful Docker Commands

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker compose -f docker-compose-npm.yml up -d` | Start NPM in background |
| `docker compose -f docker-compose-npm.yml down` | Stop and remove NPM container |
| `docker compose -f docker-compose-npm.yml restart` | Restart NPM |
| `docker logs nginx-proxy-manager` | View NPM logs |
| `docker logs -f nginx-proxy-manager` | Follow NPM logs in real-time |
| `docker network ls` | List all Docker networks |
| `docker network inspect npm_network` | Inspect npm_network details |
| `docker network connect npm_network container_name` | Connect container to network |
| `docker network disconnect npm_network container_name` | Disconnect container from network |

---

## 🛠 Troubleshooting

### ❌ "Port 80 or 443 is already in use"

**Cause:** Another service (nginx, apache, or another container) is using port 80 or 443.

**Solution:**
```bash
# Find what's using the port
sudo ss -tulpn | grep ':80\|:443'

# Stop nginx
sudo systemctl stop nginx
sudo systemctl disable nginx

# Or stop apache
sudo systemctl stop apache2
sudo systemctl disable apache2
```

### ❌ "Could not verify domain" when requesting SSL

**Cause:** DNS is not yet propagated, or port 80 is blocked.

**Check:**
```bash
# 1. Is your domain pointing to the correct IP?
dig +short app1.yourdomain.com

# 2. Is port 80 reachable from outside?
curl -I http://app1.yourdomain.com/.well-known/acme-challenge/test
```

### ❌ "Bad Gateway" (502) in NPM

**Cause:** NPM cannot reach your application's port.

**Check:**
```bash
# 1. Is your app container running?
docker ps | grep my-aspnet-app

# 2. Can you reach the app directly?
curl http://localhost:5000

# 3. If using container names, are they on the same network?
docker network inspect npm_network
```

### ❌ "Connection Refused"

**Cause:** Your app is not running, or the port is wrong.

**Solution:**
```bash
# Check if the port is listening
curl http://YOUR_SERVER_IP:5000

# Should return your app's response
```

### ❌ NPM Container Won't Start

View logs:

```bash
docker logs nginx-proxy-manager
```

Common issues:
- Port conflict (see above)
- Permission issues on mounted volumes

**Fix:**
```bash
# Fix volume permissions (if needed)
sudo chown -R 1000:1000 ./npm-data
sudo chown -R 1000:1000 ./npm-letsencrypt
```

---

## 📁 Repository Contents

| File/Folder | Description |
|-------------|-------------|
| `docker-compose-npm.yml` | Docker Compose configuration for Nginx Proxy Manager |
| `GUIDE.md` | This guide |
| `npm-data/` | NPM configuration data (auto-created after first run) |
| `npm-letsencrypt/` | Let's Encrypt SSL certificates (auto-created after first run) |

---

## 📝 Quick Reference Card

```bash
# 1. SSH into VPS
ssh user@your-server-ip

# 2. Clone repo & enter directory
git clone https://github.com/go2ismail/free-ssl-letsencrypt-aspnetcore-docker-npm-template.git
cd free-ssl-letsencrypt-aspnetcore-docker-npm-template

# 3. Start Nginx Proxy Manager
docker compose -f docker-compose-npm.yml up -d

# 4. Access admin panel
#    Open browser → http://YOUR_SERVER_IP:81
#    Login: admin@example.com / changeme

# 5. Add proxy hosts in the panel
#    Domain → IP:PORT (e.g., 5000, 8000, 8080)

# 6. Request SSL certificate from the SSL tab
```

---

## 🎯 Summary

| Before (Manual Nginx) | After (Nginx Proxy Manager) |
|-----------------------|------------------------------|
| ✍️ Edit nginx .conf files | 🖥️ Web GUI panel |
| 🔄 Manual `sudo nginx -s reload` | 🔄 Auto apply on save |
| 🔐 Manual certbot commands | 🔐 One-click Let's Encrypt SSL |
| 📝 Manual SSL renewal cron | 🤖 Auto SSL renewal |
| ❌ No visual dashboard | ✅ Dashboard with status |

Enjoy your free SSL certificates! 🎉