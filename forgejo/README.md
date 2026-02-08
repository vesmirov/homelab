# Forgejo

Self-hosted Git server (GitHub/GitLab alternative).

## Prerequisites

- LUKS volume mounted at `/mnt/vault`
- Data directory: `/mnt/vault/forgejo`
- Installed sqlite3: `sudo apt install sqlite3 -y`

## Setup

1. Create data directory:
   ```bash
   sudo mkdir -p /mnt/vault/forgejo
   ```

2. Create `.env` from example:
   ```bash
   cp .env.example .env
   ```

3. Start the container:
   ```bash
   docker compose up -d
   ```

## Access

- Web UI: `http://<host>:8082`
- SSH: `ssh://git@<host>:2222`

## Post-setup

1. Access web UI at `http://localhost:8082`
2. Complete initial configuration (SQLite database is pre-configured)
3. Create first user (becomes admin automatically)
4. Add your SSH key in Settings → SSH/GPG Keys
5. Set `DISABLE_REGISTRATION=true` in `.env`
6. Recreate container: `docker compose up -d --force-recreate`

## Clone URLs

After setup, repositories can be cloned via:
- HTTP: `http://<host>:8082/<user>/<repo>.git`
- SSH: `ssh://git@<host>:2222/<user>/<repo>.git`

## Management Scripts

Scripts are located in `scripts/` directory and should be copied to `/usr/local/bin/` for system-wide access:
```bash
sudo cp scripts/* /usr/local/bin/
sudo chmod +x /usr/local/bin/start-forgejo /usr/local/bin/check-forgejo /usr/local/bin/backup-forgejo
```

### start-forgejo

Starts the Forgejo container.

- Checks if `/mnt/vault` is mounted (required, exits if not)
- Checks if `/mnt/backup` is mounted (optional, shows warning if not)
- Runs `docker compose up -d`

### check-forgejo

Shows current status of the service.

- Mount status for `/mnt/vault` and `/mnt/backup`
- Docker container status
- HTTP health check on `localhost:8082/api/healthz`

### backup-forgejo

Creates a backup of the Forgejo database and repositories.

- Checks if `/mnt/vault` and `/mnt/backup` are mounted (required)
- Creates SQLite backup to `/mnt/backup/forgejo/db/` (7-day retention)
- Syncs all repositories to `/mnt/backup/forgejo/repos/` (mirror)
- Syncs LFS objects to `/mnt/backup/forgejo/lfs/` (mirror)
- Logs to `/var/log/homelab-backup.log`

Cron (runs daily at 3:30 AM):
```bash
sudo crontab -e
# Add: 30 3 * * * /usr/local/bin/backup-forgejo
```
