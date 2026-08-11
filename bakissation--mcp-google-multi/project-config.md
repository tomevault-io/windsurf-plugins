---
trigger: always_on
description: Conventions for AI assistants modifying this codebase. v5 is **local, stdio, user-OAuth only**. User-facing rationale for the big mechanisms (discover-first, fan-out, trimming, write-control, escape hatch) lives in `docs/features.md`; implementation rationale too long for a code comment (Windows rename retry, GET/HEAD body stripping, admin-scope gating) lives in `docs/internals.md`. Keep both in sync when behavior changes — comments stay one-line and point there.
---

# Working on mcp-google-multi

Conventions for AI assistants modifying this codebase. v5 is **local, stdio, user-OAuth only**. User-facing rationale for the big mechanisms (discover-first, fan-out, trimming, write-control, escape hatch) lives in `docs/features.md`; implementation rationale too long for a code comment (Windows rename retry, GET/HEAD body stripping, admin-scope gating) lives in `docs/internals.md`. Keep both in sync when behavior changes — comments stay one-line and point there.

## Project shape

- `src/index.ts` — entry. `buildRegistry()` registers every enabled service (the curated `SERVICES` table plus `GENERATED_SERVICES`, gates in `src/services.ts`, all filtered by `GOOGLE_TOOLSETS`), fails fast if `registry.services()` is empty (BEFORE meta tools register — escape tools would mask an empty service set), then `registerDiscoverTools()` + `registerEscapeTools()` + `registerAccountTools()`; `main()` installs the registry's custom `tools/list` handler and runs the stdio MCP server, or the `auth` / `migrate-tokens` / `config check` CLI commands.
- `src/registry.ts` — `ToolRegistry` wraps the MCP server: records `{name, service, cud, description, inputShape, meta}` per tool, derives `cud` (read/create/update/delete) from the tool name via `inferCud`, **enforces write-control** by wrapping every CUD handler, injects computed annotations (`readOnlyHint`/`destructiveHint`), and owns **discover-first visibility**: all tools register eagerly (always callable — graceful dispatch), but the custom `tools/list` handler only advertises meta tools + `reveal()`ed services. `installListHandler()` must run after registration, before connect. Service files still call `server.registerTool(...)` — `server` is now a `ToolRegistry`.
- `src/discover.ts` — `registerDiscoverTools()`: one eager `{service}_discover` meta-tool per service; returns the catalog (`registry.catalog`), reveals the service, triggers `tools/list_changed`. Never emit `tool_reference` content blocks or top-level `defer_loading` fields — strict SDK clients reject/strip them (verified against SDK 1.29).
- `src/toolsets.ts` — `GOOGLE_TOOLSETS` parsing (`all` | CSV of service names).
- `src/discovery-client.ts` — runtime Google API Discovery: `loadMethodIndex(api)` fetches `https://www.googleapis.com/discovery/v1/apis/{id}/{version}/rest` on demand, disk-caches 7 days (`DISCOVERY_CACHE_PATH`), stale-if-offline; `cudFromMethod` (HTTP verb → cud, POST read-verbs like batchGet → read), `expandPath` ({param} + {+param} templates), `searchMethods`. Deps injectable for tests — never hit the network in unit tests.
- `src/tools/google-api.ts` — `registerEscapeTools()`: eager `google_api_search` + `google_api_call`. The call tool enforces write-control ITSELF (its registry cud is read, so the wrapper doesn't gate it) via `isAllowed` on the method's derived cud — keep that check when touching this file.
- `src/fanout.ts` — cross-account fan-out: `parseAccountSelector` (`"*"` | CSV | single), `runFanout` (bounded 5, per-account isolation, merged `{results, partial}` envelope with parsed payloads — never embed raw JSON strings), `fanoutAccountField` (union schema: enum+`*` | CSV regex). Wired in `registry.ts`: read tools only, never meta (`google_api_call` infers cud=read but executes writes), never `FANOUT_EXCLUDE` (read tools that write local files). Fan-out wraps the raw handler INNERMOST so compaction runs once on the merged envelope.
- `src/tools/accounts-tool.ts` — eager `account_list` meta tool + `deriveAccountHealth` (pure, deps-injectable). Reads the token store directly — NEVER `getClient` (would attach refresh listeners). `readToken` throws on decrypt failure (only returns null when missing) — keep the per-account try/catch. Never output token values.
- `src/trim.ts` — response trimming: `compactResult` (registry re-serializes every JSON text output compactly; `GOOGLE_TRIM=off` opts out — compaction only, never the caps), `capText` (paging caps with truncated/totalChars), `sliceClean` (permanent caps — drops a trailing lone surrogate). Fat readers own their caps: `drive_read` `maxChars`/`offset`, gmail reads 50k body cap + `full` param, `calendar_list_events` AND `calendar_list_instances` list-mode `formatEvent(e, {full:false})`. New fat-payload tools should reuse these helpers.
- `src/write-control.ts` — `resolvePolicy()` (env → policy) + `isAllowed()` (deny-by-default verdict) + `config check` rendering.
- `src/token-store.ts` — encrypted token store (AES-256-GCM, key from `MASTER_KEY`); `readToken`/`writeToken`/`updateToken`. The `migrate-tokens` CLI lives in `src/migrate-tokens.ts`.
- `src/auth.ts` — OAuth flow + scope tiers (`BASE_SCOPES` always, `OPTIONAL_SCOPE_BUNDLES` env-gated, `ADMIN_SCOPES` per-account).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bakissation/mcp-google-multi](https://github.com/bakissation/mcp-google-multi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
