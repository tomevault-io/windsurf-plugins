---
trigger: always_on
description: **Karna** (`karna`) is a Lua plugin for Kong Gateway that acts as a
---

# Karna — CLAUDE.md

## Project Overview

**Karna** (`karna`) is a Lua plugin for Kong Gateway that acts as a
**WAF (Web Application Firewall) engine** compatible with the
**OWASP CoreRuleSet (CRS)**. It runs as a fully self-contained Kong
plugin with no required dependency on other plugins.

## Architecture

### Entry Point
- `kong/plugins/karna/handler.lua` — Kong plugin handler (priority 8300).
  Implements `init_worker`, `access`, `header_filter`, `body_filter`, and `log` phases.
  - On `init_worker`: loads and caches CRS rules (parsed from SecLang `.conf` files) into an LRU cache.
  - On `access`: evaluates rules against the incoming request (rule controls first, then local rules, then global CRS rules).
  - On `log`: writes JSON audit logs to disk asynchronously.
  - **Self-identification** (always on, no toggle): a request to the reserved path `GET /.well-known/karna` short-circuits in `access` and returns `{engine, version, commit, commit_short, built_at}` (build identity from `version.lua`). The same `version`/`commit` are also embedded in the `engine` block of audit log v2 (`ka_utils.lua:get_auditlog_v2`). This is a transparent license-compliance watermark — passive (no phone-home), reserved path does not reach the upstream.

### Schema
- `kong/plugins/karna/schema.lua` — Plugin configuration schema. Key settings:
  - `engine_blocking_mode` (bool): when true, matched rules return 403; otherwise detection-only.
  - `paranoia_level` (number, 1-4): OWASP CRS paranoia level.
  - `coreruleset_enabled` (default true): toggle for the OWASP CRS rule pack loaded from disk at `init_worker`. The in-repo CRS-fix rule controls (`coreruleset_fix.lua`) are always applied independently.
  - `local_rules_enabled`: per-service custom rules.
  - `rules_request`: per-service JSON rule array (all phases; each rule runs in the phase named by its `phase` field — `access` or `header_filter`).
  - `auditlog_enabled`, `auditlog_path`, `auditlog_modsec`: audit logging config.
  - `redis_host`, `redis_port`, `redis_password`: Redis connection for counters.
  - Various request validation limits (arg length, arg count, methods, content types, extensions, charsets).

### Core Modules (in `kong/plugins/karna/modules/`)
- **`ka_engine.lua`** — The rule evaluation engine. Loads CRS rules via `seclang`, resolves variables from the request context, applies transformation functions, runs operators (regex, libinjection, string match, etc.), and evaluates rule chains. This is the largest and most critical module.
- **`seclang.lua`** — SecLang (ModSecurity rule language) parser. Reads OWASP CRS `.conf` files from the path in `seclang.crs_path` (default `/opt/coreruleset/rules/`, override via `KARNA_CRS_PATH` env var). Matches canonical `SecRule <vars> "<op>" "<actions>"` only — `SecRule*` derivatives like `SecRuleUpdateTargetById` (CRS 4.x exception files) are intentionally skipped, not parsed. A defensive guard skips any malformed `SecRule` with a `WARN` print so a single bad rule cannot crash `init_worker`.
- **`ka_body_parser.lua`** — Request body parser. Handles URL-encoded, JSON, multipart, and XML body formats. Flattens nested structures into key-value pairs for rule evaluation. Supports optional base64 decoding. Gzip-encoded bodies require `lua-zlib` (declared in rockspec).
- **`ka_mcp.lua`** — MCP (Model Context Protocol) request-side detection and JSON-RPC envelope parsing. Populates the `mcp.*` variable namespace used by rules and exposes operators `mcp_method_in` and `mcp_jsonrpc_valid`. Brand-neutral (`mcp.*` names are protocol-level, not Karna-branded).
- **`ka_mcp_sse.lua`** — Response-side SSE reassembler for the MCP Streamable HTTP transport. Reconstructs `event:` / `data:` frames, evaluates rules per event in the `mcp_event` phase, and supports streaming actions (drop / replace / terminate / inject).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sicuranext/karna](https://github.com/sicuranext/karna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
