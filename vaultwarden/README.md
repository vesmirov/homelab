# Vaultwarden

Self-hosted password manager (Bitwarden-compatible), behind Caddy with automatic Let's Encrypt certificates.

## Prerequisites

- Docker with the compose plugin
- LUKS volume mounted at `/mnt/vault`: data in `/mnt/vault/vaultwarden`, Caddy state in `/mnt/vault/caddy`, this directory in `/mnt/vault/stack`
- `cryptsetup`, `sqlite3`

## Setup

1. Copy this directory to `/mnt/vault/stack`.
2. Create `.env` from the example and fill in `DOMAIN`, `VAULT_HOST`, `ACME_EMAIL`, and the SMTP block if mail is used:
   ```bash
   cp .env.example .env
   ```
3. Install the scripts:
   ```bash
   sudo cp scripts/* /usr/local/bin/
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

Recreate the container and reach the panel through an SSH tunnel to the local port, never through the public domain. Remove the token when done.

## Backups

- Offline export from a Bitwarden client, encrypted, on removable media, refreshed regularly. Restores without any server.
- restic of `/mnt/vault/vaultwarden` to independent storage.
- LUKS header backup, stored with the offline exports.

Never build images on the server.
