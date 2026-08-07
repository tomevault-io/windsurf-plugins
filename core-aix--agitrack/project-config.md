---
trigger: always_on
description: aGiTrack stands for agent + git tracking. It is a Python library and interactive CLI that combines coding-agent command line tools with automatic Git commits so agentic code changes are easier to trace.
---

# aGiTrack Requirements

aGiTrack stands for agent + git tracking. It is a Python library and interactive CLI that combines coding-agent command line tools with automatic Git commits so agentic code changes are easier to trace.

## Goals

- Provide a common interactive interface for coding-agent backends and Git commit automation.
- Support OpenCode and Claude (Claude Code) as interchangeable backends; every aGiTrack feature works the same regardless of the selected backend.
- Keep the user experience similar to the selected backend CLI, while adding aGiTrack commands for agent switching, user commits, staging, status, and configuration.
- Make agentic code changes traceable by automatically committing code changes made after agent prompts.
- Support running aGiTrack from any folder against a target working repository.

## Documentation

- `docs/user-flow.md` holds the complete, graph-rendered map of aGiTrack's interactive user flow (Mermaid flowcharts: startup gating, base↔worktree commit/stage/sync, the agent turn and integration, copy-back, the Ctrl-G menu, self-update, exit). It is linked prominently from the README. **It must be kept in sync with the actual flow:** whenever the user flow changes — a new or reworded prompt, a changed option label, a new decision based on file/commit/session status, a new Ctrl-G command, or a changed exit/update path — update the matching diagram in the same change.

## Writing Style

- **Do not use em-dashes** (Unicode U+2014, or the `&mdash;` / `&#8212;` HTML entities) in prose aGiTrack produces or ships: the website (`docs/index.html`, `docs/docs.html`), the README, docs, and other user-facing text. Use a colon, comma, parentheses, or a period instead.

## Commit Types

- Agent commits use the `<aGiTrack>` tag.
- User-triggered commits use the user-provided subject without an aGiTrack subject tag.
- Before an agent acts, if user changes already exist, aGiTrack creates a separate user commit first.
- A commit is created only when code has changed and staged changes exist.
- **A slash command that does real file work must be committed and merged like any other turn.** Some commands are pure directives (`/model`, `/clear`, `/compact`) and produce no changes, but others — most importantly `/init`, which writes `CLAUDE.md`/`AGENTS.md` — drive the agent to edit files. Those file changes must be auto-committed and merged into the base just like a typed prompt. The trap is backend-specific transcript shape: **Claude** records a `/init` as a `<command-name>` artifact row PLUS a separate `isMeta` user row holding the command's expanded instructions, and the transcript parser drops both as non-prompts — so without special handling NO turn opens and the file-creating assistant work is silently dropped (no commit, no merge). `transcripts/claude.py` therefore detects a `<command-name>` invocation (`_slash_command_name`) and lets the following expansion row (`_command_expansion_text`) open a turn labelled with the command (e.g. `/init`); commands with no expansion still open no turn. **OpenCode** already opens a turn for any `role == "user"` message, so a completed `/init` is captured natively — but a turn only commits when it has a `final_response`, which OpenCode fills only once the run reaches its `final_answer`/`finish == "stop"` message, so an interrupted command run legitimately has none. Verified on both backends by running a real `/init` end to end (Claude writes `CLAUDE.md`, OpenCode writes `AGENTS.md`) and confirming the parser yields a committable turn. Keep this distinct from the trace-exclusion rule below: pure directive commands stay out of the trace; work-doing commands become real, committed turns.

## Manual-Commit Mode (`--manual-commits` / `-m`, config `manual_commits`)

- An opt-in, strictly additive mode (default off). It **always runs without a worktree**: enabling it forces `use_worktrees=False` in `cli.py` (`--manual-commits`/`-m` implies `--no-worktree`), so it never runs in the worktree model. Everything below runs only when `ProxyRunner._manual_commits` is true; the default worktree and existing `--no-worktree` paths are untouched.
- **Goal:** natural, user-triggered commits. The agent edits the current branch directly, but aGiTrack does **not** commit each turn onto the branch. Each turn is recorded as a hidden **latent commit** on `refs/agitrack/manual/<session_id>` (chained, first-parent = the branch HEAD / previous latent tip) while **HEAD never moves**. The working tree already holds the agent's edits, so the latent commit is a durable *record* of the turn (its snapshot tree + the usual `<aGiTrack>` subject + `# Interaction Trace` + `# aGiTrack Metadata`), used for the dashboard's pending view and for folding at commit time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [core-aix/agitrack](https://github.com/core-aix/agitrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
