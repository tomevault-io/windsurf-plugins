---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`ccp` drives an **interactive** Claude Code TUI inside a detached tmux session as if it were `claude -p`. The point is billing: `claude -p` (true headless mode) bills the Agent SDK credit pool, whereas an interactive session bills the subscription pool. So `ccp.sh` automates the interactive TUI — feeding a prompt, auto-answering permission prompts, and scraping the final answer — to get headless ergonomics on the subscription pool. The README tagline is "Headless Claude Code automation without the headless mode."

Pure bash. No build, no package manager, no test suite. Dependencies: `tmux`, `jq`, and the `claude` CLI.

## Commands

```bash
./ccp.sh [-p allow|deny|ask] [-s NAME] [-e KEY=VALUE]... "<prompt>" [-- <claude-options>...]  # run headlessly
./ccp.sh --help                                            # usage
./ccp.sh "say hi"                                          # end-to-end smoke test (see below)
./ccp.sh "review" -- --model opus --add-dir /tmp           # forward claude's own options
shfmt -w ccp.sh hooks/*.sh                                 # format (.editorconfig: 2-space indent, LF)
```

There is no test suite. Verify a change by running the script end-to-end (e.g. `./ccp.sh "say hi"`) — that is the only real test: it exercises the whole launch → readiness → prompt-feed → hook → extract pipeline, and bills a live subscription session (so you must be logged into `claude`).

Before `--`: the prompt is the sole positional arg; everything else is a ccp flag (so a bare `deny` is unambiguously the prompt), and an unrecognized `-flag` is an error. Permission mode (`-p`/`--permission`, default `allow`): `allow` auto-approves every tool call (dangerous Bash is still hard-denied), `deny` rejects everything, `ask` defers to the TUI's normal prompt. `-e`/`--env KEY=VALUE` (repeatable) sets an env var on the launched session via `tmux new-session -e`. `-s`/`--session NAME` names the tmux session (default `cc-<pid>`); it may not contain `.`/`:` and may not collide with an existing session — ccp only ever kills a session it created, never one it didn't.

After `--`: everything is forwarded **verbatim** to the underlying `claude`, so its own options (`--model`, `--add-dir`, `--mcp-config`, …) just work — no per-flag knowledge in ccp, so new claude flags need no ccp change. `--` was chosen over an inline arity table precisely because claude's variadic options (`--add-dir a b c`, `--tools`, `--mcp-config`, …) make inline prompt/value disambiguation impossible. Two passthrough flags are intercepted instead of forwarded: **`--settings <file|json>`** (repeatable) is deep-merged into ccp's generated settings — later values win, and ccp's own `PreToolUse`/`Stop`/`StopFailure` hooks always override the user's for those three events while every other setting (model, env, `PostToolUse`, …) is kept; ccp's `UserPromptSubmit` touch is the lone exception that is *appended* to (not override of) the user's, so a user's own `UserPromptSubmit` hook still runs; **`-p`/`--print`** is dropped with a warning, since claude's headless mode is the very thing ccp replaces (use ccp's own `-p`/`--permission`).

Env overrides: `CCP_READY_TIMEOUT` (default 60s, wait for input box), `CCP_SUBMIT_TIMEOUT` (default 10s, confirm the prompt actually submitted — resending Enter and re-pasting as needed — before giving up), and `CCP_ANSWER_TIMEOUT` (default 0 = wait forever, since prompt complexity is unbounded).

Exit codes: `0` success; `1` runtime failure (session died / answer timeout / bad `--settings`); `2` usage/CLI error (bad args); `4` usage limit reached (quota/credit/429 wall — claude is blocking until reset); `5` turn failed via `StopFailure` (an API error ended the turn); `6` turn needs interactive input (the model called `AskUserQuestion`, unanswerable in a detached run — allow mode only); `127` missing dependency or hook; `130`/`143` interrupted (SIGINT/SIGTERM). Codes `4`, `5`, and `6` each call for a different response from a caller looping ccp: back off until reset (4), retry a transient API error (5), or refine a prompt that needs a human decision (6).

## Architecture

Four files cooperate. `ccp.sh` is the orchestrator; the three hook scripts (plus one inline `UserPromptSubmit` touch, no file) run *inside* the spawned Claude process and communicate back via files.

**`ccp.sh`** — the orchestrator, in numbered steps:
1. Writes a throwaway `--settings` JSON into a `mktemp -d` rundir, wiring four hooks (`PreToolUse` → `auto-permission.sh`, `Stop` → `dump-transcript.sh`, `StopFailure` → `dump-failure.sh`, and `UserPromptSubmit` → an inline `touch <submitted>` that lets step 4 confirm the prompt really reached the model). The user's real `~/.claude/settings.json` is never touched. Hook paths/args are baked straight into the command strings — nothing is smuggled through tmux env. Any passthrough `--settings` is deep-merged underneath this (via `jq`), with ccp's `PreToolUse`/`Stop`/`StopFailure` hooks overlaid last so they win and its `UserPromptSubmit` touch appended.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mthli/ccp](https://github.com/mthli/ccp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
