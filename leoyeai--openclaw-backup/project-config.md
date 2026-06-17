---
trigger: always_on
description: Backup and restore all OpenClaw configuration, agent memory, skills, and workspace data. Part of the MyClaw.ai (https://myclaw.ai) open skills ecosystem — the AI personal assistant platform that gives every user a full server with complete code control. Use when the user wants to create a snapshot of their OpenClaw instance, schedule periodic backups, restore from a backup, migrate to a new server, download a backup file locally, upload a backup file from another machine, or protect against data
---


# MyClaw Backup

> **Built on [MyClaw.ai](https://myclaw.ai)** — the AI personal assistant platform that gives every user a full server with complete code control, networking, and tool access. This skill is part of the MyClaw.ai open skills ecosystem.

Backs up all critical OpenClaw data to a single `.tar.gz` archive and restores it to any OpenClaw instance. Includes a built-in HTTP server for browser-based backup management.

## ⚠️ Trust Boundary & Security Model

This skill handles **highly sensitive data**: bot tokens, API keys, channel credentials, session history. Understand the security model before use:

### What each script does
- **backup.sh** — reads `~/.openclaw/` and writes a `chmod 600` archive to disk. No network access.
- **restore.sh** — overwrites `~/.openclaw/` from an archive. Requires typing `yes` to confirm. Always run `--dry-run` first.
- **serve.sh / server.js** — starts a local HTTP server. Token is **mandatory** (refuses to start without one). Shell-execution endpoints (`/backup`, `/restore`) are **localhost-only** — remote access can only download and upload files, not trigger execution.
- **schedule.sh** — modifies your system crontab to run backup.sh on a schedule. Prints the cron entry before adding. Use `--disable` to remove.

### Access control summary
| Endpoint | Remote (token required) | Localhost only |
|---|---|---|
| GET /health | ✅ (no token) | — |
| GET /backups | ✅ | — |
| GET /download/:file | ✅ | — |
| POST /upload | ✅ | — |
| POST /backup | ❌ | ✅ |
| POST /restore | ❌ | ✅ |

### Best practices
- Never start the HTTP server without `--token`
- Never expose the HTTP server to the public internet without TLS
- Always run `restore.sh --dry-run` before applying a restore
- Store backup archives securely — they contain all credentials

## Dependencies

Requires: `node`, `rsync`, `tar`, `python3`, `openclaw` CLI (all standard on OpenClaw instances).

Check: `which node rsync tar python3 openclaw`

## Scripts

| Script | Purpose |
|---|---|
| `scripts/backup.sh [output-dir]` | Create backup (default: `/tmp/openclaw-backups/`) |
| `scripts/restore.sh <archive> [--dry-run] [--overwrite-gateway-token]` | Restore — **always dry-run first** |
| `scripts/serve.sh start --token TOKEN [--port 7373]` | Start HTTP server — **token required** |
| `scripts/serve.sh stop\|status` | Stop/check server |
| `scripts/schedule.sh [--interval daily\|weekly\|hourly]` | System cron scheduling |

> **Gateway token behavior (v1.6+):** By default, `restore.sh` preserves the new server's `gateway.auth.token` after restoring `openclaw.json`. This prevents the `"gateway token mismatch"` error in Control UI / Dashboard after migration. Use `--overwrite-gateway-token` only for full disaster recovery on the same server.

## What Gets Backed Up

See `references/what-gets-saved.md` for full details.

**Includes:** workspace (MEMORY.md, skills, agent files), openclaw.json (bot tokens + API keys), credentials, channel pairing state, agent config + session history, devices, identity, cron jobs, guardian scripts.

**Excludes:** logs, binary media, node_modules, canvas system files.

## Common Workflows

### Create backup

```bash
bash scripts/backup.sh /tmp/openclaw-backups
# → /tmp/openclaw-backups/openclaw-backup_TIMESTAMP.tar.gz (chmod 600)
```

### Restore — always dry-run first

```bash
# Step 1: preview what will change
bash scripts/restore.sh openclaw-backup_TIMESTAMP.tar.gz --dry-run

# Step 2: review the output, then apply
bash scripts/restore.sh openclaw-backup_TIMESTAMP.tar.gz
```

The restore script saves a pre-restore snapshot before overwriting anything.

### HTTP server — token is mandatory

```bash
# Token is required — server refuses to start without one
bash scripts/serve.sh start --token $(openssl rand -hex 16) --port 7373
# → http://localhost:7373/?token=<generated-token>
```

**Never share the URL on a public network without a reverse proxy + TLS.**

The Web UI provides: create backup, download `.tar.gz`, upload, dry-run preview, restore.

**HTTP API (all require token except /health):**
- `GET  /health`              — Health check (unauthenticated, read-only)
- `GET  /backups`             — List backups
- `POST /backup`              — Create backup
- `GET  /download/:filename`  — Download archive
- `POST /upload`              — Upload archive (multipart, field: `backup`)
- `POST /restore/:filename`   — Restore; add `?dry_run=1` to preview

### Migrate to a new instance

**Old machine:**
```bash
bash scripts/serve.sh start --token MYTOKEN --port 7373
```

**New machine (after installing OpenClaw):**
```bash
# Download
curl -O "http://OLD_IP:7373/download/openclaw-backup_TIMESTAMP.tar.gz?token=MYTOKEN"

# Always dry-run first
bash scripts/restore.sh openclaw-backup_TIMESTAMP.tar.gz --dry-run

# Apply

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeoYeAI/openclaw-backup](https://github.com/LeoYeAI/openclaw-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
