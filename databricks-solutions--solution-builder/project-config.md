---
trigger: always_on
description: > **This file is your memory across sessions. Keep it accurate — updating it is part of the work, not an afterthought.**
---

# CLAUDE.md — Industry Demo Prompt Generator

> **This file is your memory across sessions. Keep it accurate — updating it is part of the work, not an afterthought.**
> When a change lands that alters how the system is built, packaged, run, or deployed — a new top-level directory, a stack swap, a workflow shift, a renamed/moved key file, a new dev/build/deploy script, or a new *mode* of a subsystem (e.g. how a skill is packaged, how an artifact renders) — **update this file in the same change, before you consider the task done.** If you touched something described here and the description is now wrong, fix it. Better stale-in-one-place than spread-across-three-places. See "How to keep this file accurate" at the bottom for the exact triggers.

## What this project actually is

A **system that generates Databricks demos**. Not one app — **three**, plus a skill:

1. **Generator app** (`app/`) — the user-facing tool. FastAPI + React. A user opens it, picks an industry/capability, chats with a Claude Code agent (via `claude-agent-sdk`), and the agent assembles a personalized demo by reading **blocks** from the skill below. Generated artifacts (demo code, specs, app boilerplate, deployable Databricks assets) land in a per-project directory. The home page has **three entry modes** (a `mode` field on each project): *story* (describe it → agent builds every Databricks resource), *architecture* (draw the diagram first → generate from it), and *genie-code-workshop* (agent writes Genie-Code prompts → you build the resources live in notebooks). The mode picks the agent's Build fork; capability blocks carry a `genie_code_workshop` flag so the workshop tab hides what can't be built via Genie Code (apps, Lakebase, KA, MAS).

2. **Solution-builder skill** (`.claude/skills/databricks-solution-builder/`) — the agent's brain. SKILL.md + reference blocks + stage guides + the template app. The generator's agent reads from here. **The skill is the product; the generator is the UI.** (Formerly named `databricks-demo-generator` — renamed to `databricks-solution-builder`.)

3. **Template app** (`.claude/skills/databricks-solution-builder/app/app_template/`) — a complete Node.js + Express + React Databricks App that ships *as part of every generated demo*. The agent copies + customizes it. It is **NOT** a sub-component of the generator — it's an artifact the generator emits.

4. **Test copies** (`app/test/{app_template_test,app_template_test_simple,luxebeauty_workshop}/`) — runnable, live-workspace-wired copies of the skill's template app + reference demos, used to dogfood + iterate. **The workflow is: debug in the test copy, then sync the working content back into the skill.** They must stay in lockstep. See "Test apps ↔ skill" below.

Plus: **Databricks Agent Skills (DAS)** — cloned into `app/ai_dev_kit/` (dir name kept for path stability) from `github.com/databricks/databricks-agent-skills`, holding ~28 per-resource skills for creating individual Databricks resources (pipelines, dashboards, Genie spaces, KAs, MAS, etc.). Skills live under `skills/*` + `experimental/databricks-genie`; the generator's agent uses these during the Build stage. (Migrated from the retired `databricks-solutions/ai-dev-kit`.)

## Mental model

```
USER opens generator app (app/)
   ↓ chats with agent
AGENT reads .claude/skills/databricks-solution-builder/
   - SKILL.md → 4-stage workflow (Intent → Design → Spec → Build)
   - references/blocks/{domains,capabilities,patterns}/*.md
   - stages/0X-*.md
   ↓
AGENT writes specs + scaffolds a Databricks App by copying app_template/
AGENT delegates resource creation to subagents using Databricks Agent Skills (DAS)
   ↓
ARTIFACTS land in:
   - per-project directory (specs, code)
   - the user's Databricks workspace (catalog, schema, pipeline, dashboard, app)
```

## Repository layout

```
industry-demo-prompts/
├── app/                                  # ★ Generator app (FastAPI + React)
│   ├── src/demo_prompt_generator/
│   │   ├── backend/                      # FastAPI
│   │   │   ├── app.py, router.py, models.py
│   │   │   ├── core/                     # app factory, config, DI, DB
│   │   │   ├── routes/                   # /api/* (agent, projects, messages, preview, …)
│   │   │   ├── services/
│   │   │   │   ├── agent.py              # ★ Wraps claude-agent-sdk + SSE streaming
│   │   │   │   └── skills_manager.py     # Copies skills into per-project .claude/
│   │   │   └── preview/                  # Subprocess runner that spawns generated app's start.sh
│   │   └── ui/                           # React 19 + TanStack Router + Tailwind v4
│   │       ├── routes/                   # project.$projectId.tsx is the workspace
│   │       ├── preview/                  # Preview iframe + log streaming UI
│   │       ├── lib/platform-architecture.ts  # ★ Arch-diagram CATALOG + flat-file parse/serialize + computeLayout (see below)
│   │       └── components/project/platform-diagram{.tsx,/}  # ★ ReactFlow arch-diagram editor (see below)
│   ├── test/                             # ★ 3 runnable test copies — debug live, then sync to the skill (see below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databricks-solutions/solution-builder](https://github.com/databricks-solutions/solution-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
