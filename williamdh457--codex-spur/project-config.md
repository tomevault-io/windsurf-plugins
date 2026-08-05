---
trigger: always_on
description: Codex Spur is a macOS-first Tauri desktop application that publishes user-selected models into Codex's existing model picker without modifying or injecting code into `ChatGPT.app`.
---

# AGENTS.md — Codex Spur Engineering Contract

## 1. Product intent

Codex Spur is a macOS-first Tauri desktop application that publishes user-selected models into Codex's existing model picker without modifying or injecting code into `ChatGPT.app`.

The product integrates through three supported seams:

1. a localhost OpenAI Responses-compatible proxy;
2. a generated Codex `model_catalog_json`;
3. a dedicated Codex model provider named `codex_select`.

Closing the main window must keep the proxy alive in the menu-bar process. Quitting the app must stop the proxy and release all account leases. Version 1 must not install a LaunchAgent, privileged helper, or unrelated background daemon.

## 2. Required implementation order

When bootstrapping or rebuilding the repository:

1. create and maintain this `AGENTS.md` first;
2. preserve `DESIGN-cohere.md` as a read-only visual reference;
3. create and maintain the desktop-specific `DESIGN.md`;
4. only then scaffold or edit application source.

Do not delete, move, or reinterpret unknown user files without explicit approval. Existing root-level JavaScript research artifacts are not application entrypoints and must not be included in the production bundle.

## 3. Architecture boundaries

### Frontend

- React + TypeScript is responsible for presentation, interaction state, accessibility, and invoking typed Tauri commands.
- The frontend must never receive raw access tokens, refresh tokens, API keys, session cookies, proxy bearer tokens, or decrypted credential payloads.
- TypeScript types exposed over IPC must be generated from or checked against Rust-authoritative schemas.

### Rust application core

- Rust owns provider configuration, model discovery, credential normalization, encryption, account scheduling, quota operations, protocol adaptation, Codex configuration writes, backups, and the localhost proxy.
- Domain logic must not depend directly on Tauri window types. Put Tauri commands at the boundary so core modules remain testable without a GUI.
- Long-running network operations must be cancellable and must not block the Tauri event loop.

### Local proxy

- Bind only to `127.0.0.1` in v1.
- Require a per-install bearer token for every route except a deliberately minimal health probe.
- Accept Codex-facing OpenAI Responses requests and normalize upstream Responses, Chat Completions, Anthropic Messages, and ChatGPT Codex backend traffic.
- Client disconnects must cancel upstream work and release leases.
- Do not advertise WebSocket, image, search, audio, service-tier, or parallel-tool capabilities unless the route has been verified to support them.

### Codex integration

- Use a dedicated provider id: `codex_select`. Never overwrite an existing `custom`, Nice Switch, CC Switch, or unrelated provider table.
- Preserve comments and unrelated TOML sections with `toml_edit`-style structural edits.
- Generate stable opaque route slugs; do not expose account ids, emails, provider secrets, or credential fingerprints in model slugs.
- Normal operation must not modify Codex's native `auth.json`. Native-account synchronization is a separate, explicit, backed-up action.

### Primary runtime: Codex App (GUI), not CLI-only

**The product is consumed primarily through the Codex APP / GUI** (`ChatGPT.app` embedded Codex UI, and/or Orca-hosted Codex). Agents and engineers must not assume a pure `codex` CLI workflow.

Facts for this machine/product path:

- Live provider: `model_provider = "codex_select"` with localhost proxy (typically `http://127.0.0.1:17861/v1`).
- Catalog: `model_catalog_json` → Spur opaque `spur-route-*` slugs (or instance/model path slugs).
- Session truth lives in Codex App rollouts: `~/.codex/sessions/**/rollout-*.jsonl` and `state_5.sqlite` threads — not only terminal logs.
- When diagnosing user reports of “model switch broke”, read `turn_context.model`, `thread_settings_applied`, and `task_complete.error` from the rollout. UI labels like “ChatGPT 4.5.6” map to catalog models such as `gpt-5.6-sol`; “Kimi K3” maps to the Kimi K3 spur-route slug.

### Cross-provider same-thread hazard (P0)

**Do not treat mid-thread model switching across providers as safe.** Codex App reuses the full conversation `input[]` when the user switches models in the same thread. Different upstreams emit incompatible history:

| Failure mode | Typical error | Cause |
|---|---|---|
| OpenAI after Kimi | `Invalid 'input[N].id': 'resp_…_msg'. Expected an ID that begins with 'msg'.` | Kimi/Spur assistant items use `resp_*_msg` IDs; OpenAI requires `msg*` |
| Grok/other after prior model | `Could not decrypt the provided encrypted_content` | Reasoning/encrypted blocks are not portable across providers |
| Mid-run “已处理” then silence | `stream disconnected before completion` / `401` to `:17861` | Local proxy or upstream stream end; App marks the turn processed even on failure |

Product implications (must fix in proxy/product, not by catalog-only edits):

1. On **cross-provider** resume, sanitize outbound history: rewrite/drop illegal message ids, strip foreign `encrypted_content`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [williamdh457/codex-spur](https://github.com/williamdh457/codex-spur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
