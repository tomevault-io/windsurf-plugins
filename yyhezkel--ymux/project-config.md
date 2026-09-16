---
trigger: always_on
description: This file is read at the start of every Claude session working on ymux. Keep it small. Deep references live in `docs/`.
---

# CLAUDE.md

This file is read at the start of every Claude session working on ymux. Keep it small. Deep references live in `docs/`.

## Where to start

- **`docs/vault/INDEX.md` — the vault. Read the page that covers an area BEFORE opening its source.** ~3k lines of prose standing in for ~90k lines of code, covering 96% of the tree. It is enforced: `scripts/vault-check.mjs` hashes every covered file and ci-windows fails when the prose and the code drift. When you change covered code, update the covering page in the same commit and run `node scripts/vault-check.mjs --write` — `[vault-skip]` in the PR title is the escape hatch. Details: `docs/CONTRIBUTING.md` § Updating the vault.
- `docs/ARCHITECTURE.md` — system map
- `docs/CONTRIBUTING.md` — recipes, style, commit conventions
- `docs/RELEASING.md` — version cut process
- `docs/DECISIONS.md` — **READ FIRST**: open threads + decisions log
- `docs/ZELLIJ.md` — zellij's CLI + config surface as our 0.44.3 binary reports it; read before adding a verb, don't guess from zellij.dev
- `docs/BRIEF.md` — the agent-brief wire format (`[ymux-brief]`), the Queue panel and the Briefing card; read before touching the brief parser or its surfaces
- `docs/COMPETITIVE-SCAN.md` — survey of the 8 GitHub projects named `winmux`, ideas inventory, Secrets Vault design (pre-rename doc, kept verbatim: it is about *other people's* repos and is what motivated the move to YMUX)
- `docs/IDEAS-RANKING.md` — decision table for the ideas inventory (MUST / SHOULD / COULD)

## Session workflow (memory arch)

- **`git fetch` FIRST, every session — and again before every merge/push.** ymux is worked on from several machines and servers at once (Yossi's box, collaborators, cloud sessions), so `origin/main` moves while you work. Start with `git fetch origin && git status`; if `main` is behind, pull before touching anything. Re-fetch before any merge or push — a plan built on a stale tree is worse than no plan. This is not theoretical: on 2026-08-09 a collaborator's macOS port landed 5 commits on `origin/main` mid-session and was only noticed by accident at the end. Never assume the tree you started with is current.
- **PROGRESS.txt** — append after every significant change (timestamp, task, files, result). NEVER overwrite. Too big → rename `PROGRESS_OLD_<date>.txt`, start fresh.
- **FOLLOWUPS.md / BACKLOG.md** — read both at session start. Open P0/P1 in FOLLOWUPS → surface before new work. Out-of-scope bug found in passing → one line to FOLLOWUPS (P0-P3, file:line, repro). Out-of-scope idea / mock-stub debt → BACKLOG. Never silently leave broken state. **FOLLOWUPS.md holds OPEN items only** — when one closes, move it to `FOLLOWUPS-ARCHIVE.md` with its full text plus a note saying how it closed; do not delete it and do not leave it in place as `[x]`. That archive is NOT read at session start, so nothing still needing action may be parked there. **Cite an entry by its text, never by `FOLLOWUPS.md:NN`** — the line numbers shift every time an item is added or archived, and two entries had already rotted into pointing at the wrong lines.
- **Past-work lookup order** — before re-investigating: 1) `PROGRESS.txt` + `PROGRESS_OLD_*` 2) `FOLLOWUPS-ARCHIVE.md` (a closed entry keeps the root cause, which is often not the one first written down) 3) `git log --all --oneline --grep=<keyword>` 4) memory search 5) `docs/*.md` + this file.
- **"Verified" = real run, not compile.** Build/type-check pass = syntax only. Say "compiles, untested" until run live.
- **Sync this file with code.** New port/service/endpoint/schema/deploy step → update the matching doc in the same commit. Same for the vault page covering the code you touched — CI checks that one for you (Rule #18).

## Decisions & open threads

When an idea or design question comes up:

1. If it's resolved in the same message, do it — no log entry needed.
2. If a decision is made but action is deferred, log it under **Decided** in `docs/DECISIONS.md` with the outcome and a deferral note.
3. If it stays open (user hasn't decided, blocked on input, flagged for later), log it under **Open** in `docs/DECISIONS.md` with options and current state.

When starting a new session, scan the **Open** section. Don't let threads die silently — if something's been pending a while, surface it.

## Pinned deps

- `tauri = "=2.10.3"` with `features = ["unstable"]` (app/src-tauri/Cargo.toml). The unstable feature gates `Window::add_child`, which Phase 53 uses to mount per-workspace browser webviews inside the main window. Bumping tauri requires verifying `Window::add_child`'s signature hasn't changed and the multi-webview shape still compiles. Push the bump and let CI type-check it (Rule #17 — no local `cargo check`), then smoke-test the workspace Browser window (sidebar 🌐 → open / hide via a modal / navigate / close).

## Off-limits paths

- `backup-phase23-*` folders — never touch
- Repo-root `.bat` / `.ps1` helper scripts the user maintains — never touch
- `release_notes.md` — do not commit
- `remote-manifest.json` timestamp churn — discard unless the SHA actually changed
- Linux CLI binary rebakes itself on release builds (CARGO_PKG_VERSION) — expected, commit as part of the release

## Release safety


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yyhezkel/ymux](https://github.com/yyhezkel/ymux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
