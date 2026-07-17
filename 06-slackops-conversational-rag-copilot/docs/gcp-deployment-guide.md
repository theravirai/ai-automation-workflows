# GCP Production Deployment & Infrastructure Guide

> **Target Endpoint:** `https://n8n.ravirai.dev`  
> **Infrastructure:** Google Cloud Platform (GCP) Compute Engine  
> **Instance Name:** `n8n` | **Zone:** `us-central1-a`  
> **Ingress Security:** Cloudflare Zero Trust Tunnel (`cloudflared`) | End-to-End TLS 1.3

---

## 1. Architectural Overview

Running enterprise AI automation workflows in production requires 24/7 availability, persistent storage, isolated webhook routing, and hardened ingress security. Rather than exposing port `5678` directly to the public internet with open firewall rules, this architecture leverages **GCP Compute Engine** containerized via **Docker Compose**, fronted by an encrypted **Cloudflare Zero Trust Tunnel**.

```mermaid
flowchart LR
    subgraph Internet ["Public Ingress"]
        Slack["Slack API Webhooks<br/>(app_mention, file_share)"]
        Client["Admin Browser / MCP Client<br/>(Antigravity / Cursor)"]
    end

    subgraph Cloudflare ["Cloudflare Edge Network"]
        CF["Cloudflare DNS & Edge SSL<br/>n8n.ravirai.dev<br/>(DDoS Shield & Edge Caching)"]
    end

    subgraph GCP ["GCP Compute Engine (us-central1-a)"]
        Tunnel["cloudflared Daemon<br/>(Outbound Tunnel Client)"]
        Docker["Docker Engine"]
        
        subgraph Containers ["Docker Compose Environment"]
            N8N["n8n Orchestrator<br/>Port 5678 (Internal)"]
            Vol[("Persistent SSD Volume<br/>/home/n8n/.n8n")]
        end
    end

    Slack -->|"HTTPS POST"| CF
    Client -->|"HTTPS / MCP JSON-RPC"| CF
    CF <-->|"Encrypted Tunnel Stream"| Tunnel
    Tunnel -->|"HTTP 5678 (Localhost)"| N8N
    N8N <--> Vol
```

---

## 2. Infrastructure Provisioning (GCP Compute Engine)

### Instance Sizing & Configuration
- **Machine Family:** General-purpose (`E2`)
- **Machine Type:** `e2-micro` (2 vCPUs, 1 GB RAM) or `e2-medium` (2 vCPUs, 4 GB RAM recommended for multi-turn LangChain & document chunking)
- **Boot Disk:** 25 GB Balanced Persistent Disk (Debian GNU/Linux 12 bookworm)
- **Firewall Rules:** Inbound HTTP/HTTPS rules are **disabled**; all traffic is strictly brokered through the outbound Cloudflare Tunnel.

### Swap Memory Configuration (Crucial for `e2-micro`)
To prevent `Out of Memory (OOM)` kernel kills during heavy PDF chunking and embedding operations:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## 3. Containerized n8n Stack (`docker-compose.yml`)

The n8n orchestrator runs containerized with persistent host-volume mounting:

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-production
    restart: unless-stopped
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      # Core Host & Webhook Settings
      - N8N_HOST=n8n.ravirai.dev
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.ravirai.dev/
      - GENERIC_TIMEZONE=UTC
      
      # Binary Data & Performance Tuning
      - N8N_DEFAULT_BINARY_DATA_MODE=separate
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168  # Retain execution logs for 7 days
      
      # Model Context Protocol (MCP) Server Endpoint
      - N8N_MCP_SERVER_ENABLED=true
      - N8N_DIAGNOSTICS_ENABLED=false
    volumes:
      - /home/n8n/.n8n:/home/node/.n8n
```

### Volume Persistence & File Permissions
Ensure the non-root `node` user (UID `1000`) inside the container has ownership of the database volume:
```bash
sudo mkdir -p /home/n8n/.n8n
sudo chown -R 1000:1000 /home/n8n/.n8n
sudo chmod 700 /home/n8n/.n8n
```

---

## 4. Cloudflare Zero Trust Tunnel Setup

Cloudflare Tunnel (`cloudflared`) establishes a lightweight, bidirectional tunnel directly from the GCP VM to Cloudflare's edge network.

### Benefits
1. **Zero Open Ports:** External port scanners cannot detect or probe your origin server.
2. **Automatic TLS 1.3:** Certificates are provisioned and renewed automatically by Cloudflare.
3. **DDoS Protection & Web Application Firewall (WAF):** Blocks bot floods and malformed payloads before they reach the GCP VM.

### Installation & Systemd Service
1. Install `cloudflared`:
   ```bash
   curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
   sudo dpkg -i cloudflared.deb
   ```
2. Authenticate and create the tunnel:
   ```bash
   cloudflared tunnel login
   cloudflared tunnel create n8n-gcp-tunnel
   ```
3. Route DNS to the tunnel:
   ```bash
   cloudflared tunnel route dns n8n-gcp-tunnel n8n.ravirai.dev
   ```
4. Configuration file (`/etc/cloudflared/config.yml`):
   ```yaml
   tunnel: <TUNNEL_UUID>
   credentials-file: /etc/cloudflared/<TUNNEL_UUID>.json

   ingress:
     - hostname: n8n.ravirai.dev
       service: http://localhost:5678
     - service: http_status:404
   ```
5. Install and enable the systemd service for zero-downtime auto-boot:
   ```bash
   sudo cloudflared service install
   sudo systemctl enable --now cloudflared
   ```

---

## 5. Systemd Daemon for Docker Compose

To guarantee n8n restarts automatically if the GCP instance reboots or recovers from a maintenance event, create a systemd service:

File: `/etc/systemd/system/n8n-docker.service`
```ini
[Unit]
Description=n8n Automation Engine
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/n8n
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now n8n-docker.service
```

---

## 6. Automated Backup Strategy

To prevent data loss of workflows, credentials, and execution history, an automated cron job syncs the SQLite database to a Google Cloud Storage (GCS) bucket:

```bash
#!/usr/bin/env bash
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/tmp/n8n_backups"
GCS_BUCKET="gs://cloudscale-n8n-backups"

mkdir -p "$BACKUP_DIR"
# Safe hot backup using SQLite online backup
sqlite3 /home/n8n/.n8n/database.sqlite ".backup '$BACKUP_DIR/n8n_$TIMESTAMP.sqlite'"

# Compress and upload to GCS
gzip "$BACKUP_DIR/n8n_$TIMESTAMP.sqlite"
gsutil cp "$BACKUP_DIR/n8n_$TIMESTAMP.sqlite.gz" "$GCS_BUCKET/"

# Retain only last 30 days of local backups
find "$BACKUP_DIR" -type f -name "*.sqlite.gz" -mtime +30 -delete
```
Schedule via crontab (`crontab -e`):
```cron
0 2 * * * /home/n8n/scripts/backup_n8n.sh > /dev/null 2>&1
```
