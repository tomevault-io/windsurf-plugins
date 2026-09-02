---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

```
ctf/
├── README.md
├── .gitignore
├── setup.sh          ← Interactive setup wizard (pulls images, writes manager/.env)
├── challenge/        ← BankingAI CTF (PHP + MySQL, 5 flags)
│   ├── docker-compose.yaml
│   ├── web/          ← PHP 8.2 Apache container
│   │   ├── Dockerfile
│   │   └── src/      ← Web root (live-mounted into container)
│   ├── db/           ← MySQL 8.0 init scripts
│   │   ├── bankingai.sql       ← Clean MySQL 8.0 schema + seed data
│   │   └── init_flags.sh       ← Injects FLAG_SQL_INJECTION into users table at DB init
│   └── scripts/      ← Manual multi-team bash helpers
├── task-1/           ← SWOCTS Task 1 (Python Flask, 3 flags, no MySQL)
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── app.py
│   ├── generate_artifacts.py
│   ├── requirements.txt
│   ├── static/
│   └── templates/
├── manager/          ← Flask web app for team registration + instance management
│   ├── docker-compose.yaml
│   ├── .env.example  ← Template — copy to .env and fill in values
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│   ├── config/
│   │   ├── bankingai.json   ← Flags + hints for BankingAI CTF
│   │   └── task1.json       ← Flags + hints for SWOCTS Task 1
│   └── templates/
└── .github/
    └── workflows/
        ├── publish-bankingai.yml   ← Publishes bankingai-web:latest to GHCR
        └── publish-task1.yml       ← Publishes task1-web:latest to GHCR
```

## Running the CTF Challenge (challenge/)

```bash
cd challenge

# Start single instance (first run takes ~30s for DB to init)
docker compose up --build -d

# Stop
docker compose down

# Full reset (wipes DB volume)
docker compose down -v && docker compose up --build -d

# Multi-team: each team gets its own containers on a separate port
bash scripts/add_team.sh <name> [port]   # auto-assigns port from 8000 if omitted
bash scripts/remove_team.sh <name>       # tears down + wipes DB volume
bash scripts/list_teams.sh              # show running teams and ports
```

The single-instance default is at **http://localhost** (port 80). Multi-team instances are at the auto-assigned port. The port is controlled by the `PORT` env var in `docker-compose.yaml` (`${PORT:-80}:80`).

## Quick Setup (recommended)

```bash
# Run the interactive wizard from the repo root:
bash setup.sh
# Then:
cd manager && docker compose up -d
```

The wizard:
1. Asks which CTF to run (BankingAI or SWOCTS Task 1)
2. Asks for HOST_IP
3. Auto-generates SECRET_KEY, ADMIN_TOKEN, FLAG_SECRET
4. Pulls the pre-built image from ghcr.io and tags it locally
5. Writes `manager/.env`

## Running the Manager Manually (manager/)

```bash
# 1. Copy the example env file and fill in your values
cp manager/.env.example manager/.env
# Edit manager/.env: set HOST_IP, SECRET_KEY, ADMIN_TOKEN, FLAG_SECRET,
#   CTF_COMPOSE_HOST_PATH, CTF_CHALLENGE_DIR, CTF_CONFIG_FILE, CTF_NAME

# 2. Build the challenge image (BankingAI example)
cd challenge && docker compose build

# 3. Start manager
cd manager && docker compose up --build -d
# Browse to http://localhost
```

**Key env vars in `manager/.env`:**
| Variable | Description |
|---|---|
| `CTF_COMPOSE_HOST_PATH` | Host path to the challenge docker-compose file |
| `CTF_CHALLENGE_DIR` | Host absolute path to the challenge directory |
| `CTF_CONFIG_FILE` | Container path to the JSON config (e.g. `/ctf/config/bankingai.json`) |
| `CTF_NAME` | Display name shown in page titles |
| `WEB_SERVICE_NAME` | Docker service name of the web container (default `web`) |
| `STARTUP_TIMEOUT` | Seconds to wait for web container ready (default `180`) |

`manager/.env` is gitignored — never commit it.

## Challenge Architecture

The CTF is a PHP employee portal ("BankingAI Cloud") backed by MySQL. Only port 80 is exposed.

**Flag locations and how they are set:**
| Flag env var | Where it appears |
|---|---|
| `FLAG_INSPECTED` | HTML comment in `products.php` (view source) |
| `FLAG_LOGIN` | Shown on `dashboard.php` after login |
| `FLAG_SQL_INJECTION` | Written as a `username` in the `users` DB table by `init_flags.sh` |
| `FLAG_USER_ESCALATION` | Rendered in `admin_subnav.php` nav link |
| `FLAG_FILE_UPLOAD` | Written to `/flag.txt` at container start (via `web/Dockerfile` CMD) |

**Intended exploit chain:**
1. `robots.txt` → `/staff-resources/new-employee-guide.txt` → credentials `ajohnson:Welcome2026`
2. Login → `dashboard.php` shows `FLAG_LOGIN`
3. `lookup.php` SQL injection (unsanitised `WHERE full_name LIKE '%$search%'`) → dump `users` table → get `FLAG_SQL_INJECTION` and admin password hash
4. Login as admin → `admin_subnav.php` shows `FLAG_USER_ESCALATION`
5. `admin_uploads.php` (no file type validation) → upload PHP webshell → execute → read `/flag.txt` = `FLAG_FILE_UPLOAD`
6. `FLAG_INSPECTED` is in the HTML source of `products.php` (can be found at any point)

## Customising Flags

Edit environment variables in `challenge/docker-compose.yaml` then rebuild. The `FLAG_SQL_INJECTION` value is picked up by `db/init_flags.sh` at DB initialisation — no SQL edits needed.

## Key Implementation Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Banjomenny/simpleCTF](https://github.com/Banjomenny/simpleCTF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
