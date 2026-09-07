---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

<!-- BEGIN GSTACK-CODEX MANAGED BLOCK -->
## gstack — AI Engineering Workflow

This block is managed by `gstack-codex`. Do not edit inside this block.

Skills live in `.agents/skills`. Invoke them by name, e.g. `/office-hours`.
Refresh with `npx gstack-codex init --project`.
This repo currently has the `full` pack installed.

## Available skills

| Skill | What it does |
|-------|-------------|
| `/office-hours` | YC Office Hours — two modes. Startup mode: six forcing questions that expose demand reality, status quo, desperate specificity, narrowest wedge, observation, and future-fit. |
| `/plan-ceo-review` | CEO/founder-mode plan review. Rethink the problem, find the 10-star product, challenge premises, expand scope when it creates a better product. |
| `/plan-eng-review` | Eng manager-mode plan review. Lock in the execution plan — architecture, data flow, diagrams, edge cases, test coverage, performance. |
| `/plan-design-review` | Designer's eye plan review — interactive, like CEO and Eng review. |
| `/design-consultation` | Design consultation: understands your product, researches the landscape, proposes a complete design system (aesthetic, typography, color, layout, spacing, motion), and generates font+color preview pages. |
| `/review` | Pre-landing PR review. Analyzes diff against the base branch for SQL safety, LLM trust boundary violations, conditional side effects, and other structural issues. |
| `/investigate` | Systematic debugging with root cause investigation. Four phases: investigate, analyze, hypothesize, implement. |
| `/design-review` | Designer's eye QA: finds visual inconsistency, spacing issues, hierarchy problems, AI slop patterns, and slow interactions — then fixes them. |
| `/qa` | Systematically QA test a web application and fix bugs found. |
| `/qa-only` | Report-only QA testing. Systematically tests a web application and produces a structured report with health score, screenshots, and repro steps — but never fixes anything. |
| `/ship` | Ship workflow: detect + merge base branch, run tests, review diff, bump VERSION, update CHANGELOG, commit, push, create PR. |
| `/document-release` | Post-ship documentation update. Reads all project docs, cross-references the diff, builds a Diataxis coverage map (reference/how-to/tutorial/explanation), updates README/ARCHITECTURE/CONTRIBUTING/CLAUDE.md to match what shipped, detects architecture diagram drift, polishes CHANGELOG voice with a sell-test rubric, cleans up TODOS, and optionally bumps VERSION. |
| `/retro` | Weekly engineering retrospective. Analyzes commit history, work patterns, and code quality metrics with persistent history and trend tracking. |
| `/browse` | Fast headless browser for QA testing and site dogfooding. Navigate any URL, interact with elements, verify page state, diff before/after actions, take annotated screenshots, check responsive layouts, test forms and uploads, handle dialogs, and assert element states. |
| `/setup-browser-cookies` | Import cookies from your real Chromium browser into the headless browse session. |
| `/careful` | Safety guardrails for destructive commands. Warns before rm -rf, DROP TABLE, force-push, git reset --hard, kubectl delete, and similar destructive operations. |
| `/freeze` | Restrict file edits to a specific directory for the session. |
| `/guard` | Full safety mode: destructive command warnings + directory-scoped edits. |
| `/unfreeze` | Clear the freeze boundary set by /freeze, allowing edits to all directories again. |
| `/gstack-upgrade` | Upgrade gstack to the latest version. Detects global vs vendored install, runs the upgrade, and shows what's new. |
| `/autoplan` | Auto-review pipeline — reads the full CEO, design, eng, and DX review skills from disk and runs them sequentially with auto-decisions using 6 decision principles. |
| `/benchmark` | Performance regression detection using the browse daemon. Establishes baselines for page load times, Core Web Vitals, and resource sizes. |
| `/benchmark-models` | Cross-model benchmark for gstack skills. Runs the same prompt through Claude, GPT (via Codex CLI), and Gemini side-by-side — compares latency, tokens, cost, and optionally quality via LLM judge. |
| `/canary` | Post-deploy canary monitoring. Watches the live app for console errors, performance regressions, and page failures using the browse daemon. |
| `/claude` | Claude Code CLI wrapper for non-Claude hosts - three modes. Review: independent diff review via claude -p. |
| `/context-restore` | Restore working context saved earlier by /context-save. Loads the most recent saved state (across all branches by default) so you can pick up where you left off — even across Conductor workspace handoffs. |
| `/context-save` | Save working context. Captures git state, decisions made, and remaining work so any future session can pick up without losing a beat. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyzoo-lab/openPad](https://github.com/whyzoo-lab/openPad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
