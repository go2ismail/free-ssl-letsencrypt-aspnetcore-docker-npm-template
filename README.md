# 🚀 Free SSL Let's Encrypt for ASP.NET Core Docker Apps via Nginx Proxy Manager

> Replace manual nginx config files with a beautiful web UI — Nginx Proxy Manager (NPM).  
> Auto-provision free SSL certificates from Let's Encrypt with just a few clicks.

---

## 🎥 Video Tutorial

Watch the complete step-by-step walkthrough:

[![How to Add Free SSL with Let's Encrypt to ASP.NET Core Docker Apps via Nginx Proxy Manager](https://img.youtube.com/vi/C14juEZdPWE/0.jpg)](https://www.youtube.com/watch?v=C14juEZdPWE)

**Title:** [How to Add Free SSL with Let's Encrypt to ASP.NET Core Docker Apps via Nginx Proxy Manager](https://www.youtube.com/watch?v=C14juEZdPWE)  
**Duration:** ~30 minutes  
**Description:** Covers everything from prerequisites to setup, configuration, and troubleshooting.

---

## 📂 Repository Contents

| File / Folder | Description |
|--------------|-------------|
| `docker-compose-npm.yml` | Docker Compose configuration for Nginx Proxy Manager (ports 80, 81, 443) |
| `GUIDE.md` | Complete step-by-step guide with copy-paste commands for Ubuntu |
| `npm-data/` | NPM configuration data (auto-created after first run) |
| `npm-letsencrypt/` | Let's Encrypt SSL certificates (auto-created after first run) |

---

## 🚀 Quick Start

```bash
# 1. SSH into your VPS
ssh user@your-server-ip

# 2. Clone this repository
git clone https://github.com/go2ismail/free-ssl-letsencrypt-aspnetcore-docker-npm-template.git
cd free-ssl-letsencrypt-aspnetcore-docker-npm-template

# 3. Start Nginx Proxy Manager
docker compose -f docker-compose-npm.yml up -d

# 4. Open browser → http://YOUR_SERVER_IP:81
#    Login: admin@example.com / changeme
#    (You will be forced to change credentials on first login)

# 5. In the NPM panel, add Proxy Hosts:
#    Domain → YOUR_SERVER_IP : 5000 (or 8000 / 8080)

# 6. Go to SSL tab → Request Let's Encrypt certificate → Done!
```

---

## 📖 Full Documentation

For detailed step-by-step instructions including:

- ✅ Prerequisites (Docker, DNS, firewall)
- ✅ Stopping existing nginx / Apache
- ✅ Adding proxy hosts with screenshots
- ✅ Requesting Let's Encrypt SSL certificates
- ✅ Seamless integration with existing containers
- ✅ Troubleshooting common issues

👉 **See [GUIDE.md](GUIDE.md)**

---

## 🎯 Why Nginx Proxy Manager?

| Before (Manual Nginx) | After (Nginx Proxy Manager) |
|-----------------------|------------------------------|
| ✍️ Edit nginx `.conf` files by hand | 🖥️ Web GUI panel |
| 🔄 Manual `sudo nginx -s reload` | 🔄 Auto apply on save |
| 🔐 Run certbot commands manually | 🔐 One-click Let's Encrypt SSL |
| 📝 Set up cron jobs for SSL renewal | 🤖 Auto SSL renewal |
| ❌ No visual dashboard | ✅ Dashboard with live status |

---

## 📌 Default Credentials

| Item | Value |
|------|-------|
| **NPM Admin URL** | `http://YOUR_SERVER_IP:81` |
| **Default Email** | `admin@example.com` |
| **Default Password** | `changeme` |

> ⚠️ **Change your email and password immediately after first login!**

---

## 🛠 Requirements

- Ubuntu 20.04+ / Debian 11+ VPS
- Docker & Docker Compose installed
- At least one domain name pointing to your server's IP
- Ports 80, 81, 443 open in firewall



---

## 📄 License

MIT