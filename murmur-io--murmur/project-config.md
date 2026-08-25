---
trigger: always_on
description: Guidance for Codex when working in this repository. These instructions are **binding** and override default behavior.
---

# AGENTS.md — Murmur

Guidance for Codex when working in this repository. These instructions are **binding** and override default behavior.

## Binding Rules

`AGENTS.md` is the Codex autoloaded project instruction file. The detailed rules live under
`.codex/rules/`; treat them as binding. Before editing a matching surface, read the relevant
rule file:

- `.codex/rules/rust-tauri.md`
- `.codex/rules/angular-zoneless.md`
- `.codex/rules/lock-model.md`
- `.codex/rules/agentic-workflow.md`

*(Orientation: `rust-tauri` = errors/commands/SQLCipher/additive-migrations/verify-before-destroy/gate-every-read/crash-safe-FFI/`cargo test --lib` only; `angular-zoneless` = signals-first/standalone/`@if`-`@for`/IPC→signals/dir-per-component (ts+html+scss)/Liquid-Glass views/design-tokens-only/mur-* design-system/the traps; `lock-model` = gate every read + verify-before-destroy every seal + the `convertFileSrc` leak trap; `agentic-workflow` = executable harness + adversarial-verify discipline.)*

## What Murmur is

A **local-first macOS desktop app** that records meetings, transcribes on-device, turns the transcript into a clean note via a pluggable LLM provider, and lives inside the user's **Obsidian vault**. Treat the manifests and GitHub releases as the version source of truth.

- **Stack:** Tauri 2.11 (Rust crate `murmur`, lib `meetnotes_lib`, bin `Murmur`) + Angular 22 **zoneless** (standalone, signals). IPC = Tauri commands (registered in `src-tauri/src/lib.rs` `generate_handler!`) + events. The FE talks to the backend through `src/app/core/ipc.service.ts` — **there is no NgRx**.
- **Pipeline:** capture (mic via `cpal` + system audio via a Swift **ScreenCaptureKit** sidecar) → **dual-stream** (transcribed separately, merged by wall-clock → `Me`/`Others`) → **whisper.cpp** (`whisper-rs`, Metal; selectable local model) → segments → **SQLite (canonical source of truth, SQLCipher-encrypted)** → `SummarizerProvider` → note markdown → atomic **Obsidian `.md`** export.
- **Providers (one trait, swappable):** `claude_code` (default), `anthropic` (BYO key in Keychain), `ollama` (local). Cloud-bound text passes the **redaction firewall**.
- **Three consumption surfaces over one store:** the app UI, a local read-only **MCP server** (`127.0.0.1:8765`), and the Obsidian vault.

### Module map (verify against the tree — trust code, not docs)

**Rust** (`src-tauri/src/`): `commands/` (`mod.rs` plus domain modules; Tauri commands), `lib.rs` (handler registry + setup), `state.rs` (`AppState`), `error.rs` (`AppError`/`Result`), `events.rs`; `storage/` (`db.rs` plus domain `*_store.rs`, `migration.rs` = SQLCipher whole-DB encrypt-in-place, `models.rs`); `crypto.rs` (AES-256-GCM + `encrypt_file`/`decrypt_file`, verify-before-destroy); `secrets/keychain.rs` (keyring + Security.framework user-presence-gated KEK/MK reads, service `com.meetnotes.app`, `MURMUR_DEV_DEK`/`MURMUR_DEV_KEK` debug hatches); `screenshare.rs` (best-effort auto-relock, crash-safe `CGWindowList`); `audio/` (`recorder`, `system`, `mixer`, `merge`, `wav`, `listener`); `transcribe/` (`whisper`, `model`, `live`, `types`); `reason.rs` + `reason/` (`sidecar.rs`, `afm.rs`) with the killable helper crate at `crates/murmur-brain/`; `pipeline.rs`; `mcp.rs`; `summarize/`; `settings/config.rs`; `export/`.

**Angular** (`src/app/features/`): `analytics`, `ask`, `bar`, `detail`, `folders`, `graph`, `library`, `onboarding`, `record`, `settings`. Services: `core/ipc.service.ts`, `services/{folders,toast,screen-share}.service.ts`, `core/models.ts`.

## Backend server — a SEPARATE repo at `../murmur-server/`

The accounts + sharing backend is **not in this repo** — it lives in the sibling checkout
`../murmur-server/` (GitHub `murmur-io/murmur-server`). Murmur is local-first and fully usable with
**no account**; this server is an **opt-in Tier 1** zero-knowledge relay that unlocks E2EE note +
Org "Shared Brain" sharing. It stores only **ciphertext blobs, wrapped keys, and public keys** —
never plaintext.

**When a task touches the backend/server** — accounts (OPAQUE login), link-share, Murmur↔Murmur
invites, Org sync, the sharing wire format, or anything the app calls over HTTPS — **read
`../murmur-server/` for the real server-side implementation**; do not reason from the client alone.
It is a Rust workspace:
- `crates/murmur-protocol` — the shared E2EE envelope + wire format, **compiled into BOTH the Tauri
  client (this repo) and the server**, so a format change must land in both or it's a compile error.
  A client-side sharing change usually has a server-side counterpart here (`MIT OR Apache-2.0`).
- `crates/murmur-server` — the axum + Postgres service (`AGPL-3.0`), deployed on **Railway**.

Authoritative design spec lives in THIS repo:
`docs/superpowers/specs/2026-07-04-murmur-server-spec.md` (accounts via OPAQUE, modes A/B, the
threat matrix §1.1, the one-way two-domain rule §9). Deploy / redeploy / logs / env: follow the
runbook `../murmur-server/DEPLOY.md` (Railway, GraphQL API not CLI) — never hand-roll ops.

## Common commands

```bash
# Dev (the MURMUR_DEV_DEK hatch avoids per-rebuild Keychain re-prompts; see .agents/skills/tauri-dev)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [murmur-io/murmur](https://github.com/murmur-io/murmur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
