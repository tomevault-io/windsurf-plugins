---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Teaching repository for **GitHub Copilot for Developers** — Tim Warner's 4-hour O'Reilly Live Training. Two things ship from here:

1. **Course materials** — slide deck, course plan, tutorials, certification notes, news.
2. **A working Node demo app** (`/src`) that doubles as the Segment 3 / 4 demo for the cloud agent, Custom Agents, and Agent Skills.

Treat the source of truth for "what's current in Copilot" as `README.md` and the newest `COURSE_PLAN_*.md`. They are refreshed every delivery (current: `COURSE_PLAN_JULY_2026.md`, verified 2026-07-21 — AI Credits now live, Copilot Max tier, GPT-5.6 family, Claude Sonnet 5 and Opus 4.8, cloud agent rename, Copilot Vision GA). When updating model names, prices, or feature GAs, update those two files first; this CLAUDE.md should stay structural.

**Superseded material to never reintroduce**: Copilot Extensions (disabled Nov 10, 2025), premium requests as the billing model (legacy since June 1, 2026), "coding agent" as the primary term (renamed to cloud agent in April 2026), and `gh copilot` as the agentic CLI (the package is `@github/copilot`).

## Repository Layout

```
/                    README.md, CLAUDE.md, AGENTS.md, COURSE_PLAN_*.md, *.pptx
/docs                Tutorials, info articles, reference inputs (NOT code)
  /docs/references   Microsoft Writing Style Guide + fictional-company pool
  /docs/certification GH-300 exam objectives and study notes
/src                 Node + Express demo app + sample data
/scripts             Helper scripts (PowerShell metrics report)
/.github             Copilot customization examples (also configures THIS repo)
/archive             Historical course materials — do not maintain unless explicitly asked
```

Convention: course plan + PPTX at root; prose under `/docs`; code under `/src` and `/scripts`. Don't put new tutorials at root.

## Key Architectural Insight: `.github/` is dual-purpose

Every file in `.github/` does **two jobs simultaneously**:

1. It configures GitHub Copilot's behavior for *this* repo (Tim's own workflow).
2. It is the **teaching example** Tim demos live in class.

So when editing anything under `.github/`, optimize for **clarity of demonstration** over brevity. Comments explaining *why* a frontmatter field exists are pedagogical, not noise. The `Copilot Course Teaching Demo.agent.md` agent is the meta-example: it exercises multi-model fallback (`model: ["claude-opus-4-8", "gpt-5.6-sol"]`) and references all three skills. Note `handoffs` and `argument-hint` are **VS Code only** and are ignored by the cloud agent on github.com.

The intentional pairings in `.github/`:

- **`.github/agents/*.agent.md`** + **`AGENTS.md`** at root — both are recognized. GitHub honors the **nearest `AGENTS.md`** in the directory tree, plus `CLAUDE.md` and `GEMINI.md` at root.
- **`agents/*.agent.md`** + **`.github/chatmodes/new-mode.chatmode.md`** — current vs deprecated format; teaching the rename.
- **`prompts/*.prompt.md`** — all use `agent:`. The `mode:` field is **gone from the docs entirely**, not merely deprecated. `agent:` accepts `ask`, `agent`, `plan`, or a custom agent name. Variable syntax is `${input:varName}`.
- **`skills/[name]/SKILL.md`** — required frontmatter is `name` (max 64 chars, lowercase-hyphenated, must match the parent directory) + `description` (1-1024 chars). Optional: `license`, `compatibility`, `metadata`, `allowed-tools` (experimental). The same SKILL.md works in Copilot, Claude Code, Cursor, and Codex CLI. **Do not label the feature GA or preview** — GitHub's docs assign it no maturity tier.
- **Tool scoping as security** — `Code Review and Security Expert.agent.md` deliberately omits `editFiles`; this is the live demo for "tool scope = trust boundary."

`docs/latest-github-news.md` is **maintained by hand**. The former `copilot-news-fetcher.yml` workflow depended on a third-party DeepSeek API key, last ran in March 2026, and was removed in July 2026. Refresh that file from primary sources when preparing a delivery.

## Demo App (`/src`) — Architectural Notes

Node 18+ Express app serving a browsable catalog of curated Copilot tips. It replaced the
Python FastMCP server; there is no Python, no `uv`, and no pytest suite in `/src` anymore.

- **`server.js`** — small Express server. `GET /api/tips` returns the full dataset with
  metadata and categories; `GET /api/random-tip?exclude=<id>` returns one random tip,
  optionally excluding an id. Serves `public/` statically. `PORT` env var overrides port 3000.
- **`public/index.html`** — single-page front end for the tips browser.
- **`data/copilot_tips.json`** — source data. Treat as data, not config.
- **`data/copilot-metrics-sample.json`** — static sample of a Copilot Metrics API response
  for offline demos. Note the legacy metrics APIs closed down January 29, 2026; verify this
  sample against the current schema before demoing it as live-shaped.
- **`tip-lookup.js`** — **contains an intentional bug on line 35** (`if (tip.id = targetId)`
  instead of `===`). Do not "fix" this file. It is the live target for the cloud agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timothywarner/copilot-dev](https://github.com/timothywarner/copilot-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
