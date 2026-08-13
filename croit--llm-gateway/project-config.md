---
trigger: always_on
description: This file is the canonical entry point for any AI agent (or new human contributor) working in this repo. Read it end-to-end before doing anything else.
---

# AGENTS.md — read this first

This file is the canonical entry point for any AI agent (or new human contributor) working in this repo. Read it end-to-end before doing anything else.

## What this project is

A single Rust binary plus the supporting crates it lives on:

- **`gateway`** — authenticated, OpenAI-compatible LLM proxy. Speaks `/v1/chat/completions`, `/v1/audio/transcriptions`, `/v1/models` so any OpenAI SDK talks to it. OIDC browser login + gateway-minted bearer tokens. Routes across **multiple upstream LLM backends** with health checks + RAII in-flight accounting. Injects **company-specific tools** gated by **RBAC**. Server-rendered HTML UI (dashboard / tokens / persisted multi-conversation chat).

Shared crates:
- **`session-core`** — chat-style UI substrate (Plait renderers + SSE primitives + DB schema + worker registry + `SessionDriver` trait). The gateway plugs in an `OpenAiDriver`; the trait keeps the renderers driver-agnostic so a future second consumer can paint the same chat surface without forking.
- **`shared`** — OpenAI wire types shared across the workspace.

Built on **rama 0.3** (HTTP server + router + middleware), **plait** (inline-in-handler HTML), **datastar** (client-side reactivity over `datastar-patch-elements` SSE events), and **daisyUI v5 + Tailwind v4** (styling tokens).

## Repo layout

```
/
├── AGENTS.md                    # this file
├── README.md                    # human-facing — keep current with deploy story
├── mise.toml                    # toolchain pin + build/test/lint tasks
├── Cargo.toml                   # workspace manifest (9 members)
├── Dockerfile                   # gateway runtime image
├── docs/                        # detailed design docs (index in docs/README.md)
├── ui/                          # Tailwind v4 + daisyUI v5 → app.css; TS bundle for gateway
├── gateway.example.toml         # template config — copy to gateway.toml
└── crates/
    ├── shared/                  # OpenAI wire types, shared with the CLI
    ├── session-core/            # chat-style UI substrate
    │   ├── src/                     SessionDriver trait, worker registry, db (chat_*
    │   │                            tables), Plait renderers (markdown + lumis-highlighted
    │   │                            code), SSE primitives, icons
    │   └── ui/ts/                   composer + scroll TS
    ├── gateway-core/            # base: db, config, crypto, rbac, upstreams
    ├── gateway-features/        # optional subsystems: rag, skills, comfyui, push, …
    ├── gateway-runtime/         # tool API + AppState/RamaState + chat driver
    ├── gateway-tools/           # the tool implementations
    ├── gateway-web/             # the server-rendered HTML pages
    ├── gateway/                 # the binary: router, proxy, api, main
    └── sandbox-runner/          # the sandboxed-tool execution service
```

### The gateway crate stack

The gateway is one binary assembled from three layered crates. This is **load
bearing for build speed**, not cosmetic: it used to be ~108k lines in one
compilation unit, so editing any file re-ran the whole frontend + codegen. Each
crate depends only on the ones beneath it.

```
gateway            bin + router/proxy/api/oidc      6.5k  ← thinnest, most-edited
   ├── gateway-web     server-rendered HTML pages  25.5k  ← siblings: neither
   └── gateway-tools   the tool implementations    14.5k  ←   depends on the other
          └── gateway-runtime  tool API + AppState/RamaState + chat driver  14.7k
                 ├── gateway-features  RAG, skills, ComfyUI, push, geoip, …  13.9k
                 └── gateway-core      db, config, crypto, rbac, upstreams   22.1k
```

Lines that must recompile after a one-line edit: `gateway` 6.5k, `gateway-tools`
21k, `gateway-web` 32k, `gateway-runtime` 61k, `gateway-features` 75k,
`gateway-core` 97k — against **97k for any edit** before the split. The gains are
front-loaded on purpose: the layers that churn most are the cheapest to rebuild.

`gateway-web` and `gateway-tools` are siblings: neither depends on the other, so
editing a page doesn't rebuild the tools and vice versa.

Two rules keep it that way, and both are easy to break by accident:
1. **Put new code as high in the stack as it will go.** Something belongs in
   `gateway-core` only if code below the feature layer genuinely needs it.
2. **Never reference upward.** `gateway-features` must not name `AppState` or the
   tool registry; `gateway-core` must not name a feature. One such reference
   collapses a layer.

**When adding code, put it as high in the stack as it will go.** Something only
belongs in `gateway-core` if code below the page layer actually needs it. Adding a
reference from `gateway-core` to a page — or pushing a module downward for
convenience — makes every build slow again. See [`docs/architecture.md`](docs/architecture.md#crate-boundaries).

Inside `crates/gateway-core/src/` (base layer):

```
migrations/               # (crate root) sqlx migration set, embedded by db/mod.rs
server/
    auth/oidc.rs              hand-rolled OIDC client (reqwest)
    auth/token.rs             gateway-token mint/hash helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [croit/llm-gateway](https://github.com/croit/llm-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
