---
trigger: always_on
description: EmailOps is a privacy-first, AI-native email client. AI runs locally by default via an embedded llama.cpp runtime; users can optionally route to a local Ollama instance or to remote providers (OpenRouter). The app supports multiple email accounts with context-based organization.
---

# EmailOps - AI-Native Email Client

## Project Overview

EmailOps is a privacy-first, AI-native email client. AI runs locally by default via an embedded llama.cpp runtime; users can optionally route to a local Ollama instance or to remote providers (OpenRouter). The app supports multiple email accounts with context-based organization.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Desktop Framework | Tauri 2.x |
| Backend | Rust |
| Frontend | React + TypeScript |
| Styling | Tailwind CSS |
| State Management | Zustand |
| Local Database | SQLite (via rusqlite) |
| Vector Search | sqlite-vec |
| Local AI | Embedded llama.cpp (default); optional Ollama (HTTP API) |
| Remote AI (opt-in) | OpenRouter |
| Email Providers | Gmail API, Microsoft Graph API, IMAP/SMTP |

## Sub-area Instructions

- **Backend (Rust + Tauri):** `src-tauri/CLAUDE.md` — Rust coding standards, database conventions, production guardrails, testability practices, build hygiene, backend lessons learned.
- **Frontend (React + TypeScript):** `src/CLAUDE.md` — component structure, Tauri API call centralization, Zustand patterns, frontend robustness, frontend lessons learned.

## Local Workflow — Check the Makefile First

Common development operations live in the root `Makefile`. **Before reaching for an ad-hoc `cargo …`, `npm …`, `python …`, or `sqlite3 …` command, run `grep -E '^[a-z][a-z0-9_-]*:' Makefile` (or skim the file) to see if there is already a target for what you need.** Targets are the canonical, tested way to invoke local tooling for this repo. Examples (non-exhaustive):

- **App run:** `make dev` (repo-local data dir) / `make dev-fresh` (throwaway data dir) / `make dev-trace` (tracing feature enabled)
- **Demo data:** `make demo-db` / `make demo-embed` / `make demo` (run app against demo DB) — plus `-es` variants for Spanish demo data
- **Quality gates:** `make check`, `make lint`, `make fmt`, `make test`, plus `-fast` variants (`test-fast`, `lint-fast`, `clippy-fast`, `check-fast`) that skip the embedded llama.cpp feature for faster iteration
- **Release / signing:** `make bootstrap-mac`, `make build-mac`, `make verify-mac`; Linux/Windows equivalents: `make bootstrap-linux`/`bootstrap-windows`, `build-linux`/`build-windows`, `verify-linux`/`verify-windows`, `dist-linux`/`dist-windows` — see "Linux / Windows Release Builds" below
- **Hooks / deps:** `make install`, `make hooks`, `make audit`, `make clean`

When you do need to run something the Makefile does not cover, prefer extending it (add a new target) over scattering one-off shell snippets across the codebase or your chat output — that way the next agent or developer can find it the same way.

**Keep the Makefile thin — it is an index, not an implementation.** A target's recipe should be a one-or-two-line invocation. Any recipe that needs multi-line shell logic (conditionals, loops, file generation, output capture) belongs in a standalone script under `scripts/`, called by a thin Makefile target (existing examples: `scripts/cli_bench.sh`, `scripts/eval_all.sh`, `scripts/fetch_bundled_models.sh`).

## Agent self-validation with `emailops-cli`

**When the user reports a bug in chat, drafts, classification, lenses, retrieval, memory, or any other AI feature, drive the fix through the `fix-ai-bug` skill** (`.claude/skills/fix-ai-bug/SKILL.md`). It codifies the loop in this section — frame → CLI repro → root-cause → fix (with confirmation gated to genuine design forks) → re-run until green → gated graduation into `private-evals/` — and reports back in a chat-style format that hides the raw `--trace` payload by default. Invoke it whenever a bug report lands, even from a screenshot or trace fragment.

**When the user wants to add a new AI feature or change an existing AI surface** (a new chat tool, a planner/route, a prompt edit, a shortcut fast-path, a classifier/extractor, or a retrieval/draft/memory tweak), **drive the work through the `build-ai-feature` skill** (`.claude/skills/build-ai-feature/SKILL.md`). It runs the build loop — frame → pick the seam → check setup/config fit (context budget + KV-prefix prompt cache) → CLI baseline → TDD the pure planner → wire the thin executor (+ frontend toggle/gating) → mandatory eval gate → gated eval-case graduation — and reports a chat-style before/after delta.

`emailops-cli` (gated behind the `cli` cargo feature) is a headless front-end over the same `services::*` entry points the Tauri commands call — no `AppHandle`, no webview. Use it to **drive real features and assert on structured output** while developing, instead of guessing whether a change works. It operates on the real data dir (SQLite WAL → read commands are safe while the app is open; run heavy write commands — `sync`/`classify`/`embed` — with the app closed).

Every command supports `--json`, which prints one **stable envelope** to stdout so you can parse a single shape regardless of success or failure (logs always go to stderr):

```jsonc
{ "ok": true,  "data": { /* result */ }, "error": null }                       // success

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emailops/emailops](https://github.com/emailops/emailops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
