---
trigger: always_on
description: GenerateStream, Cost(), WS.
---

# CLAUDE.md

Guidance for Claude Code working in this repo.

## Dev workflow

Before any non-trivial change, read `CONTRIBUTING.md` (build/test, the
PR workflow, and the load-bearing codebase rules) and the design docs
under `.tmp/design/` — internal notes, gitignored, kept out of the
published tree (`.tmp/design/roadmap.md` for what's next,
`.tmp/design/canonical-protocol.md` for the protocol invariants). The repo layout, ports, and build/deploy
pipeline are documented in `CONTRIBUTING.md` and the `Makefile`.

## What this is

Wyolet Relay — a high-throughput LLM router in Go. Self-hostable,
k8s-native, BYO provider keys. The wedge is the infrastructure axis —
performance, key pooling, batch orchestration, observability.

Open source (Apache-2.0), published as a Docker image (`wyolet/relay`)
and deployable to k8s. The public-facing surface (README, quickstart,
issue templates, docs at docs.wyolet.com) assumes no internal context;
keep that audience in mind when editing anything outside `.tmp/design/`.

Roadmap and stage breakdown: `.tmp/design/roadmap.md` (the index; the OSS
core backlog lives in `.tmp/design/roadmap-oss.md`). Design context lives in
the `.tmp/design/` tree (`.tmp/design/canonical-protocol.md`, `.tmp/design/adapters/`,
the per-subsystem guides) — read before non-trivial architectural
suggestions.

## Wedge — what we are NOT

- Not a marketplace / reseller of provider tokens (deferred indefinitely)
- Not Python — performance is a wedge, runtime cost matters
- Not a feature-parity clone of an existing router — infra-grade
  throughput is the goal, not matching every checkbox
- Not adaptive / quality-aware routing in v1 — bad auto-routing loses
  customers
- Not a custom wire protocol — OpenAI/Anthropic shapes are accepted as
  passthrough

## Repository layout

Three Go modules wired by a root `go.work` (`use .`, `use ./sdk`,
`use ./jobq`). The codebase is organised by responsibility.

```
app/                       — the application: domain + composition + handlers
  catalog/                 — composition layer: immutable Snapshot, COW
                             reconciler, NOTIFY listener, Bootstrap();
                             overlay merge applied here (overlay_apply.go)
  catalogview/             — PG-backed consumer read projections for admin/UX
                             endpoints (NOT the hot-path snapshot)
  catalogembed/            — composes manifest YAML → SDK catalog embed schema
  catalogvalidate/         — cross-entity graph linter for the catalog
  {provider,host,model,
   hostkey,ratelimit,
   policy,pricing,binding,
   relaykey}/              — 9 entity packages. Each: domain types, Validate(),
                             Store{List,Get,Upsert,Delete}. `binding` is the
                             first-class HostBinding join (Model×Host×adapter).
  overlay/                 — catalog overlays: user-owned sparse spec patches
                             on pristine TEMPLATE rows, merged to EFFECTIVE
                             rows at snapshot load (survives re-seed). See
                             .tmp/design/overlays.md.
  modelref/                — parses the catalog model-reference DSL + aliases
  meta/                    — identity primitives every entity carries (id/
                             name/displayName, owner)
  settings/                — typed-sectioned config layer (DB-backed settings,
                             incl. governance:* sections). See .tmp/design/settings.md.
  settingswatch/           — applies a value-typed settings section to a live
                             component on change
  adapter/                 — generic adapter framework (singular):
                             Spec, Registry, generic pipeline.Adapter
                             parameterised by upstream URL + auth strategy.
                             ONE Spec literal per wire shape lives in
                             cmd/relay/main.go.
  adapters/                — vocabulary only: Name constants
  pipeline/                — pure orchestration: reserve → pick key →
                             Adapter.Call → stream → post-flight emit
  proxy/                   — second inference flow: transparent forwarding
  routing/                 — snapshot lookup → Plan{Model, Policy, Binding,
                             Host, Keys, Rules}; resolves aliases
  keypool/                 — key selection + per-key circuit breaker
                             (state at `secret_health:*` in kv)
  hosthealth/              — per-host runtime reachability (kv-backed)
  ratelimit/               — RateLimit entity + Resolve(policy, rl) → Rules
  secret/                  — secret Store + KeyAgent (out-of-band heal/failover)
  httpapi/                 — the HTTP layer
    inference/             — data plane: /v1/* + /healthz; shape-agnostic
                             Dispatch with NO per-vendor branching
    control/               — admin plane: /auth/* + CRUD + /version + ...
  transport/ws/            — customer-facing WebSocket inference transport
  batch/                   — batch consumer: reuses Pipeline.Run (source="batch")
                             over the jobq module. See .tmp/design/ + roadmap.
  manifest/                — YAML DTOs + translate ↔ domain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyolet/relay](https://github.com/wyolet/relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
