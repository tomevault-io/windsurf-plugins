---
trigger: always_on
description: Yapit TTS - Open-source text-to-speech platform for reading documents, web pages, and text.
---


## Project Overview

Yapit TTS - Open-source text-to-speech platform for reading documents, web pages, and text.

**GitHub**: https://github.com/yapit-tts/yapit (public)
**CLI**: `../yapit-cli/` ([GitHub](https://github.com/yapit-tts/yapit-cli), [PyPI](https://pypi.org/project/yapit/)) — `uvx yapit` for markdown export, document creation from URLs/files

### Core

- `docs/architecture.md` has mermaid diagrams of the overall architecture and sequence flows.
- [[tts-flow]] — Audio synthesis pipeline: WebSocket protocol, Redis queues, pull-based workers, caching. Read for TTS bugs, latency issues, worker scaling.
- [[document-processing]] — How content becomes blocks: input paths (text/URL/file), Gemini extraction, YOLO figure detection, markdown parsing, block splitting. Read for document upload bugs, extraction failures, rendering issues.
- [[frontend]] — React architecture, component hierarchy, chrome devtools MCP workflows. Read for UI work, frontend debugging.
- [[features]] — User-facing capabilities: sharing, JS rendering, etc.

### Operations

- [[migrations]] — Alembic workflow, MANAGED_TABLES filter, seed data. Read before any DB schema changes.
- [[vps-setup]] — Production server config, Traefik, nginx reverse proxy routing, debugging. **READ BEFORE ANY SSH/prod commands** (DB creds, container names, security rules). Read for prod issues or nginx config changes (`frontend/nginx.conf`).
- [[infrastructure]] — Docker compose structure (base/dev/prod layers), worker services, config change checklist. Read for deployment issues or adding services.
- [[ci]] — GitHub Actions pipeline (~10 min), integration test setup, workflow dispatch options, debugging failures. Read when CI breaks or when changes might affect it (Dockerfiles, compose, Makefile, env files, project structure, test setup).
- [[env-config]] — Secrets management, .env files, sops encryption.
- [[dev-setup]] — **READ BEFORE TESTING.** Test commands, fixture gotchas, uv/pyproject structure. Tests WILL fail without proper env setup.
- [[dependency-updates]] — Version-specific gotchas, license checking. Always read before updating/adding any dependencies.
- [[metrics]] — TimescaleDB pipeline, event types, dashboard, health reports. Read when adding/modifying metrics.
- [[logging]] — Loguru JSON logging configuration.

**Testing:** You can claim "all tests pass" if and only if `make test-local` passes (after `make dev-cpu` if you made backend changes).

### Other domains

- [[stack-auth]] — Self-hosted auth provider: architecture, self-hosted gotchas, dev setup, upgrade pitfalls. Read for any Stack Auth work.
- [[auth]] — Gateway-level auth modes, anonymous flow, account deletion.
- [[stripe-integration]] — Token-based billing, waterfall consumption, Stripe SDK gotchas, webhook handling. Read for billing bugs, subscription issues.
- [[security]] — SSRF (Smokescreen proxy), auth trust boundaries, anonymous sessions, frontend XSS architecture, infrastructure hardening, billing security. Read for any security-related work.
- [[licensing]] — Dependency license checking.

### Notes for distill agents

- Upon any metrics/logging changes, update the monitoring agent prompt in `scripts/report.sh`.
- In addition to knowledge, make sure `docs/architecture.md`, `README.md`, etc. are up-to-date.

## Development

- **ALWAYS ask user before stopping/restarting Docker services** — they may be actively testing
- **Backend changes require restart** — If you modify backend Python code, tell user: "Backend code changed - please restart with `make dev-cpu`"
- Running integration tests / "validating everything still works" makes no sense if you applied backend changes, without running make dev-cpu to rebuild it first.
- Unit tests work without restarting the gateway. Run `make test-unit` for the full suite or `uv run pytest tests/path/to/test.py` for specific files.
- If you can debug something without ssh access, e.g. by running make sync-logs and inspecting the logs and metrics locally, do that instead, since ssh requires user approval / unnecessary friction.
- NEVER push in the same Bash tool call as git committing. Generally, NEVER push on your own except being explicitly told to push.
- The pre-commit hook runs ruff (lint + format), ty type checks, and frontend build — no need to run these separately right before committing.
- **Anonymous auth for API testing** (selfhost/dev): `curl -H "X-Anonymous-ID: test123" -H "X-Anonymous-Token: $(python3 -c "import hmac,hashlib; print(hmac.new(b'change-me-to-a-random-string', b'test123', hashlib.sha256).hexdigest())")" http://localhost:8000/v1/...` — the secret is `ANONYMOUS_SESSION_SECRET` from `.env.selfhost`.

## VPS SSH Permissions

**⚠️ CRITICAL: Destructive operations on prod require explicit user confirmation.**

Before running ANY of these on the prod VPS, get a literal "YES" from the user:
- `docker stop/restart/rm/kill`
- `docker exec` (can modify container state)
- Any file writes (`rm`, `mv`, `echo >`, etc.)
- Database modifications (anything other than SELECT)
- Service restarts, config changes

Ask explicitly: "I need to run `[exact command]` on prod. This will [effect]. Type YES to confirm."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yapit-tts/yapit](https://github.com/yapit-tts/yapit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
