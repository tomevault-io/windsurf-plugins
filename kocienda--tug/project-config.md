---
trigger: always_on
description: Tugtool is a developer tool suite. Its centerpiece is the **Session card** — a graphical surface where shell commands and AI interactions coexist in one UI, replacing the terminal. The suite includes tugcast (WebSocket multiplexer), tugcode (Claude Code bridge), tug (the unified developer CLI — changes & commits, dashes, host plumbing), tugdeck (browser frontend), tugplug (agentless skills), and Tug.app (macOS host).
---

# Claude Code Guidelines for Tugtool

## Project Overview

Tugtool is a developer tool suite. Its centerpiece is the **Session card** — a graphical surface where shell commands and AI interactions coexist in one UI, replacing the terminal. The suite includes tugcast (WebSocket multiplexer), tugcode (Claude Code bridge), tug (the unified developer CLI — changes & commits, dashes, host plumbing), tugdeck (browser frontend), tugplug (agentless skills), and Tug.app (macOS host).

## Git Policy

**ONLY THE USER CAN COMMIT TO GIT.** Do not run `git commit`, `git push`, or any git commands that modify the repository history unless explicitly instructed by the user. You may run read-only git commands like `git status`, `git diff`, `git log`, etc.

**Exceptions:**
- Autonomous implementation: when the user explicitly authorizes autonomous sub-step execution (e.g., "go on your own"), commit after each sub-step using the `/tugplug:draft` skill's message style. Report each commit hash and message.
- The `dash` and `dash-implement` skills commit on their **dash worktree** (never on `main`) via `tugutil dash commit`, as part of running a recipe / dash. `main` is only updated by the user's landing gestures.

The `/tugplug:draft` skill **never commits** — it authors the session's landing draft via `tugutil draft set`. Landing is the user's act: `/commit` (main lane) and `/dash-join <name>` (dash lane) in the Session card are the landing gestures.

## Writing prose the Session card renders

**Backtick every file path you write, every time.** Your transcript prose is rendered markdown, and a path in backticks and the same path bare are one reference wearing two faces — the reader has to work out that the difference means nothing. Backticks are the author's own emphasis and the renderer may not invent them, so consistency is yours to supply. The same goes for commands and symbols. A commit sha is the one thing you write **bare** in backticks — `` `63de5762a` ``, never `commit 63de5762a` — because the app supplies the word and displays it as `commit:63de5762a`.

Clickability is not what backticks are for: the resolver confirms a path and rules it whether or not you formatted it as code. This is about the sentence reading as one voice. The doctrine is [tuglaws/entity-presentation.md](tuglaws/entity-presentation.md#the-house-voices-backtick-every-path).

## The standalone contract

Tug is distributed as `Tug.app` to people whose projects have nothing to do with this checkout: no `tuglaws/`, no `justfile`, no `CLAUDE.md` of ours, no `~/.local/bin` symlinks, possibly no `jq` or `bun`. Everything the AI needs to drive Tug on such a project must be inside the bundle — the binaries in `Contents/MacOS/` and the plugin at `Contents/Resources/tugplug/`. The contract and its guards are in [tugplug/CLAUDE.md](tugplug/CLAUDE.md#the-standalone-contract): `just tugplug-lint` (in `just lint`) refuses checkout-only shapes under `tugplug/`, and `just test-standalone` (in `just test`) drives the real hook script and dash verbs from a scratch project with an empty PATH. Anything Tugtool-specific a skill would like to say — which recipe builds, which tests are green — belongs in `.tugtool/config.toml` or in this file, never in the plugin.

## Repository Structure

| Directory | Description |
|-----------|-------------|
| `tugrust/` | Rust crates (tugcast, tug, tugexec, tugbank, tugcore, the `*-core` libraries — tugutil-core/tugdash-core/tugchanges-core — and supporting libraries) |
| `tugproto/` | Shared protocol / message types (TypeScript) |
| `tugcode/` | Claude Code bridge (stream-json IPC); bun-compiled binary |
| `tugdeck/` | Web frontend (the Session card lives here) |
| `tugapp/` | Swift macOS app (Tug.app host) |
| `tugplug/` | Claude Code plugin (agentless skills: dash/dash-plan/dash-devise/dash-review/dash-implement/dash-audit/draft/tripwire). A dash's documents live at `.tug/dashes/<name>/` and are never tracked. |
| `tuglaws/` | Architecture laws + design decisions — the curated durable doc surface |
| `tests/` | App-test harness that drives the real Tug.app |

## Build Policy

**WARNINGS ARE ERRORS.** The Rust workspace enforces `-D warnings` via `tugrust/.cargo/config.toml`.

- `cargo build` will fail if there are any warnings
- `cargo nextest run` will fail if tests have any warnings
- Fix warnings immediately; do not leave them for later

## Testing

Run Rust tests with:
```bash
cd tugrust && cargo nextest run
```

### App-tests: run a selection, never a sweep

Every app-test launches its own `Tug.app` subprocess and the whole invocation is serialized behind a machine-wide gate, so running the corpus is expensive. **Selective runs are the default.**

```bash
just app-test-changed        # the everyday command — derived from your working diff
just app-test-select         # print that selection without running it
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kocienda/tug](https://github.com/kocienda/tug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
