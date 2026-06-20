# VPS & Server Infrastructure

> **Self-managed Ubuntu VPS in Germany running a full Docker/Coolify/Traefik stack behind UFW, Fail2Ban, and NordVPN — production-grade hosting for 3 Hermes Agent profiles and 27 cron jobs.**

---

## Hosting Specs

| Resource | Specification |
|---|---|
| **Provider** | Self-managed VPS (Germany) |
| **OS** | Ubuntu 24.04.4 LTS |
| **Kernel** | 6.8.0-124-generic |
| **RAM** | 7.8 GB total (~5.2 GB available) |
| **Disk** | 116 GB total / 94 GB free (19% used) |
| **Access** | SSH (port 22) from Windows 11 PC |

---

## VPN & Network Layer

NordVPN provides privacy and geo-flexibility via the NordLynx protocol (WireGuard-based):

| Setting | Value |
|---|---|
| **Protocol** | NordLynx (WireGuard) |
| **Kill Switch** | OFF (production requirement — services must never be interrupted) |
| **DNS** | 1.1.1.1 |
| **IPv6** | Disabled |
| **Exit Country** | Netherlands |
| **SSH** | Port 22 allowlisted |

> **Why kill switch is off:** This is a production VPS running Telegram bots, cron jobs, and web services. A VPN tunnel drop must not take down live services. The VPN provides an anonymity layer, not a hard security boundary.

---

## Container & Service Stack

```
┌─────────────────────────────────────────┐
│              Docker Engine               │
│  ┌─────────┐  ┌──────────┐  ┌────────┐  │
│  │ Coolify │  │  Traefik │  │ Nginx  │  │
│  │ (mgmt)  │  │ (proxy)  │  │(web)   │  │
│  └─────────┘  └──────────┘  └────────┘  │
│  ┌──────────────────────┐               │
│  │    PostgreSQL         │               │
│  └──────────────────────┘               │
└─────────────────────────────────────────┘
```

| Component | Role |
|---|---|
| **Docker** | Container runtime for all services |
| **Coolify** | Self-hosted PaaS — app deployment & management |
| **Traefik** | Reverse proxy & automatic TLS (Let's Encrypt) |
| **Nginx** | Web server for static/dynamic content |
| **PostgreSQL** | Relational database for applications |

---

## Firewall & Security

```bash
# UFW - default deny incoming
ufw default deny incoming
ufw default allow outgoing

# Published ports
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw allow 8000/tcp  # App services
ufw allow 8080/tcp  # App services
ufw allow 6001/tcp  # App services
ufw allow 6002/tcp  # App services
```

| Layer | Status |
|---|---|
| **UFW** | Active, default deny incoming |
| **Fail2Ban** | Active, monitoring SSH brute-force attempts |
| **SSH** | `PasswordAuthentication no`, `PermitRootLogin without-password` |
| **Published Ports** | 22, 80, 443, 8000, 8080, 6001, 6002 |

---

## Hermes Agent Footprint

The agent and all its components live under `/root/.hermes/`:

| Path | Purpose |
|---|---|
| `/usr/local/lib/hermes-agent` | Core installation |
| `/root/.hermes/config.yaml` | Main configuration |
| `/root/.hermes/.env` | API keys and secrets |
| `/root/.hermes/skills/` | 71 installed skills |
| `/root/.hermes/scripts/` | Watchdog, backup, and utility scripts |
| `/root/.hermes/cron/` | Cron job definitions (27 jobs) |
| `/root/.hermes/profiles/` | Isolated profile directories |
| `/root/.hermes/sessions/` | Conversation transcripts |
| `/root/.hermes/logs/` | Gateway and agent logs |

---

## Why This Stack Works

- **Single VPS, full platform:** Docker + Coolify eliminates the need for managed platforms
- **VPN without breaking services:** Kill switch off ensures uptime; SSH allowlist ensures access
- **Security in layers:** UFW → Fail2Ban → SSH key-only → profile isolation
- **Cost-efficient:** One server runs everything — bots, cron, databases, web apps
