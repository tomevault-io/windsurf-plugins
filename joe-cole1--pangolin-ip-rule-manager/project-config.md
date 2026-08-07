---
trigger: always_on
description: `pangolin-ip-rule-manager` bootstraps network trust for devices that cannot authenticate
---

# CLAUDE.md

## What this project is

`pangolin-ip-rule-manager` bootstraps network trust for devices that cannot authenticate
through SSO. A family member visits a trigger URL on their phone (already authenticated
via Pangolin). The app reads their public IP, adds a temporary bypass rule in Pangolin
for specific protected resources (e.g., Jellyfin), and optionally adds the IP to the
CrowdSec allowlist. A TV on the same network can then reach Jellyfin without going
through SSO. Rules expire automatically via a configurable TTL.

This runs in production with no staging environment on an Oracle VPS alongside Pangolin,
Traefik, CrowdSec, and Portainer. Pangolin going down means all services go offline.
Every change is high-stakes.

---

## Repository structure

```
pangolin-ip-rule-manager/
├── app.py                   # Entry point: config, state, cleanup loop, target coordination
├── request_handler.py       # HTTP handler (factory pattern): IP extraction, routing, HTML pages
├── pangolin_connector.py    # All Pangolin API interaction: PangolinContext dataclass, retry logic
├── crowdsec_connector.py    # CrowdSec integration via cscli subprocess
├── requirements.txt         # Dev dependencies (pytest, ruff) — not used in the runtime image
├── Dockerfile               # Trimmed Python 3.14 Alpine runtime; docker-cli included for CrowdSec via docker exec
├── docker-compose.yml       # Reference compose file — actual deployment uses Portainer
├── config.env.sample        # Every environment variable with inline documentation
└── tests/
    ├── conftest.py          # sys.path setup for pytest
    ├── test_app.py          # App-level unit + integration tests
    └── test_crowdsec.py     # CrowdSec connector unit tests
```

There is no `Makefile` and no `pyproject.toml`. The application uses Python stdlib only —
no pip dependencies at runtime.

---

## Environment variables

All configuration is injected at runtime via environment variables.
**Never hardcode credentials.** See `config.env.sample` for the canonical reference.

### Hard required — startup aborts without these

| Variable | Description |
|---|---|
| `PANGOLIN_URL` | Pangolin Integration API base URL, e.g. `https://api.yourdomain.com` |
| `RESOURCE_IDS` | Comma-separated resource IDs to protect, e.g. `1,2` |
| `PROXY_SHARED_SECRET` | High-entropy secret configured in the app and the Pangolin resource's `X-Proxy-Secret` custom request header |

### Required for core functionality — startup warns and continues without these

| Variable | Description |
|---|---|
| `PANGOLIN_TOKEN` | Pangolin API token. Missing → prominent warning; all API calls raise immediately. |
| `ORG_ID` | Pangolin organisation ID. Missing → startup resource listing skipped. |

### Optional with defaults

| Variable | Default | Description |
|---|---|---|
| `LISTEN_PORT` | `8080` | HTTP listen port inside the container |
| `RETENTION_MINUTES` | `1440` | Minutes before an IP rule expires (default: 1 day) |
| `CLEANUP_INTERVAL_MINUTES` | `60` | How often the cleanup loop runs |
| `RULE_PRIORITY` | `0` | Pangolin rule priority |
| `RULES_CACHE_TTL_SECONDS` | `3600` | Cache TTL for Pangolin rule existence checks |
| `STATE_FILE` | `/data/state.json` | Persistent state path — must be on a mounted volume |
| `SITE_NAME` | _(empty)_ | Optional label shown in the HTML check-in and error pages |
| `UPDATE_ENDPOINT_ENABLED` | `false` | Enables `/update?ip=...` endpoint (disabled by default) |

### CrowdSec (all optional, disabled by default)

| Variable | Default | Description |
|---|---|---|
| `CROWDSEC_ENABLED` | `false` | Enable CrowdSec integration |
| `CROWDSEC_ALLOWLIST_NAME` | `pangolin-ip-rule-manager` | Allowlist name in CrowdSec |
| `CROWDSEC_CACHE_TTL_SECONDS` | `3600` | Cache TTL for CrowdSec allowlist membership checks |
| `CROWDSEC_CMD_PREFIX` | _(empty)_ | Command prefix, e.g. `docker exec crowdsec` |
| `CROWDSEC_CSCLI_BIN` | `cscli` | Path to the cscli binary inside the CrowdSec container |

---

## Running tests

```bash
pytest
```

Tests live in `tests/test_app.py` (app-level) and `tests/test_crowdsec.py` (CrowdSec
connector). `conftest.py` handles `sys.path`. All tests use `monkeypatch` to avoid
real subprocess or network calls — no live instance or credentials are needed to run
the suite.

---

## Linting

The project uses **ruff** (not flake8).

Check for violations:

```bash
ruff check .
ruff format --check .
```

Auto-fix and reformat:

```bash
ruff check --fix .
ruff format .
```

The CI workflow (`python-package.yml`) auto-fixes on push and opens a PR for any
changes. On a PR build it checks without fixing and fails if violations remain.

---

## Current state and next release

**Current release:** `v2.3.0` on `master`

No outstanding planned changes. Start the next release cycle by creating a fresh
`dev` branch from `master`.

---

## Behavioral rules for Claude

These apply to every session. Do not deviate without explicit instruction.

### Before generating anything

- **Read every file before suggesting changes.** Never work from memory on file
  contents. When a zip is uploaded, extract and read the relevant files first.
- **Verify before stating.** Do not assume version numbers, import names, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joe-cole1/pangolin-ip-rule-manager](https://github.com/joe-cole1/pangolin-ip-rule-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
