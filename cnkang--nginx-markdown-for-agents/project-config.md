---
trigger: always_on
description: This file defines repository-specific engineering rules for AI agents working on `nginx-markdown-for-agents`.
---

# AGENTS.md

## Purpose
This file defines repository-specific engineering rules for AI agents working on `nginx-markdown-for-agents`.

These rules are distilled from:
- NGINX official development constraints in the Kiro local development guide (when present)
- Historical fix/doc and hidden-fix commits across local branches and remote-only commits (deduplicated by commit SHA)

Goal: prevent repeated classes of mistakes that previously caused regressions.

## Rule Priority
1. Kiro local development guide, when present (highest priority)
2. This `AGENTS.md`
3. Task-specific user instructions

If two rules conflict, follow the higher-priority source.

## Harness Map
- `AGENTS.md` remains the Codex-first contract and engineering rule map.
- `AGENTS.md` and `docs/harness/` are the owning harness truth surfaces for
  tracked repository behavior; Make/CI/checkers must consume these, not local
  adapter-only summaries.
- `docs/harness/README.md` is the repo-owned harness entrypoint.
- `docs/harness/core.md` defines the execution loop, conflict protocol, and
  status semantics.
- `docs/harness/routing-manifest.json` is the canonical structured routing
  source; `docs/harness/routing-manifest.md` is the readable overlay.
- Optional local spec inputs are read-only for local spec-oriented work. Do not
  write harness caches, annotations, or durable repo truth into local adapter
  surfaces.
- Optional local steering adapters should point back to `docs/harness/` and
  must not define stronger semantics than this file.
- Outside voice and the user-local harness state carrier are advisory execution
  tools. They may challenge or inform the current path, but they do not weaken
  the repo-owned correctness and safety contract.

## Non-Negotiable NGINX Baseline

### API and lifecycle correctness
- Use NGINX return-code semantics correctly: `NGX_OK`, `NGX_DECLINED`, `NGX_AGAIN`, `NGX_DONE`, `NGX_ERROR`.
- In HTTP phases/filters, treat `NGX_AGAIN` as suspend-and-resume, not success.
- Finalize requests with the correct code path (`ngx_http_finalize_request`) when redirect/subrequest/finalization semantics require it.
- Never send body data before headers; header forwarding state must be explicit and idempotent.

### Memory and concurrency model
- Prefer NGINX pool allocation for request-lifetime data.
- Avoid unbounded allocations in request path; every growing buffer must have an explicit cap.
- Avoid global mutable state; bind long-lived state to config/cycle structures.
- Avoid blocking behavior and blocking libraries in worker request path.

### C style and conventions
- Follow NGINX style: 4-space indent, <=80 cols where practical, no `//` comments, `_t` suffix for types.
- Keep NGINX style as the primary style contract, but require C99-or-new as
  the minimum language baseline for all C code and C snippets in docs/steering.
  Pre-C99 forms are forbidden (for example K&R function definitions, implicit
  `int`, and declarations without proper prototypes).
- Use `u_char *`, `ngx_str_t`, and NGINX helpers (`ngx_snprintf`, `ngx_memcpy`, etc.) consistently.
- Use `NULL` pointer comparisons (not `0`).
- For POSIX string helpers (for example `strcasecmp`, `strncasecmp`), include
  `<strings.h>` explicitly in the translation unit or shared header. Do not
  rely on transitive includes or implicit declarations.
- **Never dereference or perform relational operations on values that may be uninitialized, NULL, or invalid without an explicit guard.** This includes: pointer comparisons (`p > q`, `p < q`), pointer arithmetic, field access through pointers, array indexing with unvalidated bounds. When the validity of a value depends on runtime state (for example `pos/last` may both be NULL in empty buffers), use an explicit boolean flag set at the production site rather than inferring state from value relationships.

## Frequent Error Patterns and Required Prevention

### 1. Streaming backpressure and pending-chain handling
Historical issues: `5649890`, `23165d9`, `cfd4bd8`, `f97be3f`.

Required:
- If downstream filter returns `NGX_AGAIN`, persist the unsent chain in context and resume it later.
- Never overwrite pending output with terminal empty `last_buf` while data is still pending.
- On fallback from streaming to full-buffer path, reset state flags that gate conversion flow (for example conversion-attempt flags).
- Fail-open paths must still honor header/body ordering and deferred-header forwarding.

### 2. Incorrect fail-open semantics causing data loss or pointer advancement bugs
Historical issues: `23165d9`, `7cbe4fa`.

Required:
- Use semantically correct return codes in fail-open branches (`NGX_DECLINED` where needed), so caller behavior matches control intent.
- Do not advance buffer positions for unconsumed data when handing chain to next filter unchanged.
- If a fail-open branch invokes next header filter, mark header-forwarded state before the call to avoid double header emission.

### 3. Memory leaks and budget bypass in streaming/decompression
Historical issues: `23165d9`, `2c7d6a9`, `0eae34b`, `1b0df51`.

Required:
- Enforce all configured budgets (including total working-set budget), not only per-buffer budgets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cnkang/nginx-markdown-for-agents](https://github.com/cnkang/nginx-markdown-for-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
