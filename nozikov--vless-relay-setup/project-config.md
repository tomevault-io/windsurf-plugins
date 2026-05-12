---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Two-server VPN deployment automation: relay (entry node) + exit (exit node) using VLESS + XTLS-Reality + XHTTP, managed via 3X-UI panels. Pure Bash scripts, no frameworks.

## Architecture

```
User → Relay server (3X-UI + embedded XRAY, port 443)
         → VLESS Reality XHTTP inbound (sniffing: routeOnly)
         → fragment outbound (splits TLS ClientHello for DPI bypass)
         → proxy-exit outbound (VLESS Reality XHTTP, dialerProxy: fragment)
              → Exit server (standalone XRAY, port 443)
                   → routing: geoip:private → block
                   → internet (freedom outbound, domainStrategy: UseIP)

SelfSteal mode (optional):
  XRAY Reality dest → /dev/shm/caddy.sock (xver=1, PROXY protocol)
  DPI probe → Caddy (unix socket) → real website with valid cert
  DNS A-record → server IP (eliminates IP/SNI mismatch)
  Caddy also reverse-proxies 3X-UI panel and subscriptions (relay only)

CDN Fallback (optional, requires SelfSteal):
  Symmetric mode:
    Client → Cloudflare CDN → Exit server:443
      → XRAY Reality → Caddy (unix socket) → 127.0.0.1:cdn_port
      → VLESS XHTTP inbound (packet-up mode) → internet
  Asymmetric mode:
    Upload: Client → Cloudflare CDN → Exit (same XHTTP inbound as symmetric)
    Download: Client → Exit:443 (Reality XHTTP direct, same main inbound relay uses)
  Separate domain on Cloudflare (Proxied, SSL: Full)
  Exit: Caddy routes CDN path to local XRAY XHTTP inbound
  Relay: sub-proxy appends CDN VLESS links (symmetric + asymmetric) to subscriptions

Hysteria 2 (optional, requires SelfSteal):
  Client → Exit:UDP_RANGE (Hysteria 2 + Salamander obfuscation)
  Standalone binary alongside XRAY, independent systemd service
  Auth: exit UUID as password, TLS cert: copied from Caddy's Let's Encrypt
  Config: /etc/hysteria/config.yaml, certs: /etc/hysteria/certs/
  Port hopping: native listen on UDP range (e.g. 34821-35821)
  Masquerade: reverse proxy to SelfSteal site
  Link added to subscription via sub-proxy HYSTERIA_LINK env var
```

**Exit server**: XRAY runs as systemd service, config in `/usr/local/etc/xray/config.json`.
**Relay server**: 3X-UI manages its own XRAY process, config stored in SQLite at `/etc/x-ui/x-ui.db`.

Setup order is always exit first, then relay (relay needs exit server's keys/UUID).

## Entry Points

- `scripts/setup.sh` — router: delegates to setup/update/uninstall scripts. Passes extra args (`--force`, `--upgrade`)
- `scripts/setup-exit.sh` — exit server orchestration. Refuses to run if already configured (use `--force` to override)
- `scripts/setup-relay.sh` — relay server orchestration (complex, DB-driven). Same `--force` guard
- `scripts/update-exit.sh` — update exit config from latest codebase, preserving keys/UUID. `--upgrade` to update binaries
- `scripts/update-relay.sh` — update relay template + patch inbound sniffing, preserving clients. `--upgrade` to update 3X-UI
- `scripts/uninstall.sh` — teardown with `--force` and `--purge-certs` flags

## Library Modules (`scripts/lib/`)

All sourced via `BASH_SOURCE` from orchestration scripts:

- `common.sh` — logging (`log_info/ok/warn/error`), `prompt_input`, `prompt_password`, validation (`validate_domain`, `check_domain_dns`), random generation, `PROJECT_VERSION` from `VERSION` file
- `security.sh` — SSH hardening (custom port support), UFW, fail2ban
- `caddy.sh` — Caddy installation, Caddyfile generation, static site content, systemd dependency, sub-proxy setup, uninstall (SelfSteal mode only)
- `reality.sh` — Reality key generation, destination site selection
- `xray.sh` — XRAY installation, exit server JSON config (including optional CDN XHTTP inbound with padding)
- `3xui.sh` — 3X-UI install/configure, SQLite operations, SSL certs, inbound/template management
- `verify.sh` — post-setup smoke tests (services, ports, connectivity)
- `hysteria.sh` — Hysteria 2 installation, YAML config generation, cert copy from Caddy, restart, uninstall
- `sub-proxy.py` — subscription proxy: sits between Caddy and 3X-UI subscription, appends CDN VLESS + Hysteria 2 links. Passes through HTML pages (QR codes) for browsers, modifies only base64 responses for apps

## Critical Patterns

### 3X-UI Database Timing

3X-UI holds an in-memory copy of its SQLite DB. On shutdown it writes memory → DB, overwriting external changes. The mandatory pattern is:

```bash
x-ui stop          # Flush memory to DB
# ... modify DB with sqlite3 / xui_db_set ...
x-ui start         # Load fresh state from DB
```

`xui_db_set()` in `3xui.sh` handles upsert for the `settings` table with automatic SQL escaping. Complex operations (inbounds) use direct `sqlite3` calls with manual escaping.

### 3X-UI Inbound Normalization

After INSERT into `inbounds` table, 3X-UI strips fields on first restart: `subId`, `realitySettings.settings` (publicKey/fingerprint). The workaround is a two-restart cycle:

1. Insert full inbound → restart (3X-UI normalizes/strips)
2. Patch stripped fields back with `jq` → restart (xray picks up patched config)

See `patch_3xui_relay_inbound()` and `create_3xui_relay_inbound()` in `3xui.sh`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nozikov/vless-relay-setup](https://github.com/nozikov/vless-relay-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
