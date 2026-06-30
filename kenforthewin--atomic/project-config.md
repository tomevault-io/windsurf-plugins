---
trigger: always_on
description: This subtree contains documentation for Atomic. The public website syncs only `docs/manual/**`, so keep agent instructions and internal process notes outside `docs/manual/`. Treat the manual as user-facing product documentation, not internal notes. Every claim there must be checked against the codebase before publishing.
---

# Atomic Manual Documentation

This subtree contains documentation for Atomic. The public website syncs only `docs/manual/**`, so keep agent instructions and internal process notes outside `docs/manual/`. Treat the manual as user-facing product documentation, not internal notes. Every claim there must be checked against the codebase before publishing.

## Scope

- Update files in `docs/manual/**` when documenting installation, concepts, guides, self-hosting, API usage, mobile clients, MCP, or other user-visible product behavior.
- Do not put plans, architecture notes, or speculative future behavior in `docs/manual/`. Use `docs/plans/`, `docs/reference/`, or `docs/research/` instead.
- If a feature exists in code but has no clear home in the manual, add a page instead of hiding important behavior in an unrelated page.
- Keep the manual useful for real users: explain what to do, what should happen, how to verify it, and what to do when it fails.

## Website Sync

The Atomic website pulls manual docs from this repository at build/dev time:

- Website script: `../atomic-website/scripts/sync-docs.sh`.
- Source path: `docs/manual`.
- Website target path: `../atomic-website/src/content/docs`.
- `npm run sync-docs` in the website copies docs only if `src/content/docs` is missing.
- `npm run sync-docs:fresh` runs the same script with `--force`, removes the target, and copies a fresh tree.
- Without `ATOMIC_LOCAL_PATH`, the script shallow-clones `kenforthewin/atomic` from `main` or `ATOMIC_DOCS_BRANCH`.
- With `ATOMIC_LOCAL_PATH=/path/to/atomic`, the script copies from that local checkout instead of cloning.

Because the sync is a wholesale `cp -R docs/manual src/content/docs`, never put agent instructions, drafts, or non-public notes under `docs/manual/`.

## Source Of Truth

Verify docs against implementation before editing:

- Product architecture and domain behavior: `crates/atomic-core/src/lib.rs`, `crates/atomic-core/src/models.rs`, and focused modules such as `embedding.rs`, `search.rs`, `wiki/`, `agent.rs`, `canvas_level.rs`, `reports/`, `scheduler/`, and `ingest/`.
- REST routes and request/response behavior: `crates/atomic-server/src/routes/mod.rs`, the individual files in `crates/atomic-server/src/routes/`, and the generated OpenAPI annotations in those files.
- API explorer and OpenAPI URL: `GET /api/docs/openapi.json` and `/api/docs` from `crates/atomic-server/src/main.rs`.
- Frontend command names, argument transforms, and event subscriptions: `src/lib/transport/command-map.ts`, `src/lib/transport/event-normalizer.ts`, `src/stores/`, and the relevant components in `src/components/`.
- CLI flags and environment variables: `crates/atomic-server/src/config.rs`, `Dockerfile`, `server.dockerfile`, `docker-compose*.yml`, `package.json`, and `src-tauri/`.
- AI provider settings and defaults: `crates/atomic-core/src/settings.rs` and `crates/atomic-core/src/providers/`.
- MCP behavior: `crates/atomic-server/src/mcp/`, `crates/mcp-bridge/`, and the MCP integration UI under `src/components/settings/`.
- Mobile behavior: `mobile/ios/`, `mobile/android/`, `capacitor.config.ts`, and any shared HTTP API expectations in `src/lib/transport/`.

## Documentation Workflow

1. Inventory affected docs with `find docs/manual -type f | sort` and search for existing coverage with `rg "<feature-or-route>" docs/manual`.
2. Trace the implemented behavior end to end. For user-facing features, follow UI/store/transport to server route to `atomic-core`. For API docs, start at `routes/mod.rs`, then inspect the route handler and request/response types.
3. Compare docs to code and list mismatches before editing. Look for missing prerequisites, outdated command flags, undocumented settings, stale endpoint paths, incorrect defaults, missing events, and unsupported platforms.
4. Edit for completeness, not word count. Prefer a concrete setup path, exact commands, expected result, troubleshooting, and cross-links over broad product claims.
5. Add or update adjacent pages when needed. A feature spanning UI, REST, and background processing usually needs both a user guide and API/reference coverage.
6. Re-run targeted checks. At minimum, run `rg` searches for renamed routes/settings and inspect links you changed. If code changed too, run the relevant `cargo check`, `npm`, or iOS command.

## Quality Bar

Good manual pages are specific, verifiable, and task-oriented:

- Start with what the feature is for and when to use it.
- State prerequisites: server running, token required, provider configured, database selected, platform limits, network access, or model availability.
- Use exact commands and endpoint paths. Prefer copyable `curl`, `cargo`, `docker`, `npm`, or `xcodebuild` examples over prose.
- Show request bodies and important response fields for API workflows.
- Explain background behavior that affects user expectations, especially async embedding/tagging, WebSocket events, scheduled jobs, feed polling, and multi-database scope.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenforthewin/atomic](https://github.com/kenforthewin/atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
