---
trigger: always_on
description: **SquidC5 is a military-grade, security-first, AI-native C5 platform** - **Command - Control - Cognitive - Collaborative - Coordination** - under active development for **authorized** red team, penetration testing, and defensive security operations only.
---

# AGENTS.md - Instructions for AI Agents Working on SquidC5

## Classification & Mission

**SquidC5 is a military-grade, security-first, AI-native C5 platform** - **Command - Control - Cognitive - Collaborative - Coordination** - under active development for **authorized** red team, penetration testing, and defensive security operations only.

Treat every change as if the system will be deployed in high-threat environments:

- Prefer **secure defaults** over convenience
- Minimize attack surface and fingerprinting
- Never weaken auth, AI sandboxing, audit, or allow-lists without explicit human design review
- Assume hostile network exposure (internet-facing listeners, scanners, credential stuffing)

Unauthorized access assistance is out of scope. Do not help with illegal use.

## Project Stack

Primary language: Python 3.11+ - FastAPI - SQLite - Docker-first 
Operator CLI: `sc5` (also `squidc5-cli`) 
Ops UI: `/ops` (admin UI loaded only after server-side admin token check)

## Non-Negotiable Security Rules

1. **Secure by default**: New installs must ship hardened (no public docs/OpenAPI, no wildcard CORS, MCP off until enabled, exec probe on, false-shell filter on).
2. **External AI restriction**: MCP tools must remain allow-listed per token. No open-ended autonomous agent loops for external models.
3. **Admin AI / INKO shielding**: Never feed raw session output into LLM prompts without `sanitize_untrusted()`. Keep capabilities and chat tools allow-listed. Prefer offline/deterministic fallbacks when no LLM is configured. No open-ended autonomous agent loops.
4. **Determinism preference**: Templates, fixed prompts, single-step tools over free-form agentic planning.
5. **Audit everything**: Operator, MCP, Admin AI, feature toggles, and admin UI loads go through the policy engine / audit trail.
6. **No secrets in git**: Tokens, API keys, `data/`, `admin_token.txt`, `~/.config/squidc5/config.json` stay out of the repository.
7. **Port flexibility**: Never hard-require ports 80 or 443 - operators *may* use them.
8. **Admin UI isolation**: Admin-only HTML/JS must be served only after server validates an **admin** token (`/api/v1/ops/admin.js`). Non-admin clients must never receive admin control code.
9. **Public docs locked off**: `/docs`, `/redoc`, `/openapi.json` stay disabled. Feature flag `public_docs` is hard-forced `false`.
10. **Authorized use only**.

## Hardened Defaults (do not casually reverse)

| Control | Default |
|---------|---------|
| Public Swagger / OpenAPI | **OFF** (hard-locked) |
| CORS | **empty** (no `*`) |
| MCP external tools | **OFF** until **both** `SQUIDC5_MCP_ENABLED=true` and feature `mcp_enabled` |
| Shell exec probe | **ON** |
| False-shell filter | **ON** |
| Auto stage-2 stabilize | **OFF** (manual Stabilize or feature flag) |
| Health details | **minimal** (`{"status":"ok"}`) |
| Security headers | **ON** (nosniff, DENY frame, CSP, no-store) |
| Admin ops UI | **server-gated** by admin scope |
| Asymmetric key vault | **OFF** (`asym_keys`; private keys encrypted at rest, never returned) |

When adding features: **deny by default**, enable via admin feature flags or env after review.

## Development pipeline (mandatory)

Source of truth for this project: this repo’s **`AGENTS.md`**, **`docs/`**, and **`CONTRIBUTING.md`**.

### Git cycle (every change)

1. **Update main/master** - pull latest 
2. **Feature branch** - name for the change 
3. **Unit tests first** - describe expected behavior 
4. **Implement** code 
5. **Red-green-refactor** - all tests pass; clean up 
6. **Push branch** (never direct to main) 
7. **Open PR** 
8. **Wait for CI** - do not merge red 
9. **Merge** when green 
10. **Next change** - back to step 1 

### Prod after merge

```text
merge main -> CI builds Linux/Windows binaries -> GitHub Release published
 -> deploy Linux squidc5 binary ONLY (from Release assets or workflow Artifacts)
```

Releases: `https://github.com/SquidSec/SquidC5/releases` (created by CI job `github-release` on `master` only).

Public CI runs on **GitHub-hosted** runners (`ubuntu-latest`, `ubuntu-24.04-arm`, `windows-latest`; org self-hosted runners disallow public repositories). Prefer PRs into protected `master`. Releases include `squidc5-linux-x64` and `squidc5-linux-arm64` (Raspberry Pi 64-bit).

- **Never** commit/push straight to `main`/`master`
- **Never** rsync WIP source or `docker compose up --build` to prod
- **Never** deploy a feature-branch-only binary
- Docker is for **local/lab** only once binary prod path is active
- Full policy: see **Production deploy policy** under Deployment Knowledge

## Roadmap 2026-2027 (next work - prioritized)

Full detail: `docs/roadmap-2026-2027.md`. Agents plan work against this list; stay OPSEC-first, agents-on-rails.

| # | Focus | Notes |
|---|--------|--------|
| **1** | **Malleable / adaptive C2 profiles** | **TOP PRIORITY** - HTTP/S, DNS, WS; jitter; decoy; runtime switch |
| 2 | Advanced implant / beacon framework | Stagers, injection, memory-only, BOF-like; Win/Linux/macOS |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SquidSec/SquidC5](https://github.com/SquidSec/SquidC5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
