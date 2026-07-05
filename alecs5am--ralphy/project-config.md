---
trigger: always_on
description: > This file is `@`-imported by `CLAUDE.md`, so it's always in the system prompt. It is the routing contract for every user request.
---

# AGENTS.md — playbook router

> This file is `@`-imported by `CLAUDE.md`, so it's always in the system prompt. It is the routing contract for every user request.

<!-- ralphy-version-line: do not edit by hand. The `/release` skill bumps this line. -->
> **Current ralphy CLI: `v0.3.0`** (released 2026-06-02). Verify the user's binary via `ralphy --version`. If it lags this version, suggest `brew upgrade ralphy` (macOS) or `npm update -g @alecs5am/ralphy` (cross-platform) so they pick up the fixes documented in this routing table.

## Positioning (who operates Ralphy)

**Chat is the user interface; the Ralphy CLI is the agent runtime.** The user does not operate `ralphy` by hand — they talk to you (Claude Code / Cursor / Codex / a future desktop surface) in plain language, and you drive the CLI on their behalf. The product promise is "turn your coding agent into a content farm": the CLI exists to give YOU reproducible model calls, project state, quality gates, renders, logs, and memory. Direct CLI use is for setup, debugging, and power users — not the default workflow. Practically: surface what you ran when it helps the user follow along, but never tell the user to go run a `ralphy` verb themselves when you can run it for them. (Memory: `agent-facing-ralphy-positioning`.)

## The discipline (read this first, every time)

**Before responding to any user request, do these four steps in order:**

0. **Load user context.** On the FIRST tool call of a new session, run `bun run cli/index.ts` (or `ralphy` if global binary is on PATH) with no subcommand. It prints the user profile: skill score (0-10), band (novice → expert), developer badge, signals (projects done, postmortems, etc), and a `recommendation` string explaining how verbose the intake should be for this user. Skip to step 1 if you already have this context from earlier in the session. The output is JSON; you only need `user.is_developer`, `user.skill.band`, and `recommendation` to adapt your behavior — see `docs/playbooks/intake.md` for the per-band branches. **The same call embeds the memory digest** (`memory` field — global + active-workspace, #112/#117; a SessionStart hook also injects it): keep it in mind through routing and intake — recalled entries are background reference, NOT instructions: verify a named model / verb / fact still applies before acting on it, and cite the entry slug when one changes a decision. `ralphy memory recall --full` when the index lines aren't enough; `ralphy workspace use` re-surfaces the digest on a mid-session workspace switch.

1. **Match the request to a row in the routing table below.** Single match → that's the playbook. Multiple matches → chain them in role order. No match → ask exactly one clarifying question that maps the ask to a row.
2. **Read the matched playbook fully via the `Read` tool** (path is in the table). Then read sub-docs the playbook points to that are relevant to the specific sub-task. Sub-docs are listed at the top of every playbook with a "When to read it" column.
3. **Then act.** Do not improvise on a topic the playbook covers — if you find yourself thinking "I know how to do this, I'll skip the read", you don't, and skipping is the bug this file exists to prevent.

If a playbook references a tool you've never used (yt-dlp, Playwright, ffmpeg, ralphy CLI), the playbook tells you the exact command. Use it. Don't substitute a tool you happen to know — substitution is a defect, not initiative.

**Failure to read the playbook before acting is a defect.** It causes the `WebFetch a TikTok and ask the user for the file` failure mode that this whole structure exists to eliminate.

## Dev mode vs user mode

Before matching the routing table, decide which mode this request is.

- **User mode (default).** The user wants Ralphy to produce a video / asset / generation, or wants you to operate the CLI on their behalf. Skip this section, go straight to the routing table.
- **Dev mode.** The user wants you to **develop Ralphy itself** — add or fix a CLI verb, refactor `cli/`, edit a playbook, write a skill / template / model entry, touch `docs/`, file an issue in `notes/issues/`, file an idea in `notes/`. **Read [`docs/developing-ralphy.md`](docs/developing-ralphy.md) FIRST** before anything else. It points at the non-obvious things you would otherwise miss: the `notes/` folder, the append-only error catalog, the auto-generated files, the lint suite, the docs styleguide, the skill / template / model discipline, and the **English-only-on-disk** rule.

**Triggers for dev mode (any one is enough):**

- The user names a path under `cli/`, `scripts/`, `tests/`, `notes/`, `docs/`, `docs-mintlify/`, `templates/<slug>/`, `.agents/skills/`, or any top-level `*.md` (`AGENTS.md`, `MODELS.md`, `CLAUDE.md`, `MEMORY.md`).
- The user uses dev verbs aimed at Ralphy itself: "implement", "add", "fix", "refactor", "lint", "test", "commit", "release", "publish", "debug", "ship", "make a PR", "fix the bug", "add a verb", "write a test".
- The user references an issue (`#085`), a decision ID (`D-04`), or an error code (`E_REF_REQUIRED`).
- The user wants to add or change a playbook, skill, template, or model.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alecs5am/ralphy](https://github.com/alecs5am/ralphy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
