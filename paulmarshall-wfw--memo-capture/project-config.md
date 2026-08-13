---
trigger: always_on
description: <!-- current-project-agent-baseline-addendum -->
---

# AGENTS.md

<!-- current-project-agent-baseline-addendum -->
## Current Project Agent Baseline Addendum

This section carries the current All Standards project-agent baseline. Preserve stronger project-specific instructions elsewhere in this file.

- For any repo setup, maintenance, versioning, or stack-selection work, apply the engineering-project-standard skill from `~/.codex/skills/engineering-project-standard`.
- For any frontend UI design, scaffolding, review, or refinement work, apply the web-app-design-standard skill from `~/.codex/skills/web-app-design-standard`.
- For any Docker, container, image build, image publishing, registry push, or container release work, apply the docker-build-and-publish skill from `~/.codex/skills/docker-build-and-publish`.
- For browser automation, use Chrome unless the user explicitly asks for a different browser or Chrome is unavailable.
- Browser navigation and browser session changes are allowed when they are part of requested or expected verification, testing, debugging, or UI review work.
- Default to Build Mode unless the user explicitly asks for release behaviour.
- Never use `latest`.
- Always use numbered versions.
- When the project is in Git, prefer Git-derived traceability by default.
- When distribution beyond local or dev use is explicitly requested, require publishable images to support both `linux/amd64` and `linux/arm64`.
- Before adding or changing local ports, check `/Users/paulmarshall/Software Development/All Standards/local-port-registry.md`; after updating it, run `python3 "/Users/paulmarshall/Software Development/All Standards/scripts/check-local-port-registry.py"`.
- For technical build work, use Technical Build Logs in `docs/build-logs/YYYY-MM.md`; do not duplicate the same technical work in `docs/completed-tasks.md`.
- If `project-decisions.md` exists, review the relevant recorded decisions before making project changes, and preserve those decisions unless the user explicitly changes direction.


## Core Skill Policy

For any repo setup, maintenance, versioning, or stack-selection work, apply the engineering-project-standard skill from `~/.codex/skills/engineering-project-standard`.

For any frontend UI design, scaffolding, review, or refinement work, apply the web-app-design-standard skill from `~/.codex/skills/web-app-design-standard`.

For any Docker, container, image build, image publishing, registry push, or container release work, apply the docker-build-and-publish skill from `~/.codex/skills/docker-build-and-publish`.

For browser automation, use Chrome for all browser automation unless the user explicitly asks for a different browser or Chrome is unavailable.

## Broad Project Policy

Prefer explicit user intent over convenience defaults. Defaults may suggest values or preselect options, but they are not permission to mutate state, activate features, publish, overwrite files, commit, tag, release, install, delete, send, or navigate/change app or browser state unless the user explicitly chooses or requests that action.

- Default to Build Mode unless the user explicitly asks for release behaviour.
- Never use `latest`.
- Always use numbered versions.
- When the project is in Git, prefer Git-derived traceability by default.
- When the user explicitly asks for distribution beyond local or dev use, require publishable images to support both `linux/amd64` and `linux/arm64`.
- Do not let container distribution work overwrite or weaken existing Codex instructions in this file.

## Repo Workflow Notes

Verified from the current workspace scaffold.

- Install command: `npm install`
- Development command: `npm run dev:desktop`, `npm run dev:api`, and `npm run dev:worker`
- Test command: `npm test` (requires local Docker/Postgres because it runs the Postgres lane first)
- Real Postgres integration test command: `npm run test:postgres`
- Lint or typecheck command: `npm run typecheck`
- Build command: `npm run build`
- Full verification command: `npm run verify`

Dependencies may not be installed yet in a fresh checkout. Run `npm install` before executing npm scripts.

## Runtime Notes

- Product shape: Tauri desktop app with a web UI, backed by a TypeScript API and worker.
- Browser-only desktop dev URL: Vite default `http://localhost:5173` unless Vite prints another port.
- AppLauncher local web URL: `http://127.0.0.1:5177`, reserved in `/Users/paulmarshall/Software Development/All Standards/local-port-registry.md`.
- Tauri desktop dev URL: strict `http://127.0.0.1:5178` from `apps/desktop/src-tauri/tauri.conf.json`.
- API port: `MEMO_CAPTURE_API_PORT`, default `4788`.
- API base URL for desktop: `VITE_MEMO_CAPTURE_API_URL`.
- Data store: Postgres via `DATABASE_URL`.
- Artifact storage: S3-compatible object storage via `OBJECT_STORAGE_*` environment variables.
- Auth: OIDC-compatible provider via issuer, audience, client ID, and JWKS config.
- Background work: API and worker are separate commands; worker claims Postgres-backed processing jobs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulmarshall-wfw/memo-capture](https://github.com/paulmarshall-wfw/memo-capture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
