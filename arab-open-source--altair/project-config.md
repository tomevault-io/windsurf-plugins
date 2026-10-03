---
trigger: always_on
description: This file gives human and AI contributors the context they need to work on
---

# AGENTS.md — Working with Altair

This file gives human and AI contributors the context they need to work on
Altair effectively: where the project stands, what to build next, how to
write code that fits, and how to verify it. Read it before touching the
codebase, and treat it as a contract for every change you make.

---

## What Altair is

Altair is a **batteries-included web framework for Crystal**. Batteries
included means it ships the full stack — routing, controllers, views,
configuration, an ORM, generators, CLI — with sane defaults, so that
building a web app with Crystal feels like the framework is working *for*
you, not against you.

The driving philosophy, from the roadmap:

> Every phase ends with something working and visible — vertical slices,
> not horizontal scaffolding. Each phase has a clear exit criterion.

Nothing ships unless someone can look at it, click it, and see it work.

## Non-negotiables

- **Do not mention "Rails"** — not in code, comments, docs, or commit
  messages. It is an old house rule; keep it that way. Performance and API
  comparisons to other frameworks are fine in conversation, just never
  "Rails" in the repo.
- **Specs from day one.** Every phase ends with its specs passing.
- **Never skip a phase's exit criterion.** Building the ORM before a
  working demo exists is wasted effort.
- **No emojis in code or docs.**
- **No code comments that state the obvious.** Use short doc comments above
  public classes and methods (Crystal doc style), matching the existing
  code.

## Current status

- **Phases 0–5 done** (Foundation, Router, Controllers, Views, Record/ORM,
  CLI + Generators). `examples/blog` is the always-running persistence demo
  — its posts and comments survive restarts.
- 992 specs (15 pending: 7 Redis rate-limit + 1 Redis middleware + 6
  PostgreSQL concurrency contract gated on `ALTAIR_TEST_PG_CONCURRENCY`, plus
  the fuller PostgreSQL contract suite on `ALTAIR_TEST_PG_URL`), formatter
  clean, Ameba silent on framework sources.
- Phase 7 shipped four features: **testing utilities** (`Altair::Test.boot`
  with a `configure:` hook, the cookie-jar `Altair::Test::Client` with
  browser-like redirects, `migrate!`, `transactional`),
  **full authentication** (`altair g auth` generating model + migration +
  sessions/registrations controllers + views + routes; the `password_auth`
  macro hashing through `Altair::Auth::PasswordHasher` — PBKDF2-SHA256,
  no new dependencies), the **asset pipeline** (`assets/` fingerprinted
  into `public/assets/` with a manifest by `altair assets:precompile`;
  `stylesheet_link_tag` / `javascript_asset_tag` / `asset_url`; immutable
  caching on fingerprints only), and **background jobs** (typed `params`
  with compile-checked `enqueue`/`enqueue_in`/`enqueue_at`, a lazily-created
  `altair_jobs` table, atomic claiming so concurrent workers never
  double-run, exponential-backoff retries inside a per-job budget,
  `altair jobs:work` / `jobs:stats` commands, and an in-memory test mode).
  `examples/blog` demonstrates all of it end to end. Phase 7 fully complete: joins (table-qualified where,
  DISTINCT, COUNT(DISTINCT)), has_many :through (inferred source),
  polymorphic associations, cache layer (Memory + Redis), storage
  (Disk + S3 with SigV4), WebSocket Cable (auth hook + heartbeat +
  JSON envelopes), admin generator, API mode, structured logs,
  observability (/health + /metrics), dynamic redirect interpolation,
  Altair::Redis pure-Crystal client, and Wave 0 ORM hardening
  (IN chunking, custom PK + UUID, atomic writes, order/reload).
- Phase 6 hardening shipped in waves: sessions + flash + CSRF + auth
  (`require_login` / `authenticate!` + `Altair::Auth::JWT`), then a
  configuration wave — `.env` (real env vars win; `.env.<environment>`
  overrides `.env`) and `config/database.yml` per-environment settings are
  merged into `config` at boot by `Altair::Config::DotEnv` /
  `Altair::Config::Database`, and `altair new` generates both files so a
  fresh project's configuration is file-driven, not code-driven — and a
  multipart wave: `multipart/form-data` bodies parse into the parameter
  bag (scalar fields as params, files as `Altair::HTTP::UploadedFile` via
  `params.upload("avatar")`, with `UploadedFile#save` + `#content`). The
  default middleware stack shipped a security wave:
  `SecurityHeaders` (default `nosniff` / `SAMEORIGIN` / referrer policy,
  driven by `config.security_headers`), `RequestId` (`request.request_id`,
  echo-back through `config.request_id_header`, appended to the request log
  line), `RateLimit` (sliding-window, `config.rate_limit` — `MemoryStore` /
  `RedisStore` with `Retry-After` + `X-RateLimit-*`; `examples/rate_limit_demo`
  shows it live), and opt-in `Cors` (`config.cors.origins` enables it; preflight
  answered directly).
- The router shipped three post-Phase-1 waves: `resources` blocks
  (`member`/`collection`/nested, Phase 1 era), constraints + implicit
  format suffix, and `redirect` / glob segments / singular `resource`
  (Wave 3, current).
- The controller layer shipped a post-Phase-2 hardening wave: JSON-object
  rendering, `redirect_back`, `request.format`, JSON request bodies, clean
  `head`, `respond_to`, callbacks (`before_action` / `after_action` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arab-Open-Source/Altair](https://github.com/Arab-Open-Source/Altair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
