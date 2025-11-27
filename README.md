# Vaultwarden --- Secure Password Manager on Raspberry Pi 5

[![MIT
License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![Docker
Compose](https://img.shields.io/badge/Docker-Compose-blue?logo=docker&style=flat-square)](https://www.docker.com/)
[![ZFS](https://img.shields.io/badge/ZFS-OpenZFS-blue?style=flat-square)](https://openzfs.org/)
[![Vaultwarden](https://img.shields.io/badge/Vaultwarden-Bitwarden%20Compatible-purple?style=flat-square)](https://github.com/dani-garcia/vaultwarden)

This repository provides a secure, homelab-ready Docker Compose
deployment of Vaultwarden on a Raspberry Pi 5, using ZFS-backed or
EXT4-based persistent storage and a reverse-proxy-only access model
using Nginx Proxy Manager with Let's Encrypt.

⚠️ No container ports are exposed to the host or public network.\
All access is forced through the reverse proxy over HTTPS.

------------------------------------------------------------------------

## 📁 Directory Structure

### ZFS Layout (Recommended)

``` bash
tank/
├── docker/
│   ├── compose/
│   │   └── vaultwarden/
│   │       ├── docker-compose.yml
│   │       ├── .env
│   │       ├── env.example
│   │       └── README.md
│   └── data/
│       └── vaultwarden/
```

### EXT4 / Standard Filesystem Layout

``` bash
~/docker/
├── compose/
│   └── vaultwarden/
│       ├── docker-compose.yml
│       ├── .env
│       ├── env.example
│       └── README.md
└── data/
    └── vaultwarden/
```

------------------------------------------------------------------------

## 🧰 Prerequisites

-   Docker Engine
-   Docker Compose V2
-   Git
-   Raspberry Pi 5 (ARM64)
-   (Optional, Recommended) ZFS on Linux
-   A running Nginx Proxy Manager instance
-   OpenSSL or Python 3 for generating a strong admin token (both are
    commonly preinstalled)

------------------------------------------------------------------------

## ⚙️ Setup Instructions

### 1. Clone Repository

#### ZFS

``` bash
sudo zfs create -p tank/docker/compose/vaultwarden
cd /tank/docker/compose/vaultwarden
sudo git clone https://github.com/Vantasin/VaultWarden.git .
```

#### EXT4

``` bash
mkdir -p ~/docker/compose/vaultwarden
cd ~/docker/compose/vaultwarden
git clone https://github.com/Vantasin/VaultWarden.git .
```

------------------------------------------------------------------------

### 2. Create Persistent Data Directory

#### ZFS

``` bash
sudo zfs create -p tank/docker/data/vaultwarden
```

#### EXT4

``` bash
mkdir -p ~/docker/data/vaultwarden
```

------------------------------------------------------------------------

### 3. Configure Environment Variables

``` bash
sudo cp env.example .env
sudo nano .env
sudo chmod 600 .env
```

Key values to set:

``` env
DOMAIN=https://vault.example.com
ADMIN_TOKEN=super-long-random-token
DATA_DIR=/tank/docker/data/vaultwarden
SIGNUPS_ALLOWED=false
WEBSOCKET_ENABLED=true
LOG_LEVEL=info
TZ=UTC
UID=1000
GID=1000
```

> **Note:** Be sure to update the `DOMAIN` & `ADMIN_TOKEN` variables!

Generate and update `ADMIN_TOKEN` automatically (copy/paste one of these):

OpenSSL (prints a token; paste into `.env`):

``` bash
openssl rand -hex 48
```

Python (writes directly into `.env`):

``` bash
sudo python3 - <<'PY'
import pathlib, re, secrets
p = pathlib.Path('.env')
text = p.read_text()
token = secrets.token_urlsafe(48)
text = re.sub(r'^ADMIN_TOKEN=.*$', f'ADMIN_TOKEN={token}', text, flags=re.M)
p.write_text(text)
print(f"ADMIN_TOKEN updated in {p}")
PY
```

------------------------------------------------------------------------

### 4. Create (or use) the Proxy Network

This stack expects the external Docker network used by Nginx Proxy
Manager to already exist (e.g., `npm_proxy` from your [Nginx Proxy Manager](https://github.com/Vantasin/Nginx-Proxy-Manager.git) repo). If it
does not, create it once:

``` bash
sudo docker network create npm_proxy
```

If the network already exists (e.g., created by [Nginx Proxy Manager](https://github.com/Vantasin/Nginx-Proxy-Manager.git)),
this command is a no-op.

------------------------------------------------------------------------

## 🌐 [Nginx Proxy Manager](https://github.com/Vantasin/Nginx-Proxy-Manager.git) Setup

### Add Proxy Host

#### Details tab

| Field               | Value                    |
|:--------------------|:-------------------------|
| Domain Names        | `vault.example.com`      |
| Scheme              | `http`                   |
| Forward Hostname/IP | `vaultwarden`            |
| Forward Port        | `80`                     |
| Access List         | `Public` (or as desired) |
| Cache Assets        | Disable                  |
| Block Common Exploits | Enable                 |
| Websockets Support  | Enable                   |

#### SSL tab

| Field               | Value/Action                                 |
|:--------------------|:----------------------------------------------|
| SSL Certificate     | Request new Let's Encrypt certificate         |
| Force SSL           | Enable                                       |
| HTTP/2 Support      | Enable                                       |
| HSTS Enabled        | Optional (enable if you want strict transport)|
| HSTS Sub-Domains    | Optional (with HSTS)                         |

------------------------------------------------------------------------

## ▶️ Start Vaultwarden

``` bash
sudo docker compose config -q && sudo docker compose up -d
```

------------------------------------------------------------------------

## 🔐 Accessing Vaultwarden

- Main UI: `https://vault.example.com`
- Admin UI: `https://vault.example.com/admin`

------------------------------------------------------------------------

## 🔒 Security Model

-   No exposed Docker ports (`expose` only shares within the proxy
    network)
-   Forced HTTPS via the reverse proxy
-   Reverse-proxy only ingress
-   ZFS snapshot-safe persistence

------------------------------------------------------------------------

## 🙏 Acknowledgments

-   [Vaultwarden](https://github.com/dani-garcia/vaultwarden) ---
    Lightweight Bitwarden-compatible password manager\
-   [Docker](https://www.docker.com/) --- Container runtime and
    orchestration\
-   [OpenZFS](https://openzfs.org/) --- Enterprise-grade filesystem with
    snapshots\
-   [Nginx Proxy
    Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)
    --- Reverse proxy with automatic HTTPS\
-   [ChatGPT](https://openai.com/chatgpt) --- Documentation and
    automation assistance
