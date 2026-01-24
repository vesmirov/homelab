# Homelab

Self-hosted personal infrastructure. Privacy by design.

A collection of self-hosted services replacing well-known commercial alternatives. Each service runs in Docker and can be deployed independently. No vendor lock-in, full data ownership.

## Services

| Service | Replaces |
|---------|----------|
| [Nextcloud](./nextcloud) | Google Drive, Contacts, Calendar |
| Vaultwarden | 1Password, LastPass |

## Requirements

- Docker
- Docker Compose
- Cloudflare Tunnel / Tailscale configured on server

## Usage

cd <service>
docker compose up -d
