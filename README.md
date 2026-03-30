Your GitHub README.md
# 🚀 Listmonk and PostgreSQL Installation(RHEL) and Configuration Guide

This guide provides a step-by-step procedure to:
- Prepare a Linux environment
- Configure security settings
- Deploy Listmonk using Docker Compose

---

## 📦 1. Docker & Docker Compose Version Check

Verify installation:

```bash
docker --version
docker compose --version
```

If not installed, follow official Docker documentation.

🔐 2. Security and Firewall Configuration

Disable Firewall
```bash
systemctl status firewalld
systemctl disable --now firewalld
systemctl status firewalld
```

Disable SELinux

Check status:
```bash
getenforce
```
Disable permanently:
```bash
grubby --update-kernel ALL --args selinux=0
reboot
```
Verify after reboot:
```bash
getenforce
```
⚙️ 3. Listmonk Deployment
📁 Step 3.1: Directory and Permissions Setup

```bash
mkdir -p /opt/listmonk-service
mkdir -p /opt/listmonk-service/config
touch /opt/listmonk-service/config/config.toml
chmod 644 /opt/listmonk-service/config/config.toml
chown root:root /opt/listmonk-service/config/config.toml
```

Edit config:

```bash
vi /opt/listmonk-service/config/config.toml
```

🐳 Step 3.2: Docker Compose Setup
```bash
cd /opt/listmonk-service
vi docker-compose.yml
```
Paste:
```bash
services:
  db:
    image: postgres:17-alpine
    container_name: listmonk_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: listmonk
      POSTGRES_PASSWORD: StrongPass123
      POSTGRES_DB: listmonk
    volumes:
      - /opt/listmonk-service/data:/var/lib/postgresql/data
      - /etc/localtime:/etc/localtime
    networks:
      - listmonk-net

  app:
    image: listmonk/listmonk:latest
    container_name: listmonk_app
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - "9000:9000"
    volumes:
      - /opt/listmonk-service/config/config.toml:/listmonk/config.toml
      - /etc/localtime:/etc/localtime
    command: ["./listmonk", "--config", "/listmonk/config.toml"]
    networks:
      - listmonk-net

networks:
  listmonk-net:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.0.0/24
```

▶️ 4. Starting the Service
Start Containers
docker compose up -d
🛠 Step 4.2: Database Initialization (First Time)

Check logs:

```bash
docker logs listmonk_app
```

If error:

"database does not appear to be setup"

Run:
```bash
docker compose restart
docker exec -it listmonk_app ./listmonk --install --config /listmonk/config.toml
docker compose restart
```

⚠️ Important Notes
🔹 Subnet Conflict

If 10.0.0.0/24 is already in use:

Change to:

172.100.0.0/24
🔹 Persistent Data

Database stored in:

/opt/listmonk-service/data

⚠️ Do NOT delete this directory — data loss will occur.

🎯 Access Listmonk

Open browser:

```bash
http://your-server-ip:9000
```


```md
![Docker](https://img.shields.io/badge/docker-ready-blue)
![PostgreSQL](https://img.shields.io/badge/postgres-17-green)
![License](https://img.shields.io/badge/license-MIT-yellow)
