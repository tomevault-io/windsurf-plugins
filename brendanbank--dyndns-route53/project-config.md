---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Dynamic DNS web service that implements the DynDNS v2 protocol (`/nic/update` and `/nic/delete` endpoints). It accepts DNS update and delete requests via HTTP and manages records through pluggable backends: AWS Route53 (via boto3), BIND nsupdate (via dnspython TSIG), and Hetzner Cloud DNS (via REST API). Designed to work with OPNsense's DynDNS client and other DynDNS v2-compatible clients.

Supports multi-user operation with global domains (admin-managed), per-domain backends with shared credentials, user-owned hostnames, and a Bootstrap 5 web UI for administration.

## Running

**Docker (pre-built image):**
```
docker compose up -d
```
Pulls `ghcr.io/brendanbank/dyndns-route53:latest` from GHCR. Traefik handles TLS; serves on HTTPS (port 443 by default, configurable via `HTTPS_PORT`). Database and admin user are created automatically on first boot.

**Docker (local build):**
```
docker compose up --build
```

**Local development:**
```
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env   # edit with SECRET_KEY, FERNET_KEY, ADMIN_PASSWORD
.venv/bin/python dyndns.py
```
Runs Flask dev server on `0.0.0.0:8080`. Web UI at `http://localhost:8080/admin/login`.

**Testing an update:**
```
./tests/dyndns-client-test.sh  # tests against HOST from .env
```

**Generate a bcrypt password hash:**
```
python3 getpwd.py [optional-plaintext-password]
```

**Backup databases:**
```
./scripts/backup.sh                    # Docker (default)
./scripts/backup.sh --local            # local install
./scripts/backup.sh --restore ./backups/  # restore from latest backup
```

## Architecture

### Application Factory

`dyndns.py` uses Flask's application factory pattern (`create_app()`). It initializes:
- Flask-SQLAlchemy (`db`) with dual binds: main DB (`instance/dyndns.db`) and events DB (`instance/events.db`)
- Flask-Login for web session auth
- Flask-WTF CSRF protection (exempted for `/nic/update` and `/nic/delete`)
- Two blueprints: `nic_update_bp` (DynDNS API) and `web_bp` (admin UI)

Gunicorn uses the factory syntax `dyndns:create_app()` to create the app at worker startup.

### Data Model

```
User 1---* Hostname *---* DomainBackend *---1 Domain
                              1---* BackendConfig
```

- **Domain** — global zone managed by admin (e.g. `dyn.bgwlan.nl`). Has backends and hostnames.
- **DomainBackend** — backend config for a domain (aws/nsupdate/hetzner). Unique constraint on `(domain_id, backend_type)`. Has credentials via BackendConfig.
- **Hostname** — user-owned FQDN, globally unique (e.g. `myhost.dyn.bgwlan.nl`). Belongs to one domain and one user. Has configurable TTL (30–86400s, default 60). Stores last known IPv4/IPv6. Has optional many-to-many relationship with backends via `hostname_backends` table.
- **BackendConfig** — per-backend encrypted key-value pairs (e.g. `aws_access_key_id`). FK to `domain_backends`. Values Fernet-encrypted.
- **User** — username, bcrypt password hash, role (`admin`/`user`), active flag, TOTP secret, `web_login` flag (controls web UI access independently of API access). Has hostnames.
- **Event** — DNS update audit log (separate SQLite bind `events`). Records user, hostname, IP, backend, response.
- **RateLimitConfig** — global defaults or per-user overrides for `requests_per_minute` / `requests_per_hour`. `is_global=True` row is the fallback; per-user rows have `user_id` set.
- **RateLimitEvent** — timestamped request log used by the rate limiter. Pruned automatically to the last hour.
- **HealthCheck** — result of the most recent health check run per hostname/record-type. Cleared before each run.
- **HealthCheckConfig** — single-row config: `enabled` flag and `check_interval_minutes`.

### Request Flow

`GET /nic/checkip` — unauthenticated. Returns the client's IP as seen by the server. HTML by default; `?format=plain` returns just the IP.

`GET /nic/update` authenticates via HTTP Basic Auth (or query params) against the `users` table. For each hostname in the request:
1. Check rate limit via `check_rate_limit()` — returns `abuse` if exceeded
2. Look up `Hostname` record by exact name match + user ownership
3. If not found → `nohost`
4. Get backends for the hostname via `Hostname.get_backends()` — returns hostname-specific backends if configured, otherwise all domain backends
5. For each backend: get Fernet-encrypted credentials, create account via `AccountFactory`, call `createrecords()` passing the hostname's TTL
6. Log one `Event` per backend attempt
7. Aggregate result: `good` if any backend succeeded, `nochg` if all unchanged, error otherwise

The `updatetype` parameter is deprecated and ignored — all backends for the domain are updated automatically.

`GET /nic/delete` uses the same auth and hostname lookup. If `myip` is provided, only the matching record type (A or AAAA) is deleted; if omitted, both A and AAAA records are deleted. Calls `deleterecords()` on each backend. Events are logged with `event_type='dns_delete'`.

### Plugin System (`lib/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brendanbank) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
