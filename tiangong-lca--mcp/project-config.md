---
trigger: always_on
description: `tiangong-lca-mcp` owns TianGong LCA MCP transports and tool exposure: STDIO, authenticated Streamable HTTP, local Streamable HTTP, OAuth helpers, tool registration, and lifecycle-model preprocessing that is intentionally part of the MCP surface. Start here when the task may change how MCP clients connect or what this MCP server exposes.
---


## Repo Contract

`tiangong-lca-mcp` owns TianGong LCA MCP transports and tool exposure: STDIO, authenticated Streamable HTTP, local Streamable HTTP, OAuth helpers, tool registration, and lifecycle-model preprocessing that is intentionally part of the MCP surface. Start here when the task may change how MCP clients connect or what this MCP server exposes.

## Bootstrap Order

Load docs in this order:

1. `AGENTS.md`
2. `.docpact/config.yaml`
3. `scripts/docpact route --root . --intent <intent>` when you need path-specific routing
4. `docs/agents/repo-validation.md` when proof, runtime caveats, or CI behavior matters
5. `docs/agents/repo-architecture.md` when transport, auth, OAuth, or tool ownership is unclear
6. `README.md`, `README_CN.md`, `DEV_EN.md`, or `DEV_CN.md` only when user setup or maintainer runtime details are needed

Do not start by assuming that remote search behavior or product UI truth lives in this repository.

Preferred docpact commands:

- `scripts/docpact route --root . --intent transport-auth`
- `scripts/docpact route --root . --intent mcp-tools`
- `scripts/docpact route --root . --intent remote-search-wrappers`
- `scripts/docpact route --root . --intent openlca-tidas`
- `scripts/docpact route --root . --intent repo-docs`

## Repo Ownership

This repo owns:

- MCP transports under `src/index.ts`, `src/index_server.ts`, and `src/index_server_local.ts`
- Supabase JWT verification, protected-resource metadata, and auth configuration under `src/_shared/**` and `src/http_app.ts`
- MCP tool registration and tool wrappers under `src/tools/**`
- MCP prompts and resources under `src/prompts/**` and `src/resources/**`
- the fixed-client Supabase OAuth 2.1 protected-resource discovery and bearer-verification boundary
- the checked-in client example in `mcp_config.json`

This repo does not own:

- remote search implementation or Edge Function business logic
- Next.js product UI behavior
- standalone TIDAS conversion, export, or batch tooling
- workspace integration state after merge

Route those tasks to:

- `tiangong-lca-edge-functions` for remote hybrid-search or API runtime behavior
- `tiangong-lca-next` for product UI behavior
- `tidas-tools` for standalone conversion, validation, and export tooling
- `lca-workspace` for root integration after merge

## Runtime Facts

- Repo-local documentation governance is encoded in `.docpact/config.yaml` and enforced locally by the pre-push docpact gate; `.github/workflows/ai-doc-lint.yml` is manual-dispatch fallback.
- The only supported package-management path is pnpm `11.24.0`, selected by `packageManager`; installs use the root `pnpm-lock.yaml` with `--frozen-lockfile`.
- Runtime and compiler baselines are Node `24.19.0`, TypeScript `7.0.2`, and `@tiangong-lca/tidas-sdk` `0.2.0`; TypeScript 5/6 and compiler-API formatting plugins are not allowed in the direct or recursive graph.
- `pnpm lint` is read-only: type-aware Oxlint, Prettier check, and TypeScript typecheck run without rewriting source. Use `pnpm format` for explicit formatting writes.
- `pnpm test` runs real Node assertions for tool registration, every declared TIDAS dataset validation boundary, CRUD/search guards, LifecycleModel `validationIssues`, authenticated/local Streamable HTTP envelopes, cancellation, Supabase JWT claims, protected-resource discovery, client admission, Origin rejection, and removed-state absence.
- SDK validation uses `validateEnhanced()` exactly once per entity and consumes normalized `validationIssues`. Insert/update prepares and validates once before transport. Reads use actor-bound PostgREST; ordinary create/save/delete uses `app_dataset_create`, `app_dataset_save_draft`, and `app_dataset_delete` (`DB-CORE-WRITE-01`), while LifecycleModels use `save_lifecycle_model_bundle` and `delete_lifecycle_model_bundle` (`EDGE-BUNDLE-01`). No MCP write uses raw table DML. Strict-invalid LifecycleModels therefore perform zero auth/REST/process-lookup requests and can never reach a write; their stable error envelope exposes normalized code/path/severity without raw Zod parsing.
- `pnpm prepush:gate` is the canonical gate. It also proves the packed runtime consumer, a clean arbitrary-path worktree, build, audit, and pack contract.
- Nested package tests resolve and verify the exact native `pnpm`/`pnpm.exe` executable first with `shell: false`; Corepack JavaScript is only a fallback. Do not assume `COREPACK_ROOT` exists in `pnpm/setup` runners.
- `.github/workflows/quality-gate.yml` runs the canonical gate on Linux x64, Windows x64, macOS arm64, and Linux arm64.
- Published binaries:
  - `tiangong-lca-mcp-stdio`
  - `tiangong-lca-mcp-http`
  - `tiangong-lca-mcp-http-local`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tiangong-lca/mcp](https://github.com/tiangong-lca/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
