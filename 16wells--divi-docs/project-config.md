---
trigger: always_on
description: > **Read this first.** If you are a Claude agent working on this project (Cowork, Claude Code, or a Claude.ai Project), this file orients you. It takes about 60 seconds.
---

# CLAUDE.md — Divi 5 Technical Documentation (Internal Product)

> **Read this first.** If you are a Claude agent working on this project (Cowork, Claude Code, or a Claude.ai Project), this file orients you. It takes about 60 seconds.

## Who You're Working For

**Skip Shean** runs **16Wells**, a digital marketing consulting practice serving equity, options, and futures businesses. Skip is the human in charge. **This is an internal 16Wells product** — owned and built by 16Wells, not delivered to a client. The "users" are public Divi 5 implementers (humans + AI assistants). When this document says "the product" or "the site," it means the Divi 5 Technical Documentation site. When it says "you," it means the Claude agent reading this file.

**Important branding note:** The company name is always **16Wells** — capital W, no spaces, no suffix. Never "16 Wells," never "16Wells Digital Marketing," never "16Wells LLC." Just 16Wells.

## Project Overview

This is the **Divi 5 Technical Documentation** site — a community-maintained MkDocs Material documentation project for the Divi 5 WordPress theme. The repo lives at https://github.com/16wells/divi-docs and deploys to https://16wells.github.io/divi-docs/ via GitHub Actions. See `01-context/product-charter.md` for the full positioning, audience, scope, and success/kill criteria.

## Key Files

- **SKILL.md** — Complete documentation template, frontmatter schema, writing standards, and content type definitions. READ THIS FIRST before creating or editing any documentation pages.
- **claude-skills/README.md** — References to external Claude skills (e.g. style-guide → Divi variables); install/update from their repos.
- **mkdocs.yml** — Site config and navigation. Update this whenever you add new pages.
- **docs/manifest.json** — Machine-readable map of the site structure.
- **templates/doc-template.md** — Blank template for new reference doc pages.

## How to Get Oriented on Specific Topics

| If you need to know... | Read this |
|---|---|
| What's happening *right now* — in-progress work, awaiting decisions, deployed state | `01-context/state.md` |
| What changed recently across Claude surfaces (chronology) | `01-context/activity-log.md` |
| Open decisions or pending owner items | `01-context/decisions-log.md` |
| Working assumptions, patterns, or gotchas | `01-context/insights.md` |
| Product positioning, audience, scope, success/kill criteria | `01-context/product-charter.md` |
| Build/delivery architecture and scope detail | `01-context/project-scope.md` |

## Iterative Memory — Update As You Go

Work may happen across multiple Claude surfaces (Cursor/Cowork, Claude Code, and Claude chats). These surfaces do not share live memory, so this repository's context files are the handoff mechanism — but only if they actually reflect reality. Memory upkeep is part of the work, not a chore at the end of it.

**Resume discipline — at session start, read in this order:**
1. This `CLAUDE.md` for project orientation.
2. `01-context/state.md` for what is happening *right now*.
3. The most recent 5 entries in `01-context/activity-log.md` for chronology.
4. Open items in `01-context/decisions-log.md`.

If `state.md` is current, the chat history should be irrelevant to resumption.

**Checkpoint triggers — update the appropriate file IMMEDIATELY when:**

| Event | Update |
|---|---|
| A decision is made (closed or opened) | `decisions-log.md` |
| A finding, quirk, gotcha, or pattern is discovered | `insights.md` |
| A meaningful step completes (commit, deploy, gate passed, file created, milestone met) | `activity-log.md` (append) and `state.md` (refresh in-progress section) |
| External system state changes (deploy, config, schedule, data) | `state.md` → External Systems State |
| You pause for human review or sign-off | `state.md` → Awaiting Human Decision |
| You sense context-window pressure or are about to be compacted | full flush of `state.md` first, then continue |
| The user says "checkpoint" or runs `/checkpoint` | full pass: state, activity log, decisions, insights — then commit |

**Commit discipline.** Commit after each significant checkpoint, not in batches at session end. The gap between *logged* and *committed* is what bites at thread-switch time.

**Visibility.** When you update a memory file, say so out loud ("logged decision X," "refreshed state.md"). The discipline has to be observable.

**If your context gets compacted mid-session:** re-read this `CLAUDE.md`, `01-context/state.md`, and the most recent activity-log entry before continuing. Compaction is a mini session-start.

**The mental model:** `state.md` is the live dashboard. `activity-log.md` is the chronological journal. `decisions-log.md` is the ledger. `insights.md` is the working-memory scratchpad.

## Working Style

A few principles for how to actually do the work. Drawn from Andrej Karpathy's guidance on coding with LLMs — they apply whether you are Claude Code, Cowork, or a chat agent making edits.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [16wells/divi-docs](https://github.com/16wells/divi-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
