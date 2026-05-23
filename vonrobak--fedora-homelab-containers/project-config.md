---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Philosophy

This is a digital sovereignty project — self-hosted infrastructure built for independence, not convenience. Every service replaces a cloud dependency. Every decision should:
- **Increase owner understanding** — prefer learning over turnkey solutions
- **Minimize external dependencies** — self-host when the trade-off is reasonable
- **Preserve data ownership** — all data lives on owned hardware with tested backups
- **Build layered resilience** — no single failure should take down the system

## Project Overview

A learning-focused homelab building production-ready, self-hosted infrastructure using Podman containers managed through systemd quadlets. Platform: Fedora Workstation 43.

**Current Services (29 containers, 16 service groups):**
- **Core Infrastructure:** Traefik (reverse proxy), CrowdSec (threat intel), Authelia + Redis (SSO + YubiKey MFA)
- **Applications:** Nextcloud + MariaDB + Redis (file sync), Vaultwarden (passwords), Jellyfin (media), Immich + PostgreSQL + Redis + ML (photos), Gathio + MongoDB (events), Homepage (dashboard)
- **Audio:** Audiobookshelf (audiobooks/podcasts), Navidrome (music streaming)
- **Downloads:** qBittorrent (torrent client)
- **Home Automation:** Home Assistant (smart home; Matter Server decommissioned 2026-05-18 pending matter.js successor, Plejd integration planned)
- **Monitoring:** Prometheus, Grafana, Loki, Alertmanager, Promtail, cAdvisor, Node Exporter, UnPoller, Alert Discord Relay

## Architecture

### Container Orchestration

- **Runtime:** Podman (rootless containers, UID 1000, SELinux enforcing)
- **Orchestration:** systemd quadlets (`~/containers/quadlets/` symlinked to `~/.config/containers/systemd/`)
- **Management:** `systemctl --user` commands
- **Service Discovery:** Traefik Docker provider with Podman socket

### Security Architecture

**Layered Middleware** (fail-fast principle):
```
Internet → Port Forward (80/443)
  → [1] CrowdSec IP Reputation (cache lookup - fastest)
  → [2] Rate Limiting (tiered: 100-200 req/min)
  → [3] Authelia SSO (YubiKey/WebAuthn + TOTP - phishing-resistant)
  → [4] Security Headers (applied on response)
  → Backend Service
```

Each layer is more expensive than the last. Reject malicious IPs before wasting resources on auth checks.

### Network Segmentation

8 networks for trust boundaries (see `AUTO-NETWORK-TOPOLOGY.md`):
- `reverse_proxy` - Internet-facing services + Traefik (default route for internet access)
- `monitoring` - Prometheus, Grafana, Loki, exporters (Internal=true, no internet)
- `auth_services` - Authelia + Redis (isolated auth backend)
- `media_services` - Jellyfin | `photos` - Immich stack
- `nextcloud` - Nextcloud + MariaDB + Redis
- `home_automation` - Home Assistant + Matter Server | `gathio` - Gathio + MongoDB

### Traefik Configuration

**Static config:** `config/traefik/traefik.yml` | **Dynamic config:** `config/traefik/dynamic/` (auto-reloads):
- `routers.yml` - All service routing rules
- `middleware.yml` - CrowdSec, rate limiting, auth, security headers
- `tls.yml` - TLS 1.2+
- `rate-limit.yml` - Tiered (global: 50/min, auth: 10/min, API: 30/min, public: 200/min)
- `security-headers-strict.yml` - CSP/HSTS

## Critical Conventions

### Traefik Routing (ADR-016)

**ALL Traefik routing is defined in `config/traefik/dynamic/routers.yml`, NEVER in container labels.**

Why: Separation of concerns (quadlets = deployment, Traefik = routing), centralized security auditing, fail-fast middleware ordering enforced consistently, single source of truth.

**To add a new route:** Edit `config/traefik/dynamic/routers.yml` — add router under `http.routers` and service under `http.services`. Traefik auto-reloads within 60s.

### Deployment Procedure

1. Create quadlet file in `~/containers/quadlets/` (NO Traefik labels)
2. Add route to `config/traefik/dynamic/routers.yml` with standard middleware chain:
   - **Default (Authelia):** `crowdsec-bouncer@file, rate-limit@file, authelia@file, security-headers@file` — used by qBittorrent, Homepage, Grafana, etc.
   - **Native auth** (Jellyfin, Nextcloud, Immich, Vaultwarden, HA, Navidrome, Audiobookshelf): `crowdsec-bouncer@file, rate-limit-public@file, compression@file, security-headers@file` — NO authelia
3. `systemctl --user daemon-reload && systemctl --user enable --now <service>.service`
4. Verify: `curl -I https://service.patriark.org`

Pattern-based deployment available via `homelab-deployment` skill (see `.claude/skills/homelab-deployment/`).

### Update Strategy

Most services use `:latest` tags. **Exceptions (pinned, manual upgrade only):**
- **Databases:** PostgreSQL, MariaDB (major version migrations required)
- **Immich:** Pinned to specific version (tight ML + postgres coupling)

Rollback: BTRFS snapshots enable instant recovery if an update breaks a service.

See ADR-015 for full rationale. Workflow: `scripts/update-before-reboot.sh` before DNF updates.

### Architecture Decision Records

**20 ADRs documenting architectural decisions** (see `docs/*/decisions/` for full details)

**Design-Guiding ADRs (affect future decisions):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vonrobak/fedora-homelab-containers](https://github.com/vonrobak/fedora-homelab-containers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
