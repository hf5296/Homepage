# Docker Homelab Dashboard

A sanitised version of the Homepage configuration used to organise and monitor services in my Linux homelab. The stack runs [Homepage](https://gethomepage.dev/) as the service portal and [Netdata](https://www.netdata.cloud/) for host and container observability.

This public repository intentionally excludes live host addresses, generated logs, credentials and machine-specific service configuration.

## What this demonstrates

- Docker Compose service orchestration
- Read-only Docker socket integration for container status
- Host and container monitoring with Netdata
- A single dashboard for eight self-hosted services
- Separation of reusable configuration from environment-specific values
- Basic operational practices for updates, health checks and recovery

## Architecture

```text
                         Linux homelab host
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Browser ──► Homepage :3000 ──► service links and status     │
│                    │                                         │
│                    └──── read-only Docker socket             │
│                                                              │
│  Netdata :19999 ──► host, process and container metrics      │
│                                                              │
│  Linked services                                             │
│  ├─ Portainer               ├─ SearXNG                       │
│  ├─ OpenWebUI               ├─ Inflation Tracker             │
│  ├─ Trade Journal           ├─ Format Converter              │
│  └─ Leverage Calculator     └─ Netdata                       │
└──────────────────────────────────────────────────────────────┘
```

Homepage and Netdata are defined in `docker-compose.yml`. The remaining applications run as separate homelab workloads and are represented by the sanitised template in `config/services.example.yaml`.

## Repository structure

```text
.
├── docker-compose.yml
├── .env.example
└── config/
    ├── services.example.yaml
    ├── settings.yaml
    ├── widgets.yaml
    ├── docker.yaml
    ├── kubernetes.yaml
    ├── proxmox.yaml
    ├── custom.css
    └── custom.js
```

## Run locally

1. Clone the repository.
2. Copy `.env.example` to `.env` and set `HOMEPAGE_ALLOWED_HOSTS` for your environment.
3. Copy `config/services.example.yaml` to `config/services.yaml`.
4. Replace every `<HOMELAB_HOST>` value with a hostname or LAN address reachable from the Homepage container.
5. Start the dashboard:

```bash
docker compose up -d
```

Homepage will be available on port `3000` and Netdata on port `19999`.

## Operations

Update the images and recreate the containers:

```bash
docker compose pull
docker compose up -d
```

Useful checks after an update:

```bash
docker compose ps
docker compose logs --tail=100 homepage netdata
```

The dashboard configuration is stored as text so it can be reviewed and restored. Netdata configuration, state and cache data use named Docker volumes.

## Security notes

- Do not expose the dashboard or Netdata directly to the public internet.
- Restrict access with a host firewall, VPN or authenticated reverse proxy.
- Keep real service addresses and credentials out of the public repository.
- The Docker socket is mounted read-only, but access to it is still security-sensitive.
- Netdata requires elevated host visibility; review its capabilities and mounts for your own threat model.
- Pin image versions in environments where repeatable deployments are more important than automatic updates.

## Current limitations

- This repository documents the dashboard layer rather than provisioning every linked service.
- Backups and restore testing are performed outside this Compose stack.
- The public service file uses placeholders and must be adapted before deployment.
