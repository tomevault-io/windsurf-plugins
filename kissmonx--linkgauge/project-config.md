---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Human-facing docs live in
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Human-facing docs live in
[`README.md`](README.md) / [`README.zh-CN.md`](README.zh-CN.md) — read the README first for
what the product does; this file covers how to change it without breaking its conventions.

## What this project is

LinkGauge is a desktop network performance tester: Rust + Tauri 2 backend, Vue 3 + TypeScript
frontend. Ping uses the system command; all TCP/UDP testing runs on
[riperf3](https://github.com/therealevanhenry/riperf3), a pure-Rust iperf3 implementation
vendored at `vendor/riperf3` and compiled **into** the application.

**The zero-external-binary property is a product guarantee, not an implementation detail.**
The README and the installer notes both promise that no iperf3 (or ssh, or any other network
tool) executable is bundled, resolved, or spawned. Do not introduce a dependency that shells
out to one. When a feature needs a protocol, add a pure-Rust library — that is why the SSH
console uses `russh` rather than `Command::new("ssh")`.

## Commands

```bash
npm ci                       # install frontend deps
npm run tauri dev            # run the desktop app
npm run dev                  # browser-only preview (simulated data, see "Preview mode")
npm run build                # vue-tsc --noEmit && vite build  — run before every commit
cargo test   --manifest-path src-tauri/Cargo.toml
cargo check  --manifest-path src-tauri/Cargo.toml
cargo fmt    --manifest-path src-tauri/Cargo.toml -- --check
cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings
npm run tauri build          # NSIS installer (Windows) / add -- --bundles appimage,deb on Linux
```

All four checks — `npm run build`, `cargo fmt --check`, `cargo clippy -- -D warnings` and
`cargo test` — must pass before you commit; CI runs exactly these on Windows and Linux
(`.github/workflows/ci.yml`), so a local failure is a red build. The tree is currently clean
on all four, and `clippy` is gated at `-D warnings`, so a new warning breaks the build rather
than accumulating.

## Layout

```text
src/                    Vue 3 frontend
  App.vue               all app state, task queue, cross-window sync, event handling
  components/           panels (config / dashboards / status / console), chart, icons
  types.ts              frontend half of the IPC contract
  i18n.ts               every user-visible string
  terminal.ts           SSH console line buffer
  styles.css            entire visual system (no scoped styles in components)
src-tauri/src/
  lib.rs                Tauri builder, window creation, command registration
  runner.rs             riperf3 client/server tasks, Ping, per-test log files, cancellation
  ssh.rs                SSH remote console: session, PTY shell, output decoding
  report.rs             HTML/PDF report generation
  settings.rs           settings.json read/write, config export
  system.rs             local network interface information
  models.rs             Rust half of the IPC contract
vendor/riperf3/         vendored engine + local patch — see "Vendored engine"
```

## Conventions that are easy to get wrong

### Comments and strings are in different languages, deliberately

- **Source comments are Chinese.** Match that. Comments explain *why*, not *what* — most
  existing comments document a non-obvious constraint (a race, a protocol quirk, a layout
  trap). Write that kind, or none.
- **User-visible strings are never hard-coded.** They go through `src/i18n.ts` on the
  frontend and `tr()` / `tr_format!` in Rust.

### Adding a UI string

`src/i18n.ts` defines `const zh = {...}`, derives `MessageKey` from it, then declares
`const en: Record<MessageKey, string>`. Adding a key to `zh` makes the compiler demand the
English one — so add to `zh` first and let `vue-tsc` find the gap. Keys are dotted by area
(`cfg.`, `srv.`, `ssh.`, `dash.`, `sdash.`, `st.`, `rep.`, `err.`, `log.`, `preview.`).

### Adding a backend message

Rust user-facing text uses `tr(locale, "中文", "English")` or the `tr_format!` macro. The
locale is read **live** from `AppState.locale` via `current_locale()` on each emission, not
captured at session start — switching language mid-run must change subsequent engine logs.
`runner.rs` and `ssh.rs` each define their own local copies of these helpers.

### Adding a Tauri command

1. Write it in the owning module with `#[tauri::command]`.
2. Register it in the `invoke_handler![...]` list in `lib.rs` — forgetting this compiles fine
   and fails at runtime.
3. Request/response structs get `#[serde(rename_all = "camelCase")]`, and the matching
   TypeScript interface goes in `src/types.ts`. These two files are one contract; change them
   together.
4. New long-lived state goes in a `struct XxxState` registered with `.manage()`.

### Adding shared state (the multi-window rule)

Client and server are tabs that can be dragged out into separate OS windows (`main`,
`client`, `server`). Every window runs the same `App.vue` and they stay in lockstep by
broadcasting a full state bundle. **Any state that must survive a tab being detached has to
be wired in four places:**

1. the `SyncState` interface in `types.ts`
2. `syncBundle()` in `App.vue`
3. `applySync()` in `App.vue`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KISSMonX/LinkGauge](https://github.com/KISSMonX/LinkGauge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
