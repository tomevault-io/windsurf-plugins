---
trigger: always_on
description: Virtual guitar rig/pedalboard in Rust + Slint. macOS, Windows, Linux.
---

# OpenRig — Claude Code

Virtual guitar rig/pedalboard in Rust + Slint. macOS, Windows, Linux.

## Laws

1. **The main folder is the owner's.** The agent never touches it: no `git`, `Edit`/`Write`, stage, worktree, or file created via Bash (`touch .dev-rules/…`, `mkdir`, temp files) — not even when the owner says "turn the gate off". Work only in `.solvers/issue-N/`, a real clone made by `scripts/solver-setup.sh <N> <branch> [release-base]`. Where the clone lives is the machine owner's call: when his global `CLAUDE.md` names a workspaces folder (e.g. an external disk), pass it as `OPENRIG_SOLVERS_ROOT` and the clone and its `target/` live there, with `.solvers/issue-N` a link to it; when it names none, the clone lives in `.solvers/issue-N`. `git worktree` is forbidden. If something must happen in the main folder, hand the owner the command. Details: `docs/development/gitflow.md` → "Isolated workspace".
2. **Short chat, in pt-BR.** 1–2 sentences; no tables, headers, recaps or "next steps". A question asks one thing in one line. Long diagnosis goes to the issue. Repo content (code, comments, docs, commits, branches, PRs, issue comments) is English; only `README.pt-BR.md` / `README.es-ES.md` are not.
3. **Validation checklist** whenever the owner must validate (ear, eye, hardware, real app), in chat AND on the issue: the main-folder command `git fetch && git checkout {type}/issue-N && git pull`, the absolute `run:` line printed by `scripts/solver-setup.sh`, then `1. [ ]` items of what only he validates. The only list allowed in chat. Format: `docs/development/gitflow.md` → "Validation checklist".
4. **Never assume; never ask the obvious.** Scope, data model, behaviour, layer, file or A-vs-B unclear → stop and ask one short question at a time, never pick an alternative he did not pick. Already agreed or inferable from code/context → decide and go.
5. **Issue content lives in the issue.** History, diagnosis, hypotheses, measurements, decisions, "why" stories and status go to `gh issue comment`. The repo states only what is true now: no `#N` in `CLAUDE.md`, skills, hooks, `docs/**` or code comments. Found one in a file you touch → post it on its issue, leave the current truth.
6. **N streams = N isolated pipelines.** Own input, runtime, output, rebuild, failure, latency and CPU scheduling. Runtimes on the I/O path are selected by stream/device identity — never by rate, never "all that match". Mixing happens only in the backend (cpal/JACK). Code where two streams know each other is a bug: stop, report, fix. Details: `docs/audio-config.md` → "Isolation is by stream identity". No realtime worker carries more DSP than one core handles with headroom, not even inside one chain: the parallel paths of a split (any independent branch) each run on their own realtime worker, never serially on the chain's single worker; before delivering, measure per-thread CPU on the rig (`ps -M -p <pid>`: no `97R` thread near 100%).
7. **One file, one responsibility.** Every production file opens with `//! Responsibility: <one sentence>` (`// …` in `.slint`) with no "and"/list. A new responsibility is a new file, never the end of an open one. Caps `.rs` 600 / `.slint` 500 are only the alarm; tests have no cap; `mod.rs`/`lib.rs` < 100 LOC. Details: `docs/development/file-organization.md`.

## Real-time invariants — never regress

1. Round-trip latency. 2. Audio quality (noise, aliasing, THD, frequency response). 3. Stream stability — zero xruns, dropouts, clicks. 4. Stream isolation (law 6). 5. Streams are stereo inside: mono → `Stereo([s,s])`, DualMono → independent `[L,R]`; mono out only via `OutputBlock.mode == mono` (`apply_mixdown`); never auto-pan. 6. Stable callback jitter. 7. Audio-thread CPU. 8. Zero allocation, lock, syscall or I/O on the audio thread. 9. Golden samples within tolerance. 10. Per-stream volume is immutable unless the owner asks; if `crates/engine/src/volume_invariants_tests.rs` breaks, the source is wrong, not the test.

**Red flags — stop and report:** new xrun/click, latency +1 ms unexplained, golden tests failing, `Mutex`/log/I/O in processing, "the sound changed on one OS", "the volume changed", any state shared by 2+ streams.

**Trade-offs:** sound + stability + isolation > latency > audio-thread CPU > cross-platform > ergonomics > new features. A feature never justifies a regression; discuss first.

**Cross-platform:** never hardcode paths; Linux/Orange Pi/JACK fixes stay behind `cfg`.

## Code

- Zero warnings. Blocks never name specific models/brands. Constants once. No visual config in business logic.
- Every state change is a `Command` (`crates/application/src/command.rs`); the GUI calls `dispatcher.dispatch`, MCP/gRPC share the variant; never `borrow_mut()` in a callback. Slint is a pure dispatcher (callback → `Event` → pure fn); no `AppWindow` in tests. Core (`State`/`Event`/`Command`/`SideEffect`) has no Slint dep.
- System `config.yaml` vs `project.yaml`: "must this value travel with the project file?" Yes → project. `docs/adr/0003-system-vs-project-config.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpfaria/OpenRig](https://github.com/jpfaria/OpenRig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
