---
trigger: always_on
description: Operating notes for Claude Code working in this repo. The user-facing pitch, install, layout, and build instructions live in [`README.md`](./README.md). This file is the *how* — rules, conventions, and recipes specific to changing the codebase.
---

# CLAUDE.md

Operating notes for Claude Code working in this repo. The user-facing pitch, install, layout, and build instructions live in [`README.md`](./README.md). This file is the *how* — rules, conventions, and recipes specific to changing the codebase.

## Naming — don't "fix" these

The display name is **FerrisScope** (mixed case). Everything technical stays lowercase: crates (`ferrisscope-core`, `ferrisscope-kube-ext`, `ferrisscope-agent`, `ferrisscope-app`), the binary `ferrisscope`, the Tauri identifier `dev.ferrisscope.desktop` (Tauri 2 warns against an `.app` suffix on macOS), disk paths under `~/.config/ferrisscope/`, and the SSA field manager constant `"ferrisscope"` (`fetch::FIELD_MANAGER`).

The `core` crate must stay Tauri-free so a future TUI / CLI can reuse the engine. If you're adding `tauri` to `core/Cargo.toml`, stop and reconsider.

## Hard architectural rules

- **One reflector per `(cluster, resource_kind)`.** Never start a second watch for data already cached.
- **Reflectors are lazy** — started on first subscribe, torn down a few seconds after the last unsubscribe.
- **Frontend is a mirror, not a source of truth.** All canonical state lives in Rust.
- **A "cluster" owns a task supervisor.** Disconnecting aborts the supervisor — no orphaned tasks, no leaked sockets.
- **Frontend business logic budget is near zero.** View state in Zustand; everything else over Tauri commands.

## Think it through before shipping

When asked to build, fix, or improve something, actively look for ways it can break before reporting done. Don't stop at "the happy path compiles."

- **Walk the failure modes.** Empty / missing / malformed input. Concurrency and cancellation. Who frees the resource. 403/404/409/410 from the apiserver. Long names, missing namespace, disconnected cluster, stale reflector.
- **Fix root causes, not symptoms.** For bugs, ask *why* the bad state became reachable before patching where it surfaced. Add a regression test.
- **Finish the loop.** Loading, empty, error, and stale states are part of the feature — a button with no disabled state during in-flight work is unfinished.
- **Surface what you didn't cover.** Wrong assumptions, untested UI paths, related bugs you spotted — name them in the response rather than implying success.

Scale the paranoia to blast radius: more for `fetch.rs` / agent loop, less for a chip-color tweak.

## Conventions

- **Rust.** `rustfmt` defaults, `clippy::pedantic` opt-in per crate. Errors via `thiserror` in libraries, `anyhow` in the binary. No `unwrap()` outside tests. Tracing via `tracing` + `tracing-subscriber`, never `println!`.
- **TypeScript.** `strict: true`, no `any`. Tauri command bindings via the typed wrapper in `ui/src/api.ts` — never call `invoke()` with stringly-typed names from components.
- **Commits.** Conventional commits (`feat:`, `fix:`, `refactor:`, `chore:`, `docs:`). Small and reviewable.
- **Task completion check.** Any task touching Rust → `cargo fmt --all -- --check` before reporting done. CI fails on drift.
- **Tests.** Backend: unit tests next to code, integration tests in `tests/` against a `kind` cluster (gated behind the `integration` feature). Frontend: Vitest for utilities; Playwright is reserved for once the UI stabilises.
- **Every new functionality must ship with tests.** No exceptions, including bug fixes (regression test) and small refactors that change behaviour. New Rust function / module → `#[cfg(test)]` next to it; new Tauri command → command-shape test in `ui/src/api.test.ts` plus the relevant backend integration test; new store reducer → reducer test in `ui/src/store.test.ts`; new component or atom → render + interaction test next to it (`*.test.tsx`); new pure utility under `ui/src/lib/` → unit test next to it. PRs without tests are incomplete — push back rather than land code that drops coverage. The only acceptable exception is purely visual tweaks (colour, spacing) where there is genuinely nothing to assert beyond what the snapshot would already show, and even then prefer a render test that pins the structural class names / aria attributes.

## Design system

The visual + interaction reference for the **Default theme** lives in [`./design/Helmsman v2/`](./design) (`hv2-rail.jsx`, `hv2-dock.jsx`, `hv2-settings.jsx`, `hv2-ui.jsx`, plus `Helmsman v2.html` for previews). It's the source of truth for layout, spacing, colors, motion, and component anatomy for that theme.

Other themes (Lens, VS Code, Readable) are intentional siblings without matching `hv2-*.jsx` artifacts. They diverge on typography, sizing, chrome, and palette by design. Don't try to "harmonize" them back into Helmsman.

- **Read before you build (Default).** Open the matching `hv2-*.jsx` before changing any UI surface in the Default theme (rail, dock, modal, table, palette, settings, fleet card, status pill, container dot…). Don't reinvent.
- **Don't edit `design/`.** Reference artifact, not application code. Push divergences back into the relevant atom in `ui/src/components/ui/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dzcorp/FerrisScope](https://github.com/dzcorp/FerrisScope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
