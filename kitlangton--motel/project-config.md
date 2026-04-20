---
trigger: always_on
description: - Install deps: `bun install`
---

# AGENTS.md

## Commands
- Install deps: `bun install`
- Run the TUI: `bun run dev` or `bun run start` (auto-ensures a managed
  OTLP daemon is running in the background so traces ingest while the TUI
  is up)
- Start the background daemon only: `bun run daemon` (same as `motel start`)
- Stop the managed daemon: `bun run stop`
- Daemon status JSON: `bun run status`
- Restart daemon + relaunch TUI: `bun run restart`
- Run the local server in the foreground (no daemon, no TUI): `bun run server`
- Run tests: `bun run test`
- Query services via CLI: `bun run cli services`
- Query traces via CLI: `bun run cli traces <service> [limit]`
- Query a span via CLI: `bun run cli span <span-id>`
- Query spans for one trace: `bun run cli trace-spans <trace-id>`
- Search spans via CLI: `bun run cli search-spans [service] [operation] [parent=<operation>] [attr.key=value ...]`
- Search traces via CLI: `bun run cli search-traces <service> [operation] [attr.key=value ...]`
- Query trace stats via CLI: `bun run cli trace-stats <groupBy> <agg> [service] [attr.key=value ...]`
- Query logs via CLI: `bun run cli logs <service>`
- Search logs via CLI: `bun run cli search-logs <service> [body] [attr.key=value ...]`
- Query log stats via CLI: `bun run cli log-stats <groupBy> [service] [attr.key=value ...]`
- Query logs for one trace: `bun run cli trace-logs <trace-id>`
- Query logs for one span: `bun run cli span-logs <span-id>`
- Query facets via CLI: `bun run cli facets <traces|logs> <field>`
- Print Effect setup instructions: `bun run instructions`
- Build the web UI: `bun run web:build`
- Dev the web UI (with hot reload): `bun run web:dev`
- Typecheck: `bun run typecheck`
- Effect LSP diagnostics over the whole project: `bunx effect-language-service diagnostics --project tsconfig.json --format text`
- Effect LSP interactive setup wizard: `bunx effect-language-service setup`

## Release Strategy
- npm package: `@kitlangton/motel`
- Current published npm `latest`: `0.2.4` (`npm view @kitlangton/motel dist-tags --json`)
- Tags are versioned as `vX.Y.Z` (`git tag --sort=-version:refname` shows `v0.2.4`, `v0.2.3`, ...)
- Publishing is handled by GitHub Actions in `.github/workflows/publish.yml`, not by local manual `npm publish`
- The publish workflow triggers on `git push` of tags matching `v*` or via manual `workflow_dispatch`
- The workflow runs `bun install --frozen-lockfile`, `bun run typecheck`, `bun run test`, then `npm publish --provenance`
- `npm publish` runs `prepublishOnly`, which builds the web UI via `bun run web:build`
- Before tagging a release, make sure the committed `package.json` version matches the intended git tag exactly
- Preferred release flow: update `package.json` version, commit the release changes, create tag `v<package.json version>`, push the commit and tag, then verify the GitHub Actions publish and npm dist-tags
- Do not create or push release tags from a dirty worktree with unrelated uncommitted changes; ask before including unrelated edits in a release

## Effect LSP
The repo is wired up with `@effect/language-service` as a `tsconfig.json` `plugins` entry. Editors that pick up the TypeScript workspace plugin (Zed, VSCode, Cursor, NVim via vtsls) will surface Effect-specific diagnostics, quick fixes, and refactors inline. In Zed this requires selecting the workspace TypeScript version — it does so automatically when `node_modules/typescript` is present.

## Verification
- The built-in verification step is `bun run typecheck`.
- For runtime verification, start the TUI or server once, then query `http://127.0.0.1:27686/api/services`, `http://127.0.0.1:27686/api/spans/<span-id>`, `http://127.0.0.1:27686/openapi.json`, and `bun run cli logs motel-otel-tui`.
- For span-centric debugging, use `http://127.0.0.1:27686/api/spans/search?...`, `http://127.0.0.1:27686/api/spans/<span-id>/logs`, and `http://127.0.0.1:27686/api/traces/<trace-id>/spans`.

## API Notes
- List and search endpoints return a `meta` object with `limit`, `lookback`, `returned`, `truncated`, and `nextCursor`.
- `/api/traces` and `/api/traces/search` return summaries by default. Use `/api/traces/<trace-id>` for the full trace tree.
- `/api/logs` and `/api/logs/search` support `severity` (e.g. `?severity=ERROR`), case-insensitive body search, and `attrContains.<key>=<substring>` for substring search inside attribute values.
- `/api/spans/search` supports `traceId` to scope to one trace, `attr.<key>=<value>` for exact match, and `attrContains.<key>=<substring>` for case-insensitive substring search inside attribute values.
- `/api/ai/calls` searches AI SDK calls (streamText, generateText, etc.) with first-class filters for `model`, `provider`, `sessionId`, `functionId`, `operation`, `status`, `text` (cross-field search), and returns compact summaries with previews and token usage.
- `/api/ai/calls/<span-id>` returns the full detail of a single AI call including complete prompt messages, response text, tool calls, timing, and correlated logs.
- `/api/ai/stats` aggregates AI call statistics by `provider`, `model`, `functionId`, `sessionId`, or `status` with aggregations: `count`, `avg_duration`, `p95_duration`, `total_input_tokens`, `total_output_tokens`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitlangton/motel](https://github.com/kitlangton/motel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
