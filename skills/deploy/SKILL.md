---
name: deploy
description: "Deploy OpenClaw to production: Fly.io, Docker, VPS, and self-hosted setups. Use when deploying, containerizing, or setting up production environments."
metadata:
  {
    "openclaw":
      {
        "emoji": "🚀",
        "requires": { "anyBins": ["flyctl", "docker", "openclaw"] },
      },
  }
---

# Deploy OpenClaw

Production deployment patterns for OpenClaw.

## Fly.io (Recommended)

### Public Deployment

```bash
# Deploy with public endpoint
fly launch --config fly.toml
fly deploy
```

The `fly.toml` in the repo root provides a standard config with public HTTP ingress.

### Private Deployment (No Public IP)

Use `fly.private.toml` for zero public exposure:

```bash
fly launch --config fly.private.toml
fly deploy --config fly.private.toml
```

Access via:
```bash
# SSH tunnel
fly proxy 3000:3000 -a <app-name>

# WireGuard (persistent)
fly wireguard create
```

### Fly.io Essentials

```bash
# Check status
fly status -a <app-name>

# View logs
fly logs -a <app-name>

# SSH into container
fly ssh console -a <app-name>

# Scale resources
fly scale vm shared-cpu-2x --memory 2048 -a <app-name>
```

## Docker

### Build

```bash
docker build -t openclaw:latest .
```

### Run (Secure Defaults)

```bash
docker run -d \
  --name openclaw \
  --cap-drop=ALL \
  --read-only \
  --tmpfs /tmp \
  -v openclaw_data:/data \
  -e NODE_ENV=production \
  -e OPENCLAW_STATE_DIR=/data \
  -p 3000:3000 \
  openclaw:latest \
  node dist/index.js gateway --allow-unconfigured --port 3000
```

Security checklist:
- `--cap-drop=ALL` (drop all Linux capabilities)
- `--read-only` (immutable root filesystem)
- Non-root user in Dockerfile (`USER node`)
- No host Docker socket mounting
- Explicit port exposure only

### Docker Compose

```yaml
version: "3.8"
services:
  openclaw:
    build: .
    cap_drop: [ALL]
    read_only: true
    tmpfs: [/tmp]
    volumes:
      - openclaw_data:/data
    environment:
      - NODE_ENV=production
      - OPENCLAW_STATE_DIR=/data
    ports:
      - "3000:3000"
    restart: unless-stopped

volumes:
  openclaw_data:
```

## Self-Hosted VPS

### Setup

```bash
# Install Node.js 22+
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt install -y nodejs

# Install OpenClaw
sudo npm install -g openclaw@latest

# Create state directory
sudo mkdir -p /opt/openclaw/data
sudo chown $(whoami) /opt/openclaw/data

# Run gateway
OPENCLAW_STATE_DIR=/opt/openclaw/data openclaw gateway run --bind lan --port 3000
```

### Systemd Service

```ini
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=openclaw
Environment=NODE_ENV=production
Environment=OPENCLAW_STATE_DIR=/opt/openclaw/data
ExecStart=/usr/bin/openclaw gateway run --bind lan --port 3000
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Reverse Proxy (nginx)

```nginx
server {
    listen 443 ssl;
    server_name openclaw.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/openclaw.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Environment Variables

Required for production:

```bash
NODE_ENV=production
OPENCLAW_STATE_DIR=/data        # Persistent storage
NODE_OPTIONS="--max-old-space-size=1536"  # Memory limit
```

Channel-specific (set as needed):
```bash
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_WHATSAPP_FROM=...
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...
```

> Never commit `.env` files. Use secrets management (Fly secrets, Docker secrets, etc.)
