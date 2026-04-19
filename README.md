# Hermes Agent + WebUI Docker Compose

Self-hosted AI agent with a web interface, via Docker Compose.

## Overview

This setup runs two containers:

- **Hermes Agent** (`nousresearch/hermes-agent`) - The AI agent with memory, skills, and messaging integrations
- **Hermes WebUI** (`ghcr.io/nesquena/hermes-webui`) - A browser-based interface for interacting with Hermes

Both containers share volumes for state (`~/.hermes`) and workspace files.

## Prerequisites

- Docker 20.10+
- Docker Compose v2+
- At least one LLM API key (OpenRouter recommended)

## Quick Start

### 1. Clone and configure

```bash
git clone https://github.com/your-repo/hermes-w-ui-docker.git
cd hermes-w-ui-docker
cp .env.example .env
```

Edit `.env` and set at least one API key:

```bash
OPENROUTER_API_KEY=sk-or-v1-...
# or
OPENAI_API_KEY=sk-...
```

### 2. Start

```bash
docker compose up -d
```

### 3. Access

Open [http://localhost:8787](http://localhost:8787) in your browser.

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENROUTER_API_KEY` | Yes* | - | OpenRouter API key (recommended) |
| `OPENAI_API_KEY` | Yes* | - | OpenAI API key |
| `ANTHROPIC_API_KEY` | Yes* | - | Anthropic API key |
| `TELEGRAM_BOT_TOKEN` | No | - | Telegram bot token for messaging |
| `DISCORD_BOT_TOKEN` | No | - | Discord bot token for messaging |
| `HERMES_DEFAULT_MODEL` | No | `openrouter/anthropic/claude-sonnet-4-5` | Default model |
| `WEBUI_PORT` | No | `8787` | WebUI port on host |
| `HERMES_WEBUI_PASSWORD` | No | - | Password-protect the web UI |
| `UID` | No | `1000` | UID for file permissions |
| `GID` | No | `1000` | GID for file permissions |

*Set at least one provider API key.

## Volume Mounts

| Volume | Container Path | What Persists |
|--------|----------------|---------------|
| `hermes-home` | `/root/.hermes` (hermes), `/home/hermeswebui/.hermes` (webui) | Config, sessions, skills, memory |
| `hermes-workspace` | `/root/workspace` (hermes), `/workspace` (webui) | User workspace files |

## Security

This setup includes security hardening by default:

- **Read-only root filesystem** for hermes container
- **No new privileges** (`no-new-privileges:true`)
- **Tmpfs mount** for `/tmp` (hermes)
- **Resource limits** to prevent resource exhaustion
- **Health checks** on both services

For remote access, always enable password protection:

```bash
HERMES_WEBUI_PASSWORD=your-strong-password
```

## Health & Monitoring

Both services have health checks configured:

```bash
# View container health
docker compose ps

# View hermes logs
docker compose logs hermes

# View webui logs
docker compose logs webui
```

The webui will only start after hermes is healthy (via `depends_on` with `condition: service_healthy`).

## Updating

To update to the latest versions:

```bash
docker compose pull
docker compose up -d
```

## Troubleshooting

### Container won't start

Check logs:
```bash
docker compose logs hermes
docker compose logs webui
```

### WebUI shows "Agent not ready"

Wait for hermes to become healthy:
```bash
docker compose ps
```

### Permission issues

Ensure UID/GID in `.env` matches your user:
```bash
id $(whoami)
```

### Reset state

To reset all state (config, sessions, memory):

```bash
docker compose down -v
docker compose up -d
```

## Coolify Deployment

In Coolify:

1. Create a new "Docker Compose" deployment
2. Paste the contents of `docker-compose.yml` into the editor
3. Set environment variables in the env panel (from `.env.example`)
4. Deploy

The `UID` and `GID` should match the user that owns the mounted workspace directory on the host.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Host Machine                        │
│                                                         │
│   ┌─────────────────┐      ┌─────────────────┐          │
│   │   hermes-agent  │      │   hermes-webui  │          │
│   │  (container)    │◄────►│   (container)   │          │
│   │                 │      │                 │          │
│   │  /root/.hermes  │◄────►│/home/hermeswebui│          │
│   │  /root/workspace│      │  /.hermes       │          │
│   └────────┬────────┘      └────────┬────────┘          │
│            │                        │                    │
│      hermes-home               hermes-workspace          │
│            │                        │                    │
│   ┌────────┴────────────────────────┴────────┐          │
│   │              Named Volumes               │          │
│   └──────────────────────────────────────────┘          │
│                                                         │
│   Port 8787 (host) ──────► webui:8787                   │
└─────────────────────────────────────────────────────────┘
```

## License

MIT