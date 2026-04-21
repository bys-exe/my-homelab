# 🏠 Personal Homelab

A self-hosted homelab running on an old laptop repurposed as a home server. Built to replace cloud services with privacy-respecting, locally-hosted alternatives — accessible from anywhere via a secure VPN tunnel.

---

## Stack

| Service       | Purpose                                                        | Tech            |
| ------------- | -------------------------------------------------------------- | --------------- |
| **Immich**    | Self-hosted photo & video backup (Google Photos alternative)   | Docker          |
| **Homepage**  | Unified dashboard for all services                             | Docker          |
| **Tailscale** | Secure remote access via VPN mesh network                      | WireGuard-based |
| **Immich ML** | On-device machine learning for face recognition & smart search | Docker          |

---

## Architecture

```
                        ┌─────────────────────────────────┐
                        │         Old Laptop Server       │
                        │         (Ubuntu Server)         │
                        │                                 │
  Local Network  ──────▶│  :2283  Immich                  │
                        │  :3000  Homepage Dashboard      │
                        │                                 │
                        │  ┌─────────────────────────┐    │
                        │  │        Docker           │    │
                        │  │  immich-server          │    │
                        │  │  immich-machine-learning│    │
                        │  │  postgres               │    │
                        │  │  redis (valkey)         │    │
                        │  └─────────────────────────┘    │
                        │                                 │
                        │  Tailscale (VPN mesh)           │
                        └───────────────┬─────────────────┘
                                        │
                        ┌───────────────▼─────────────────┐
                        │       Tailscale Network         │
                        │   Access from phone / laptop    │
                        │     anywhere in the world       │
                        └─────────────────────────────────┘
```

---

## Services

### Immich

A full-featured, self-hosted photo and video backup solution. Replaces Google Photos completely.

- Automatic mobile backup
- Face recognition and person tagging (runs on-device via the ML container)
- Smart search powered by CLIP embeddings
- Timeline view, albums, shared libraries
- Runs as 4 coordinated containers: server, machine learning, postgres, redis

**Access:** `http://<server-ip>:2283` on local network, or via Tailscale IP from anywhere

![Immich Screenshot](./immich-1.png)

### Homepage Dashboard

A clean, customizable dashboard that shows all running services in one place.

- Live status indicators for each service
- Quick-access links to all self-hosted apps
- Configured via YAML files

**Access:** `http://<server-ip>:3000`

![Homepage Screenshot](./server-dashboard.png)

### Tailscale

Tailscale creates a secure WireGuard-based mesh VPN between devices. This means the home server is reachable from anywhere — phone, laptop, college — without exposing any ports to the public internet.

- No open ports on the home router
- End-to-end encrypted traffic
- Works behind CGNAT (most Indian ISPs (i hate you airtel))

---

## Setup

### Prerequisites

- A machine running Ubuntu Server (tested on 22.04 LTS)
- Docker and Docker Compose installed
- A Tailscale account (free tier is sufficient)

### Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

### Clone this repo

```bash
git clone https://github.com/YOUR_USERNAME/homelab.git
cd homelab
```

### Deploy Immich

```bash
cd immich
cp .env.example .env
# Edit .env with your preferred passwords and storage paths
nano .env
docker compose up -d
```

### Set up Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Follow the auth link, then your server is accessible from any Tailscale-connected device.

---

## Environment Variables

Create a `.env` file in the `immich/` directory:

```env
# Storage
UPLOAD_LOCATION=/path/to/your/photos
DB_DATA_LOCATION=/path/to/postgres/data

# Database credentials (change these)
DB_PASSWORD=your_secure_password
DB_USERNAME=postgres
DB_DATABASE_NAME=immich

# Immich version (leave as-is for latest stable)
IMMICH_VERSION=release
```

> ⚠️ Never commit your `.env` file. It's listed in `.gitignore`.

---

## Why I Built This

I wanted to stop relying on Google Photos for storing personal photos while also learning how real server infrastructure works. This project taught me:

- **Linux server administration** — managing a headless Ubuntu Server
- **Docker & containerization** — running and coordinating multiple containers with Docker Compose
- **Networking** — local network access, port mapping, and VPN-based remote access with Tailscale
- **Service dependencies** — understanding how Immich's server, ML, database, and cache containers depend on each other
- **Self-hosting** — the full lifecycle of deploying, maintaining, and accessing a production-like service

---

## References

- [Immich Documentation](https://docs.immich.app)
- [Tailscale Documentation](https://tailscale.com/kb)
- [Docker Documentation](https://docs.docker.com)
