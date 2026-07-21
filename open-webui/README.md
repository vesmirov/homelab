# Open WebUI

Self-hosted AI chat interface (ChatGPT alternative) backed by Ollama, with RAG and Jupyter-powered code execution.

## Services

- **app** — Open WebUI web application
- **ollama** — Local LLM runtime (embeddings + optional local models)
- **jupyter** — Sandbox for code execution and code interpreter (custom image with data/PDF/Office libraries)
- **files** — nginx serving generated Jupyter artifacts as downloadable files

## Prerequisites

- LUKS volume mounted at `/mnt/vault`
- Data directory: `/mnt/vault/open-webui`
- Installed sqlite3: `sudo apt install sqlite3 -y`

## Setup

1. Create data directories:
   ```bash
   sudo mkdir -p /mnt/vault/open-webui/{data,ollama,jupyter}
   ```

2. Create `.env` from example:
   ```bash
   cp .env.example .env
   ```

3. Generate secrets and add them to `.env`:
   ```bash
   openssl rand -hex 32   # WEBUI_SECRET_KEY
   openssl rand -hex 32   # JUPYTER_TOKEN
   ```
   Set `OLLAMA_BASE_URL` to your Ollama backend and `CORS_ALLOW_ORIGIN` to the
   origins you access the UI from (semicolon-separated).

4. Start the containers (builds the custom Jupyter image on first run):
   ```bash
   docker compose up -d
   ```

## Access

- Web UI: `http://<host>:3000`
- Generated files: `http://<host>:8090`

## Post-setup

1. Access web UI at `http://localhost:3000`
2. Create first user (becomes admin automatically)
3. Disable open sign-ups in Settings → Admin → General once your users are created
4. (optional) Pull local models: `docker compose exec ollama ollama pull <model>`

## Management Scripts

Scripts are located in `scripts/` directory and should be copied to `/usr/local/bin/` for system-wide access:
```bash
sudo cp scripts/* /usr/local/bin/
sudo chmod +x /usr/local/bin/start-open-webui /usr/local/bin/check-open-webui /usr/local/bin/backup-open-webui
```

### start-open-webui

Starts the Open WebUI stack.

- Checks if `/mnt/vault` is mounted (required, exits if not)
- Checks if `/mnt/backup` is mounted (optional, shows warning if not)
- Runs `docker compose up -d`

### check-open-webui

Shows current status of the service.

- Mount status for `/mnt/vault` and `/mnt/backup`
- Docker containers status
- HTTP health check on `localhost:3000/health`

### backup-open-webui

Creates a backup of the Open WebUI database and files.

- Checks if `/mnt/vault` and `/mnt/backup` are mounted (required)
- Creates SQLite backup of `webui.db` to `/mnt/backup/open-webui/db/` (7-day retention)
- Syncs application data (uploads, vector DB) to `/mnt/backup/open-webui/data/` (mirror)
- Syncs Jupyter work files to `/mnt/backup/open-webui/jupyter/` (mirror)
- Ollama models are not backed up (large and re-downloadable)
- Logs to `/var/log/homelab-backup.log`

Cron (runs daily at 3:45 AM):
```bash
sudo crontab -e
# Add: 45 3 * * * /usr/local/bin/backup-open-webui
```
