# Vaultwarden

Self-hosted password manager (Bitwarden-compatible).

## Prerequisites

- LUKS volume mounted at `/mnt/vault`
- Data directory: `/mnt/vault/vaultwarden`
- installed sqlite3: `sudo apt install sqlite3 -y`

## Setup

1. Create data directory:
   ```bash
   sudo mkdir -p /mnt/vault/vaultwarden
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

## Management Scripts

Scripts are located in `scripts/` directory and should be copied to `/usr/local/bin/` for system-wide access:
```bash
sudo cp scripts/* /usr/local/bin/
sudo chmod +x /usr/local/bin/start-vaultwarden /usr/local/bin/check-vaultwarden
```

### start-vaultwarden

Starts the Vaultwarden container.

- Checks if `/mnt/vault` is mounted (required, exits if not)
- Checks if `/mnt/backup` is mounted (optional, shows warning if not)
- Runs `docker compose up -d`

### check-vaultwarden

Shows current status of the service.

- Mount status for `/mnt/vault` and `/mnt/backup`
- Docker container status
- HTTP health check on `localhost:8081/alive`

### backup-vaultwarden

Creates a backup of the Vaultwarden database.

- Checks if `/mnt/vault` and `/mnt/backup` are mounted (required)
- Creates SQLite backup to `/mnt/backup/vaultwarden/db/`
- Keeps last 7 days, removes older backups
- Logs to `/var/log/homelab-backup.log`

Cron (runs daily at 3:00 AM):
```bash
sudo crontab -e
# Add: 0 3 * * * /usr/local/bin/backup-vaultwarden
```
