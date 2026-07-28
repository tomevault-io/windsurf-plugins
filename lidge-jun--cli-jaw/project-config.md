---
trigger: always_on
description: This repository is a Node.js ESM orchestration runtime for boss/employee dispatch, Web UI, browser/CDP automation, Telegram/Discord channels, memory, heartbeat, and PABCD orchestration.
---

# CLI-JAW Claude Guide

This repository is a Node.js ESM orchestration runtime for boss/employee dispatch, Web UI, browser/CDP automation, Telegram/Discord channels, memory, heartbeat, and PABCD orchestration.

## Documentation Map

- Start at `structure/INDEX.md` for the current architecture map.
- Keep `README.md`, `AGENTS.md`, this file, and `structure/AGENTS.md` aligned when command/API/orchestration behavior changes.
- Do not use the old `devlog/structure/` path for architecture docs; the active folder is `structure/`.

## Build & Deploy Contract

- The running server executes compiled `dist/` (`jaw serve` → `dist/server.js`), never the TS sources. After changing `server.ts`/`src/**`/`bin/**`, run `npm run build` before telling anyone to restart; frontend changes additionally need `npm run build:frontend`. Full rules: `AGENTS.md` § Build & Deploy Contract.

## Current Runtime Notes

- PABCD entry is explicit: `jaw orchestrate`, `/orchestrate`, or `/pabcd`. Resume is explicit `/continue`; natural-language “continue/계속/이어서” remains a normal prompt.
- Workflow helper slash commands are `/plan`, `/interview`, `/deliberate`, `/planaudit`, `/review`, `/search`, `/goal`, `/goalplan`, `/team`, `/task`, `/fork`, and `/gd`. Dynamic `/skill:<id>` injects an active skill on CLI/Web. `/plan` is a compatibility guide for users expecting a plan command; it maps to PABCD P and does not create another planning mode. `/planaudit` is the canonical remote-safe spelling; `/plan-audit` is not registered. `/search <query>` forces the active search skill policy, rewrites focused queries, discovers candidate URLs, and uses browser commands only for evidence verification after candidates exist. Bounded automation is a `/goal run ...` subcommand family, not a separate top-level `/autopilot` command; current `/goal run` controls are tracking-oriented runtime gates.
- `/goal plan [hint]`, `/goalplan [hint]`, and `cli-jaw goal plan [hint]` create a pending plan-mode goal. The raw hint is stored separately as `planHint`, not as the durable objective. Agents must refine with `/goal refine <specific objective>`, `cli-jaw goal refine "<specific objective>"`, or `/api/goal` `refine-objective` before checkpoints are accepted.
- Agent pause is a two-tap audited gate. After the first `--agent --audit` attempt, the goal remains persisted as `active` but status/API surfaces expose derived `pauseGate: { armed: true, reason: "pause_gate_pending" }`; one audit/finalizer goal-continuation may run, and if that turn exits with the gate still armed it emits `goal_pause_gate_pending` without scheduling another kick. A second audited pause pauses the goal; a productive checkpoint clears the gate.
- PABCD forward transitions require `jaw orchestrate <phase> --attest '{"from","to","did",...}'` (C→D also `checkOutput`/`exitCode`). Goal mode self-advances but still uses attestation as proof-of-work. See `structure/prompt_flow.md`.
- Optimization/score-maximization goals follow the optimization-loop discipline (LOOP-PHASE-DEATH/CONTINUITY/CANDIDATE-ANCHOR/INSTANCE-CHECK + GATE-ORACLE-VALIDITY):
  classify candidate changes, ban a class after 3 consecutive discards, force evaluator-gate work on repeated D-phase deaths.
  Canonical: dev-pabcd §10, dev-testing §9.5; injected via orchestration template and goal continuation.
- Pre-prompt context hooks: optional `~/.cli-jaw/context-hooks.json`, scopes `main`/`heartbeat`, `cli-jaw hooks inspect`. See `docs/dev/pre-prompt-context-hooks.md`.
- **Telegram Hub** (P0–P4): forum-topic routing via Dashboard `/api/dashboard/telegram-hub`; hub commands `/setthread` `/threads` `/hubhelp`; per-topic `model`/`systemPrompt` overrides (P4). One bot token → one long-poller. See `structure/telegram.md`.
- Bounded local search (prompt-injected): Grep/Glob from one known file or narrow directory only; external/Korean search via `/search` / active search skill. See `structure/prompt_flow.md`.
- `npm test` runs `tests/run.mts` (programmatic driver, `isolation:'process'`). See `structure/infra.md`.
- `/review` is a project-dir review workflow: it uses configured `projectDirs` or a validated recent-context git repo, never JAW_HOME/`process.cwd()` fallback, treats `/review [focus]` user text as the highest-priority scope signal, resolves the review scope from the current conversation focus plus recent goal/chat context and commit history/diffs/worktree/untracked files, saves a Markdown report with scope evidence, and scopes `--fix` to Critical/High findings as new working-tree patches on top of current `HEAD` without rewriting commits. Git ranges are evidence for the conversation-selected work item, not permission to include unrelated recent commits.
- Korean promotional/content writing (홍보 쓰레드, 인스타 카드뉴스, 링크드인, 웹/블로그 게시물, 윤문) is owned by the active private runtime `k-writing` skill, not free-form prose or the retired `k-thread-gen` label. Route by channel first, then run the mandatory workflow: pre-search, content-type detection, 3-candidate hook scoring, tone/module formatting, and anti-AI-tell plus 인간다움 checks before output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidge-jun/cli-jaw](https://github.com/lidge-jun/cli-jaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
