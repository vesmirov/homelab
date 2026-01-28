# Nextcloud

Cloud storage with PostgreSQL and Redis.

## Services

- **app** — Nextcloud web application
- **db** — PostgreSQL database
- **cache** — Redis for distributed caching and file locking
- **cron** — Background tasks runner

## Setup

1. Copy `.env.example` to `.env` and set secure passwords
2. Start containers: `docker compose up -d`
3. Complete web setup at `http://localhost:8080`
4. Apply post-install configuration (see below)
5. Set background jobs to "Cron" in Settings → Administration → Basic settings

## Post-install configuration

Add to `config/config.php`:

### Redis caching

```php
'memcache.local' => '\\OC\\Memcache\\APCu',
'memcache.distributed' => '\\OC\\Memcache\\Redis',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'redis' => [
    'host' => 'cache',
    'password' => '',
    'port' => 6379,
],
```

### Reverse proxy (Cloudflare)

```php
'trusted_proxies' => ['172.18.0.0/16', '127.0.0.1'],
'forwarded_for_headers' => ['HTTP_X_FORWARDED_FOR', 'HTTP_CF_CONNECTING_IP'],
```

Adjust `trusted_proxies` subnet to match your Docker network (`docker network inspect nextcloud_nextcloud`).

### Video previews

The custom Dockerfile includes FFmpeg for video thumbnail generation. Add to `config/config.php`:

```php
'enable_previews' => true,
'enabledPreviewProviders' => [
    'OC\Preview\PNG',
    'OC\Preview\JPEG',
    'OC\Preview\GIF',
    'OC\Preview\BMP',
    'OC\Preview\XBitmap',
    'OC\Preview\MP3',
    'OC\Preview\TXT',
    'OC\Preview\MarkDown',
    'OC\Preview\Movie',
    'OC\Preview\HEIC',
],
'preview_max_x' => 1024,
'preview_max_y' => 768,
'preview_max_memory' => 512,
```

Note: `OC\Preview\Movie` handles all video formats (MP4, MKV, AVI, etc.) via FFmpeg.

## Management Scripts

Scripts are located in `scripts/` directory and should be copied to `/usr/local/bin/` for system-wide access:
```bash
sudo cp scripts/* /usr/local/bin/
sudo chmod +x /usr/local/bin/start-nextcloud /usr/local/bin/check-nextcloud
```

### start-nextcloud

Starts the Nextcloud containers.

- Checks if `/mnt/vault` is mounted (required, exits if not)
- Checks if `/mnt/backup` is mounted (optional, shows warning if not)
- Runs `docker compose up -d`

### check-nextcloud

Shows current status of the service.

- Mount status for `/mnt/vault` and `/mnt/backup`
- Docker containers status
- HTTP health check on `localhost:8080`

### backup-nextcloud

Creates a backup of the Nextcloud database and syncs files.

- Checks if `/mnt/vault` and `/mnt/backup` are mounted (required)
- Dumps PostgreSQL to `/mnt/backup/nextcloud/db/`
- Syncs files to `/mnt/backup/nextcloud/data/` (mirror, deletes removed files)
- Keeps last 7 database dumps, removes older
- Logs to `/var/log/homelab-backup.log`

Cron (runs daily at 3:15 AM):
```bash
sudo crontab -e
# Add: 15 3 * * * /usr/local/bin/backup-nextcloud
```
