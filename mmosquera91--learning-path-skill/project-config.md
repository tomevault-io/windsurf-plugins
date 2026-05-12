---
trigger: always_on
description: > This document is for **developers extending or debugging** the skill. For user-facing usage, see [README.md](README.md).
---

# AGENTS.md — Builder's Guide to the Learning Path Skill

> This document is for **developers extending or debugging** the skill. For user-facing usage, see [README.md](README.md).

---

## 1. Project Overview

Learning Path Generator is a Hermes Agent skill that acts as a personal AI tutor. It generates structured learning paths for any topic, delivers daily tasks via Telegram (cron-driven), evaluates submissions with a two-axis rubric, and adapts the plan based on performance.

The entire state machine lives in a single SQLite file. Cron jobs run with zero prior context — every prompt is self-contained with all SQL and decision logic inline. The skill is split into a router (SKILL.md) and four subskills loaded on demand to keep context lean for local models.

**Stack:** Hermes Agent + SQLite (WAL mode) + Telegram delivery + cron tool

> **v1.1 rename note (2026-04-06):** The skill was renamed from `learning-path` to `tutor`. This aligns with Hermes v0.7 where `/tutor` is the only slash command that auto-registers (derived from the skill name). Sub-commands like `submit`, `confirm`, `edit` are now `/tutor submit`, `/tutor confirm`, `/tutor edit` — consistent with the `/tutor <subcommand>` pattern. All paths, cron prompts, and internal references updated. DB data is preserved.

---

## 2. Architecture Decisions

These are the non-obvious choices that shaped the skill. Understanding them prevents re-litigating solved problems.

| Decision | Choice | Why |
|----------|--------|-----|
| Persona embedded in SKILL.md | No separate Hermes profile | Cron jobs run on the default profile — Hermes has no `--profile` flag for cron. A separate profile would be cleaner but the cron layer doesn't support it. |
| Subskill router pattern | 4 subskills loaded on demand | A monolithic SKILL.md would be 500+ lines. Local models (Ollama) struggle with long prompts. The router keeps context ~180 lines; subskills load only when needed. |
| SQLite over JSON files | WAL mode, foreign keys | Concurrent access from cron + agent sessions. JSON would need manual locking and can't query. SQLite handles atomic writes natively. |
| Explicit `/submit` command | 20h window as fallback only | Without `/submit`, a casual question like "what is a lifetime in Rust?" gets evaluated as a task submission. The 20h window asks for confirmation before evaluating. |
| LLM outputs structured JSON for evals | JSON schema in prompt | Free-form text leads to inconsistent scoring. JSON forces the model to commit to numbers and specific feedback. Easier to parse, store, and compare across sessions. |
| Cron prompts include ALL SQL inline | ~500 lines per prompt | Cron sessions start with zero context — no conversation history, no loaded skills. Every query and decision branch must be in the prompt. |
| `init_db.py` is idempotent | Run on every cron invocation | Cron has no guarantee the DB exists. Running init every time is safe (it checks `schema_version` and exits early if already initialized). |

---

## 3. File Structure with Responsibilities

```
~/.hermes/skills/tutor/
├── SKILL.md                  # Router — persona, rules, command dispatch, pitfalls
├── AGENTS.md                 # This file — builder's guide
├── README.md                 # User-facing documentation
├── LICENSE                   # MIT
├── .gitignore                # Ignores learning.db, __pycache__
│
├── subskills/
│   ├── init.md               # /tutor init flow: syllabus gen → URL validation → confirm → save
│   ├── daily.md              # Cron daily task: path check → inactivity → review check → task gen → deliver
│   ├── eval.md               # /submit flow: pending task → evaluate → score → decision → update state
│   └── adapt.md              # Weekly review + /tutor review: metrics → adaptation rules → report
│
├── templates/
│   ├── syllabus.md           # Syllabus presentation format (sent to user for review)
│   ├── daily-task.md         # Daily task Telegram format
│   ├── evaluation.md         # Full rubric + JSON output schema for evaluations
│   ├── weekly-report.md      # Weekly report template
│   └── milestone.md          # Module completion celebration message
│
├── scripts/
│   ├── init_db.py            # Idempotent DB initialization. Creates all tables + default config.
│   └── migrate_db.py         # Schema migration engine. Up/down migration with backup support.
│
└── learning.db               # Runtime state — created by init_db.py, ignored by git
```

### What Each File Does NOT Do

| File | Does NOT |
|------|----------|
| `SKILL.md` | Does NOT contain task generation logic, evaluation logic, or SQL for daily operations — it delegates to subskills |
| `init.md` | Does NOT handle evaluation or daily task delivery — only syllabus creation and path activation |
| `daily.md` | Does NOT evaluate submissions — it only generates and delivers tasks |
| `eval.md` | Does NOT generate tasks — it only evaluates submitted responses |
| `adapt.md` | Does NOT evaluate or generate tasks — it only reviews weekly metrics and adjusts the path |
| `init_db.py` | Does NOT migrate — use `migrate_db.py` for schema changes |
| `migrate_db.py` | Does NOT create a fresh DB — use `init_db.py` first |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmosquera91/learning_path_skill](https://github.com/mmosquera91/learning_path_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
