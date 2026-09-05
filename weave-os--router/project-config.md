---
trigger: always_on
description: > **Mirror notice.** Generated from [CLAUDE.md](CLAUDE.md). Edit CLAUDE.md, then run `make generate-agent-guides`; CI rejects drift.
---

# router — AGENTS

> **Mirror notice.** Generated from [CLAUDE.md](CLAUDE.md). Edit CLAUDE.md, then run `make generate-agent-guides`; CI rejects drift.

Root guide for AI agents in the `router/` subproject. Covers cross-cutting design + the layer model. **First read for any task:** [README](README.md), then this file. Then read the `CLAUDE.md` inside the package you're editing — each subpackage has its own with focused recipes + invariants.

## Engineering principles

- **Patterns of Enterprise Application Architecture** (Fowler)
- **Designing Data-Intensive Applications** (Kleppmann)
- **Design Patterns** (GoF)
- **CLEAN architecture** (Martin) — especially dependency inversion
- **DRY**
- **Small expert team** — explicit composition, readable wiring; reject DI containers, reflection, framework magic
- **Concise comments, sparingly** — default to none. Only when *why* is non-obvious (hidden constraint, subtle invariant, workaround, surprising behavior). Never rehash code, never reference current task/PR/caller, no multi-paragraph. If removing wouldn't confuse, don't write.
- **Non-tautological tests** — every test must assert behavior that breaks if prod code deleted.

## Layer model and import rules

Three concentric layers. Imports flow inward only.

```
+-------------------------------------------------------------------+
|  cmd/router/main.go             (composition root — wires all)    |
|                                                                   |
|  +-------------------------------------------------------------+  |
|  |  internal/api/admin       (presentation: /health, /validate,|  |
|  |                            /admin/v1/*)                     |  |
|  |  internal/api/anthropic   (/v1/messages, passthrough,       |  |
|  |                            /v1/route)                       |  |
|  |  internal/api/openai      (/v1/chat/completions)            |  |
|  |  internal/api/gemini      (/v1beta/models/:modelAction)     |  |
|  |  internal/api/feedback    (/f/<token> no-login feedback     |  |
|  |                            page; deliberately no auth       |  |
|  |                            middleware)                      |  |
|  |  internal/api/analytics   (/v1/analytics/* read-only        |  |
|  |                            routing-decision export;         |  |
|  |                            ra_ keys only)                   |  |
|  |  internal/server          (route registration)              |  |
|  |  internal/server/middleware (auth, timeout, cluster/embed   |  |
|  |                              overrides, OTel timing)        |  |
|  |  internal/postgres        (adapter: SQLC over pgx; also     |  |
|  |                            session-pin store + billing repo |  |
|  |                            impls)                            |  |
|  |  internal/sqlc            (generated; regenerate via        |  |
|  |                            `make generate`)                 |  |
|  |  internal/router/cluster  (Router impl: AvengersPro,        |  |
|  |                            Multiversion)                    |  |
|  |  internal/providers/*     (Client impls: anthropic, openai, |  |
|  |                            google native, openaicompat,     |  |
|  |                            httputil)                        |  |
|  |  internal/observability/otel (span emitter; adapter)        |  |
|  |  internal/observability/apm  (second OTel/SigNoz adapter:   |  |
|  |                            gin HTTP spans + Go runtime      |  |
|  |                            metrics; independent of otel's   |  |
|  |                            per-decision span emitter)       |  |
|  |  internal/pubsub          (adapter: GCP Pub/Sub cache-      |  |
|  |                            invalidation + recharge-needed   |  |
|  |                            notify)                           |  |
|  |                                                             |  |
|  |  +-------------------------------------------------------+  |  |
|  |  |  internal/auth      (identity domain: types,          |  |  |
|  |  |                      repos, Service.VerifyAPIKey,     |  |  |
|  |  |                      APIKeyCache, id/hashing, Tink    |  |  |
|  |  |                      encryptor)                       |  |  |
|  |  |  internal/billing   (balance checks + inference       |  |  |
|  |  |                      debits: Service + Repo iface;    |  |  |
|  |  |                      Postgres impl in                 |  |  |
|  |  |                      internal/postgres)                |  |  |
|  |  |  internal/feedback  (pure HMAC signer for the         |  |  |
|  |  |                      no-login feedback-link token;    |  |  |
|  |  |                      no I/O)                            |  |  |
|  |  |  internal/analytics (routing-decision export domain:  |  |  |
|  |  |                      Decision row shape, keyset       |  |  |
|  |  |                      cursor, schema + price book;     |  |  |
|  |  |                      Service + Repository iface,      |  |  |
|  |  |                      Postgres impl in                 |  |  |
|  |  |                      internal/postgres)                |  |  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weave-os/router](https://github.com/weave-os/router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
