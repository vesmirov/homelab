# Vaultwarden

Self-hosted password manager (Bitwarden-compatible), behind Caddy with automatic Let's Encrypt certificates.

## Prerequisites

- Docker with the compose plugin
- LUKS volume mounted at `/mnt/vault`: data in `/mnt/vault/vaultwarden`, Caddy state in `/mnt/vault/caddy`, this directory in `/mnt/vault/stack`
- `cryptsetup`, `sqlite3`, `restic`

## Setup

1. Copy this directory to `/mnt/vault/stack`.
2. Create `.env` from the example and fill in `DOMAIN`, `VAULT_HOST`, `ACME_EMAIL`, and the SMTP block if mail is used:
   ```bash
   cp .env.example .env
   ```
3. Install the scripts and the backup timer:
   ```bash
   sudo cp scripts/* /usr/local/bin/
   ```
   ```bash
   sudo cp deploy/vault-backup.service deploy/vault-backup.timer /etc/systemd/system/
   ```
   ```bash
   sudo systemctl daemon-reload && sudo systemctl enable --now vault-backup.timer
   ```
4. Start:
   ```bash
   sudo vault-open
   ```

## Scripts

### vault-open

Opens the LUKS image, mounts it, verifies the mount point, runs `docker compose up -d`. Required after every reboot: the volume is not opened automatically. The mount point is immutable, so the containers cannot start on an unmounted volume.

### vault-close

Stops the containers, unmounts and closes the volume.

### vault-update

Copies the SQLite database to `/mnt/vault/backups` (last 5 kept), then `docker compose pull`, `up -d`, and prunes old images. Images use floating tags because Bitwarden clients update themselves and require a matching server. Suggested weekly cron in root's crontab:

```
0 4 * * 1 /usr/local/bin/vault-update >> /var/log/vault-update.log 2>&1
```

Exits without changes if the volume is closed.

Rollback: `vault-close`, restore the last copy from `/mnt/vault/backups` over `db.sqlite3`, pin the previous tag in `docker-compose.yml`, `vault-open`. Database migrations are one-way, so the copy is required.

### check-vaultwarden

Mount status, container status, local and public health endpoints, web vault responds, `/admin` returns 404.

## Admin panel

Disabled unless `ADMIN_TOKEN` is set. Store it as an argon2 hash in single quotes:

```bash
docker run --rm -it vaultwarden/server:latest /vaultwarden hash --preset owasp
```

Recreate the container. The panel is served only on the loopback port 8081 and is never reachable through the public domain. Two ways in:

- Tailscale: let tailscaled expose the loopback port inside the tailnet, with a certificate for the machine's MagicDNS name. Runs once, survives reboots:
  ```bash
  sudo tailscale serve --bg --https=8082 http://127.0.0.1:8081
  ```
  Then open `https://<machine>.<tailnet>.ts.net:8082/admin` from a device in the tailnet. Only WireGuard peers reach it, no host port is opened.
- SSH tunnel: `ssh -L 8081:127.0.0.1:8081 <host>` and `http://localhost:8081/admin`.

### vault-backup

Daily restic backup to S3-compatible storage, run by `vault-backup.timer` at 03:30. Credentials and the repository password live in `/mnt/vault/restic.env` (root, 0600), inside the LUKS volume:

```
RESTIC_REPOSITORY=s3:https://<endpoint>/<bucket>
RESTIC_PASSWORD=<repository password>
AWS_ACCESS_KEY_ID=<access key>
AWS_SECRET_ACCESS_KEY=<secret key>
```

What goes in: a consistent SQLite copy taken with `sqlite3 .backup`, the data directory without the live database files and the icon cache, `.env`, and Caddy state so certificates survive a rebuild. Retention: 14 daily, 8 weekly, 12 monthly. `restic check` runs on the first day of each month. The unit is skipped while the volume is closed.

First run, once:

```bash
sudo bash -c 'set -a; . /mnt/vault/restic.env; set +a; restic init'
```

```bash
sudo vault-backup
```

## Restore

On any machine with restic, the repository password and a token for the bucket:

```bash
restic restore latest --target /tmp/restore
```

The database is at `/tmp/restore/mnt/vault/backups/restic-stage/db.sqlite3`, the rest under `/tmp/restore/mnt/vault/`. Put `db.sqlite3` into the data directory, remove any stale `db.sqlite3-wal` and `db.sqlite3-shm`, restore attachments and `rsa_key.pem` next to it, then `vault-open`. Verify with `sqlite3 db.sqlite3 "PRAGMA integrity_check"` before starting.

Test the restore, not the backup: do this once after setup and after any change to the script.

## Backups

- restic to independent storage, see `vault-backup`. Without the repository password the backups are unreadable, keep it outside the server.
- Offline export from a Bitwarden client, encrypted. Restores without any server. Attachments are not included in exports.
- LUKS header backup, stored outside the server.

Never build images on the server.
