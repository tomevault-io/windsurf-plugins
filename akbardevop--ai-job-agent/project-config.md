---
trigger: always_on
description: This is a job application automation toolkit. It automates filling and submitting applications on LinkedIn Easy Apply, Greenhouse, Lever, Jobvite, and Ashby, plus email triage and Google Sheets tracking.
---

# AI Job Application Agent - Claude Code Instructions

## Overview

This is a job application automation toolkit. It automates filling and submitting applications on LinkedIn Easy Apply, Greenhouse, Lever, Jobvite, and Ashby, plus email triage and Google Sheets tracking.

Two ways to drive it:

1. **Bundled skills (recommended)** — type `/job-coach`, `/job-apply`, `/job-evaluate`, `/job-cv`, `/job-track`, `/job-triage`, `/job-status`, `/job-outreach`, `/job-followup`, `/job-dashboard`, `/job-interview`, `/job-patterns`, or `/job-setup` in any Claude Code session. Each skill wraps the scripts below and renders results as a markdown table. Install once with `bash skills/install.sh`.
2. **Raw scripts** — call the Node.js / Python scripts in `scripts/` directly from your terminal (documented below).

## Bundled Skills

Installed by `bash skills/install.sh` (symlinks them into `~/.claude/skills/`). Each skill's prompt lives at `skills/<name>/SKILL.md` — read it to see exactly what the agent does.

| Skill | Wraps | Output |
|-------|-------|--------|
| `/job-coach` | the persona — intake, research, slate, dispatch | target-role slate + next-move suggestions; persists to `config/search-plan.md` |
| `/job-setup` | `wizard.sh` equivalent — but in chat | conversational onboarding, auto-reads user files, writes configs, registers skills |
| `/job-apply <url> [--submit]` | 5 ATS fillers (auto-routed by URL host) | result table: platform, outcome, exit code |
| `/job-track [sync]` | local CSV + `google-sheet-sync.py` | counts-by-status table + recent activity |
| `/job-triage [query]` | `outlook-triage.js` | classified-email counts + preview table, step-through extract/mark-read |
| `/job-status <updates.json>` | `tracker-status-update.py` | before/after diff, confirmation prompt, result summary |
| `/job-outreach <target>` | `send-cold-email.js` + Claude (the agent drafts) | research → draft preview → dry-run → send → log to `outreach-log.csv` |
| `/job-followup [send]` | `outreach-log.csv` + `send-cold-email.js` | urgency table (overdue/due/soon/waiting) + optional step-through send |
| `/job-dashboard [live]` | `job-dashboard.mjs` (zero deps) | ANSI tables: applications + outreach + follow-ups. Snapshot in chat or `npm run dashboard` for live TUI |
| `/job-evaluate <url>` | WebFetch + 7-block A-G rubric + `/job-cv` chain + tracker append | structured eval report at `reports/<date>-<company>-<role>.md` + tailored PDF + tracker row |
| `/job-cv <jd>` | `scripts/generate-tailored-cv.mjs` (Playwright Chromium → PDF) | Claude rewrites base CV (no invention) → ATS-friendly PDF at `output/cv-<company>-<date>.pdf` |
| `/job-interview <company>` | `application-tracker.csv` + WebSearch + WebFetch | company snapshot + 10-15 likely Qs + 5-8 STAR stories from real projects + 5 smart asks + 3 red flags |
| `/job-patterns` | `application-tracker.csv` + `outreach-log.csv` | rejection diagnostics (by ATS / time-to-rej / geography / role-type / day-of-week) + 3 actionable fixes |

**First-run flow (gstack-style):** open Claude Code anywhere and paste this one message:

> Install ai-job-agent: run `git clone --single-branch --depth 1 https://github.com/AkbarDevop/ai-job-agent ~/.claude/skills/ai-job-agent && cd ~/.claude/skills/ai-job-agent && bash skills/install.sh` then add an "ai-job-agent" section to my CLAUDE.md telling you to treat me like a career-coaching client — open-ended job-search talk → `/job-coach` (intake + research + slate), job URL alone → `/job-apply`, "evaluate this URL" → `/job-evaluate` (rubric + report + tailored CV), "tailor my CV" → `/job-cv`, "prep me for X interview" → `/job-interview`, "what's working / why am I getting rejected" → `/job-patterns`, reaching out to a person → `/job-outreach`, follow-ups → `/job-followup`, "how am I doing" → `/job-dashboard`, tracker → `/job-track`, status updates → `/job-status`, Outlook → `/job-triage`, first-time setup → `/job-setup`. Never answer job-search questions conversationally — always dispatch. Tell me when done, then run /job-setup and chain into /job-coach intake.

Claude clones into `~/.claude/skills/ai-job-agent/`, registers all 13 skills as symlinks under `~/.claude/skills/`, writes the coach-first routing block into CLAUDE.md, runs `/job-setup` (identity + resume + optional msmtp), and chains into `/job-coach intake` (target roles + companies + timeline → live market research → ranked slate of next moves). **After that, the user talks naturally** — "I want to apply to this url", "email the VP at X", "evaluate this posting", "tailor my CV for this JD", "prep me for my interview at Y", "what's working" → diagnostics, "how am I doing" — and Claude dispatches to the right skill. No `cd` required ever; skills work from any directory in any session. `bash wizard.sh` and `bash setup.sh` still exist for non-Claude-Code users.

Skills find this repo via `$AI_JOB_AGENT_ROOT` → `~/.claude/skills/ai-job-agent/REPO_PATH` → `~/ai-job-agent`. Set the env var or rerun `install.sh` from a non-default clone location.

## Unified mode — `npm run agent`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AkbarDevop/ai-job-agent](https://github.com/AkbarDevop/ai-job-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
