---
trigger: always_on
description: Welcome, Agent. You are tasked with developing and maintaining **Spotifust**. These instructions are deterministic and take precedence over generic best-practice defaults you might otherwise apply. Ambiguity in this document is a bug — if you find a case it doesn't cover, add a clarifying note to `TODO.md` under **Architectural Debt** instead of guessing.
---

# AGENTS.md — AI Agent Operating Manual

Welcome, Agent. You are tasked with developing and maintaining **Spotifust**. These instructions are deterministic and take precedence over generic best-practice defaults you might otherwise apply. Ambiguity in this document is a bug — if you find a case it doesn't cover, add a clarifying note to `TODO.md` under **Architectural Debt** instead of guessing.

Target performance envelope: **< 25MB RAM baseline**, single binary, zero external runtime dependencies (no Node, no bundled browser engine).

---

## 0. Definitions (read before anything else)

To avoid the two most common failure modes — treating an async task as a process, and treating every keystroke as a "task" — the following terms are fixed for the rest of this document:

| Term | Means | Does NOT mean |
| :--- | :--- | :--- |
| **Process** | A separate OS-level process (`std::process::Command`, a sidecar binary, a spawned executable) | A `tokio::spawn`ed async task — those are explicitly required (see §5.B) and run inside the same process/address space |
| **Atomic task** | One checklist item (`- [ ]`) as it appears in `TODO.md`'s Development Backlog | A single line of code, a single function, or a single file edit |
| **Phase boundary** | Completion of every item within a numbered Phase in `TODO.md` | Reaching a compilable intermediate state mid-phase |

---

## 1. Core Operating Constraints

* **No Web Overhead.** Absolute prohibition of web-views, embedded browser engines, or any JS runtime, including for debugging/devtools purposes.
* **Single-Process Monolith.** No `std::process::Command`, no sidecar binaries, no IPC across process boundaries. `librespot` and `rspotify` are compiled directly into the application. `tokio::spawn` for async tasks inside the same process is not only allowed but required — see §5.B.
* **The Elm Rule (Pure MVU).** All UI-relevant state changes flow through `Message` → `update()`. Do not introduce `Rc<RefCell<T>>` or `Arc<Mutex<T>>` inside UI-owned structs to shortcut this. Cross-thread communication (e.g., audio task → UI) must go through `tokio::sync::mpsc` channels surfaced as `iced::Subscription`s, never through shared mutable state.
* **Zero Crashing Policy — with one explicit exception.** See §2.

---

## 2. Error Handling Contract

* **Inside the running application (after the `iced::Application` loop has started):** `.unwrap()`, `.expect()`, and `panic!()` are forbidden. Every fallible operation returns `Result<T, AppError>`, and errors are surfaced to the user via `Message::ErrorEncountered(AppError)`.
* **Bootstrap exception (before the event loop exists):** in `main()`, prior to `iced::Application::run()`, fail-fast with a clear `eprintln!` and `std::process::exit(1)` is acceptable — there is no UI yet to route an error message to. Keep this window as small as possible; move config/env loading into the update loop where feasible instead of expanding this exception.
* **Define one central error type.** Use `thiserror` for `AppError`, with variants per subsystem (`AppError::Auth`, `AppError::Playback`, `AppError::Network`, `AppError::Cache`). Do not let raw `librespot` or `rspotify` error types leak into `Message` variants — wrap them.
* **Never use `Mutex`/`RwLock` poisoning as a control-flow signal.** If you find yourself calling `.lock().unwrap()`, that's a sign shared-state locking crept in where it shouldn't have (see the Elm Rule) — refactor to message passing instead of handling the poison case.

---

## 3. `TODO.md` Synchronization Protocol

`TODO.md` is the single source of truth for project state. Treat it as a state machine, not a changelog.

### 3.1 When to touch it

* **On session start:** read `TODO.md`, greet the user with a brief summary of where the project stands, and **ask explicitly whether they want to continue with the next pending task**. If `TODO.md` doesn't exist, create it using the schema in §3.3.
* **On completing one atomic task** (per the §0 definition — one checklist item): mark it `[x]` in the same edit that completes the corresponding code change. Don't batch multiple completed items into a single later rewrite. After marking it done, **ask the user whether to proceed with the next pending item** before starting anything new.
* **On discovering new work mid-task** (a missing edge case, a follow-up refactor): append it to **Architectural Debt** immediately, don't just remember it — you won't carry memory into the next session.
* **Not on every intermediate compile or every function written.** If you're rewriting `TODO.md` more than once per checklist item, you're over-triggering this rule.

### 3.2 How to update it safely

Always re-read `TODO.md` immediately before editing it, even if you wrote it earlier in the same session — do not trust a stale in-context copy. Edit surgically (only the relevant checkbox/section); don't regenerate the whole file from memory, since that risks silently dropping items you didn't fully recall.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gefydev/spotifust](https://github.com/gefydev/spotifust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
