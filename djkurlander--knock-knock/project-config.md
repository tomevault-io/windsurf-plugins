---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Knock-Knock is a multi-protocol honeypot monitoring system that captures unauthorized login attempts on SSH (port 22), Telnet (port 23), and SMTP (port 587), and displays real-time attack data through a live web dashboard. It can be deployed via Docker or as two coordinated systemd services.

## Commands

### Service Management (Production)
```bash
# Restart all services
./restart.sh

# Reset all data and restart (blocklist is preserved)
./restart.sh --reset-all

# Reset blocklist only (deletes blocklist.txt + clears knock:blocked in Redis)
python monitor.py --reset-blocklist

# Individual service control
systemctl start|stop|restart|status knock-monitor knock-web

# Docker
docker compose up -d
docker compose down
docker compose logs -f
```

### Development (Direct Execution)
```bash
source .venv/bin/activate

# Individual honeypots (ports 22, 23, 587)
python ssh_honeypot.py
python telnet_honeypot.py
python smtp_honeypot.py

# Log monitor + geo-enricher — spawns all three honeypots as subprocesses
# Add --save-knocks to store individual knocks in SQLite (all protocols)
# Add --save-knocks=SIP,SMTP to store only specific protocols
python monitor.py

# Web server (HTTP, default port 8080 — Cloudflare Origin Rule proxies 80→8080)
python3 -m uvicorn main:app --host 0.0.0.0 --port 8080 \
  --proxy-headers --forwarded-allow-ips='*' --workers 2

# Web server (HTTPS, default port 8443)
python3 -m uvicorn main:app --host 0.0.0.0 --port 8443 \
  --ssl-keyfile certs/key.pem --ssl-certfile certs/cert.pem \
  --proxy-headers --forwarded-allow-ips='*' --workers 2
```

### Debugging
```bash
# Service logs (systemd)
journalctl -u knock-monitor -f
journalctl -u knock-web -f

# Service logs (Docker)
docker compose logs -f honeypot-monitor
docker compose logs -f web

# Database queries
sqlite3 data/knock_knock.db "SELECT * FROM knocks ORDER BY id DESC LIMIT 10;"

# Redis connectivity
redis-cli ping

# Check per-protocol feed lists
redis-cli llen knock:recent:ssh
redis-cli llen knock:recent:tnet
redis-cli llen knock:recent:smtp

# Watch for SMTP connections (even without AUTH — honeypot logs every connect)
journalctl -u knock-monitor -f | grep SMTP
```

## Architecture

```
SSH Attacker  → ssh_honeypot.py  (port 22)  ─┐
Telnet Attacker → telnet_honeypot.py (port 23) ─┼→ stdout → monitor.py
SMTP Attacker → smtp_honeypot.py (port 587) ─┘        (GeoIP, DB, Redis)
                                                              ↓
                                                  SQLite DB (data/) + Redis pub/sub
                                                              ↓
                                                   main.py (FastAPI, port 8080/8443)
                                                              ↓
                                               Browser WebSocket → Live Dashboard
```

**Two Services:**
- `monitor.py`: Spawns all three honeypots as subprocesses, merges their stdout via a shared `queue.Queue`, performs GeoIP lookups, updates SQLite intel tables, publishes to Redis. Individual knocks saved to per-protocol SQLite tables with `--save-knocks` (all) or `--save-knocks=SIP,SMTP` (selective). Honeypots check `knock:blocked` Redis set on each connection to reject blocked IPs instantly.
- `main.py`: FastAPI server with WebSocket endpoint `/ws`, subscribes to Redis, broadcasts to all connected browsers.

**Data Flow:**
- Monitor spawns honeypots as subprocesses and reads their stdout (both systemd and Docker)
- Each honeypot emits JSON: `{"type": "KNOCK", "proto": "SSH"|"TNET"|"SMTP", "ip": ..., "user": ..., "pass": ...}`
- Inter-service communication via Redis pub/sub channel `knocks_stream`
- Stats cached in memory, refreshed every 60 seconds and broadcast to all clients
- SQLite databases in `data/` directory for persistence

**Deployment modes:**
- **Docker:** `docker compose up -d` — monitor spawns all honeypots internally
- **Systemd:** Two unit files in `systemd/` — monitor spawns all honeypots internally

## Key Files

| File | Purpose |
|------|---------|
| `ssh_honeypot.py` | SSH honeypot (port 22) — legacy paramiko version (unused, kept as fallback) |
| `telnet_honeypot.py` | Telnet honeypot (port 23), raw socket with IAC negotiation |
| `smtp_honeypot.py` | SMTP honeypot (port 587), AUTH LOGIN + AUTH PLAIN |
| `monitor.py` | Spawns honeypots, GeoIP enrichment, DB writes, Redis publish |
| `main.py` | FastAPI server, `ConnectionManager`, `GlobalStatsCache`, WebSocket |
| `constants.py` | Shared protocol enum: `PROTO` dict and `PROTO_NAME` reverse lookup |
| `index.html` | Single-page dashboard with WebSocket client |
| `restart.sh` | Service orchestration (systemd and Docker) |
| `Dockerfile` | Single image for honeypot-monitor and web containers |
| `docker-compose.yml` | Docker deployment (Redis, honeypot+monitor, web) |
| `stats.py` | CLI utility for printing database statistics |
| `dbtool.py` | DB management: `--list-tables`, `--backup`, `--remove-knocks` |
| `extras/` | Optional utilities (Cloudflare UFW rules, texture generation, visitor reports) |

## Data Directory


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djkurlander/knock-knock](https://github.com/djkurlander/knock-knock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
