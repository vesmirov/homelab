# Homelab

Self-hosted personal infrastructure. Privacy by design.

A collection of self-hosted services replacing well-known commercial alternatives. Each service runs in Docker and can be deployed independently. No vendor lock-in, full data ownership.

## Services

| Service | Replaces |
|---------|----------|
| [Nextcloud](./nextcloud) | Google Drive, Contacts, Calendar |
| [Vaultwarden](./vaultwarden) | 1Password, LastPass |
| [Forgejo](./forgejo) | GitHub, GitLab |

## Requirements

- Docker
- Docker Compose
- (optional) Cloudflare Tunnel
- (optional) Tailscale

## Remote Access

If you don't have a static IP or port forwarding configured, you can use one of these solutions for remote access:

### Cloudflare Tunnel

Exposes services via Cloudflare's network without opening ports.
```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

Documentation: [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/)

### Tailscale

Creates a private mesh VPN between your devices.
```bash
sudo systemctl enable tailscaled
sudo systemctl start tailscaled
```

Documentation: [Tailscale](https://tailscale.com/kb/)

### Note

Both can be used simultaneously. Cloudflare Tunnel is better for public access, Tailscale for private device-to-device connections.
