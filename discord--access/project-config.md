---
trigger: always_on
description: You are helping implement features and bug fixes in the **Access** codebase — a company-internal
---

You are helping implement features and bug fixes in the **Access** codebase — a company-internal
Okta access control portal. This document covers how the code is structured, key patterns,
gotchas, and security requirements for development.

If you notice that anything in this file conflicts with or is missing from the current codebase,
flag it to the user and suggest updating the relevant section.

> **Maintenance watermark:** this document was last reconciled against the codebase at `main`
> commit **`fb260a2`** (latest merged PR #495; includes the end-to-end async-SQLAlchemy
> migration #480/#481 and the `uv` + `ty` toolchain from #507). When refreshing it, review
> commits merged after this point and update the anchor.

## Companion docs — load on demand

This file holds the always-relevant rules. Deeper, area-specific material lives in sibling docs
under `.claude/` that are **not** auto-loaded — read the matching one when your task touches
that area:

- **Frontend / UI work** → `.claude/access-dev-frontend.md` — generated `src/api/` client,
  React Query v5, layout & design principles.
- **Running tests or writing migrations** → `.claude/access-dev-testing.md` — the `uv`/`ruff`/`ty`
  toolchain, Alembic migrations and CI, factories, the Okta sandbox.
- **Syncer or notification-plugin work** → `.claude/access-dev-sync.md` — the sync authority
  model and notification cadence.

## Stack

**Backend:** FastAPI + Pydantic v2 + **async** SQLAlchemy 2.0 (asyncio) + Alembic
**Frontend:** React 18 + TypeScript, Vite, MUI, React Query
**Auth:** Authlib (OIDC or Cloudflare Access JWT — operator's choice)
**Plugins:** pluggy
**Tooling:** `uv` (deps + venv, pinned to Python 3.13), `ruff` (lint/format), `ty` (type check)
**Tests:** pytest + Factory Boy (backend), vitest + React Testing Library (frontend)

## Open source constraints — read before making changes

The Access repo is open source under the Apache 2.0 License. Any change merged to `main` becomes
public. This means:

- **No operator-specific logic in the Access repo.** Don't hardcode one organization's Okta
  attributes, email formats, internal URLs, private-repo references, or anything tied to how a
  single operator runs Access. A change that only works for one operator's infrastructure does
  not belong upstream.
- **Changes should serve all operators.** Features and fixes should be general-purpose.
  Operator-specific behavior belongs in a plugin or in that operator's own private repo, not
  upstream.
- **No references to any operator's private code.** The Access repo must be self-contained.
- **Maintain Apache 2.0 License compliance.** Any dependency or code introduced must be compatible.
- **Prefer backwards compatibility for incremental changes.** Many organizations run Access.
  Breaking changes to the API, data model, or plugin interface affect them too — prefer additive
  changes and deprecation over breaking ones. That said, some changes (major framework migrations,
  significant redesigns) intentionally break backwards compatibility and that's acceptable. Use
  judgment: routine bug fixes and features should be additive; larger architectural changes can
  set a higher bar.

Operator-specific code (plugin implementations, container definitions, Kubernetes configs, and
deployment config) lives in the operator's own private repo, not here.

### The one carve-out: `.github/workflows/docker-image.yml`

This single workflow is a deliberate, pre-existing exception. It builds and pushes the release
image to Discord's own Google Artifact Registry, so it necessarily hardcodes the
`discord-access-prd` GCP project, Discord's workload identity provider, and the
`us-east1-docker.pkg.dev/discord-access-prd` registry path. It cannot be made general-purpose;
another operator running Access publishes to their own registry from their own pipeline.

Because the file is already operator-bound, operator-specific *release-pipeline plumbing* may be
added to it — for example the `if: failure()` step that alerts a Discord channel via the
`DISCORD_CI_ALERTS_WEBHOOK` secret. Two rules keep the carve-out contained:

- **The exception is this file only.** Every other workflow runs for all operators and on forked
  PRs, and must stay general-purpose and secret-free. Note especially the near-namesake
  `docker-build.yml`: it is the merge-requirement counterpart that verifies the image builds from
  a clean checkout, and its header documents that it is *deliberately* secret-free so forked-PR
  builds work. Adding operator-specific plumbing there would break exactly what it exists to
  guarantee. When in doubt about which of the two a change belongs in: pushing the release
  artifact is `docker-image.yml`; proving the build works is `docker-build.yml`.
- **Degrade to a no-op, never a failure.** Anything added here that depends on a Discord-only
  secret must skip cleanly when that secret is absent, since secrets don't propagate to forks.
  A fork's run should not gain a confusing extra error.

Treat growth of this carve-out as a judgment call worth surfacing, not a precedent to lean on.
Application logic never belongs here regardless; this covers CI plumbing only.

## Security and design goals


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [discord/access](https://github.com/discord/access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
