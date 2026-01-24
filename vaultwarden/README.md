# Vaultwarden

Self-hosted password manager (Bitwarden-compatible).

## Prerequisites

- LUKS volume mounted at `/mnt/vault`
- Data directory: `/mnt/vault/vaultwarden`

## Setup

1. Create data directory:
   ```bash
   sudo mkdir -p /mnt/vault/vaultwarden
   sudo chown -R $USER:$USER /mnt/vault/vaultwarden
   ```

2. Create `.env` from example:
   ```bash
   cp .env.example .env
   ```

3. Generate admin token and add to `.env`:
   ```bash
   openssl rand -base64 48
   ```

4. Start the container:
   ```bash
   docker compose up -d
   ```

## Access

- Admin panel: `<domain>/admin`
- Web vault: `<domain>`

## Post-setup

1. Create first user via web interface
2. Set `SIGNUPS_ALLOWED=false` in `.env`
3. Recreate container: `docker compose up -d --force-recreate`
4. Enable 2FA in user settings
