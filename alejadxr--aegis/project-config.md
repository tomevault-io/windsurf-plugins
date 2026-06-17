---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is AEGIS (v1.5)

Autonomous cybersecurity defense platform with four modules: Surface (attack surface management), Response (autonomous incident response), Phantom (live honeypot operations), and Deception (Honey-AI campaign decoys, Enterprise tier). Deployed on Mac Pro (YOUR_SERVER_IP) as the **sole defense brain**, protecting real services. AEGIS owns iptables locally — there is no external firewall agent in production.

v1.5 key additions: AI-offline mode (`AEGIS_AI_MODE=offline`), real firewall execution (`AEGIS_REAL_FW=1`), 122 Sigma rules + 5 chain rules hot-reload, unified design system.

### Topology
- **Raspberry Pi 5 + Hailo-10H** (`100.93.30.20`) = network gateway for the Mac Pro (Tailscale relay + routing). The Pi is part of the perimeter, not a separate defense node.
- **Mac Pro** = AEGIS brain (`cayde6-api`, `cayde6-frontend` via PM2). Owns detection, response, blocking, deception, threat intel.
- The optional `firewall-agent/` directory in this repo is a portable iptables service (port 8765). It is **not deployed**; AEGIS_FIREWALL_URL is intentionally unset so AEGIS handles iptables in-process.

## Commands

### Local Development
```bash
cd backend && source venv/bin/activate && python -m uvicorn app.main:app --reload --port 8000  # Backend
cd frontend && npm run dev          # Frontend (localhost:3000)
cd frontend && npm run build        # Build check (zero TS errors required)
```

### Production (Mac Pro)
```bash
# SSH
ssh $AEGIS_SSH_USER@$AEGIS_HOST_IP

# PM2 management
pm2 restart aegis-api              # Backend on port 8000
pm2 restart aegis-frontend         # Frontend on port 3007 (uses npx next start)
pm2 logs aegis-api --lines 30 --nostream

# Deploy frontend
cd /path/to/aegis/frontend && npm run build
rsync -avz --delete --exclude='node_modules' --exclude='.git' --exclude='.env' frontend/ $AEGIS_SSH_USER@$AEGIS_HOST_IP:~/AEGIS/frontend-prod/
# Then: pm2 restart aegis-frontend

# Deploy backend
rsync -avz --exclude='__pycache__' --exclude='venv' --exclude='*.db' --exclude='.env' backend/ $AEGIS_SSH_USER@$AEGIS_HOST_IP:~/AEGIS/backend/
# Then: pm2 restart aegis-api

# Trigger scan
curl -X POST -H "X-API-Key: YOUR_API_KEY" http://YOUR_SERVER_IP:8000/api/v1/surface/scan/now
```

### Operational notes
```bash
# AEGIS owns iptables on Mac Pro directly. There is no external firewall agent.
# AEGIS_FIREWALL_URL is intentionally unset; firewall_sync stays off.
# (Retired: Rasputin on Pi was the previous external firewall agent — not used anymore.)

# NEVER use lsof on Mac Pro — it hangs and creates zombie processes. Use netstat or ps aux instead.
```

## Architecture

### Backend (FastAPI + SQLAlchemy async + PostgreSQL)
- `app/main.py` — FastAPI app with lifespan (starts scanner, honeypots, log watcher, firewall setup, firewall sync)
- `app/core/openrouter.py` — Multi-model routing via OpenRouter (hunter-alpha main, 5+ free models)
- `app/core/ai_mode.py` — `AI_MODE` flag (`full`/`local`/`offline`). Set `AEGIS_AI_MODE=offline` to run without any AI API key.
- `app/core/auth.py` — API key auth middleware, demo client auto-seed
- `app/core/events.py` — In-memory async event bus (pub/sub)
- `app/core/guardrails.py` — Action approval system (auto_approve / require_approval / never_auto)
- `app/core/firewall_client.py` — Optional external firewall client (only used if `AEGIS_FIREWALL_URL` is set; currently unset in prod)
- `app/services/ai_engine.py` — Agentic AI: triage → classify → decide → execute → verify → audit. Falls back to local heuristics when `AI_MODE=offline`.
- `app/services/firewall_local.py` — Local system firewall: `MacOSFirewall` (pfctl), `LinuxFirewall` (iptables), `NoopFirewall` (default). Gated by `AEGIS_REAL_FW=1`. IP injection-safe via `ipaddress` validation.
- `app/services/rules_loader.py` — Loads and indexes 122 Sigma rules + 5 chain rules from `app/rules/`. Hot-reload via `reload()`. O(1) lookup by event type.
- `app/services/correlation_engine.py` — Evaluates rules using the type index. ~6× faster than linear scan on the default rule set.
- `app/services/scheduled_scanner.py` — APScheduler: full scan 2h, quick scan 30min, discovery 1h, adaptive alert mode
- `app/services/log_watcher.py` — Tails PM2 logs (macOS) or journalctl (Linux). Controlled by `AEGIS_MONITORED_APPS`. 11 internal source markers prevent self-detection.
- `app/services/firewall_sync.py` — Optional sync with external firewall agent every 5 min (conditional on `AEGIS_FIREWALL_URL`; **disabled in prod** since AEGIS handles iptables locally)
- `app/modules/surface/` — nmap/nuclei subprocess wrappers, AI risk scoring, hardening checks
- `app/modules/response/` — Alert ingestion, AI analysis, active response (pfctl/iptables + 403 middleware + external firewall)
- `app/modules/phantom/` — SSH honeypot (paramiko, port 2222), HTTP decoy (aiohttp, port 8888), rotation engine. Jinja2 templates in `app/templates/honeypot/` for offline mode.

### Frontend (Next.js 14 + Tailwind + TypeScript)
- Design system: "Refined Dark Command" — Outfit font, Azeret Mono for data, zinc palette, #22D3EE cyan + #F97316 orange accents

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alejadxr/AEGIS](https://github.com/alejadxr/AEGIS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
