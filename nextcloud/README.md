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
