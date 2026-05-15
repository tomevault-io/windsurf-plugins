---
trigger: always_on
description: OpenDataWorks is a unified data portal for metadata management, workflow orchestration, lineage analysis, and intelligent query.
---

# AGENTS.md

## Purpose

OpenDataWorks is a unified data portal for metadata management, workflow orchestration, lineage analysis, and intelligent query.

When working in this repository, optimize for:

- preserving clear boundaries between the main Java backend, the Vue frontend, and the Python-based DataAgent intelligent-query module
- keeping workflow, metadata, and intelligent-query behavior aligned with the current deployment model under `deploy/`
- preferring targeted, low-risk changes over broad refactors or speculative abstractions

## Tech Stack & Runtime

- `backend/`: main business backend for metadata, workflow, lineage, and platform APIs
- `frontend/`: main web application
- `dataagent/dataagent-backend/`: FastAPI-based intelligent-query backend
- `dataagent/.claude/skills/dataagent-nl2sql/`: intelligent-query skill bundle
- `deploy/`: Docker Compose, environment templates, and image/build assets

### Frontend stack

- `Vue 3` + `Vite 5`
- `Vue Router 4`
- `Pinia`
- `Element Plus` as the primary existing UI component library
- `Sass` for current style organization
- `Tailwind CSS` is an approved additive styling layer for new frontend work when utility-first styling improves implementation speed or consistency
- `ECharts`, `Vue Flow`, `CodeMirror`, `Axios`

### Backend stack

- Main backend:
  - `Java 8`
  - `Spring Boot 2.7`
  - `Spring MVC` + `WebFlux`
  - `MyBatis-Plus`
  - `MySQL 8`
  - `Flyway`
  - `Lombok`
  - `Hutool`
  - `JSqlParser`
  - `Apache POI`
- Intelligent-query backend:
  - `Python`
  - `FastAPI`
  - `Pydantic`
  - `PyMySQL`
  - `Alembic`
  - `AnyIO`

### Frontend stack rules

- Do not rewrite established `Element Plus` or `Sass` surfaces just to force Tailwind adoption.
- When introducing Tailwind CSS, do it incrementally and only for the touched UI area.
- Keep frontend changes aligned with the existing Vue component structure and routing/state patterns already in the repo.

### Node / nvm baseline

- This repository uses `nvm`.
- Before running any frontend command (`vite`, `npm --prefix frontend ...`, build/dev), load and switch Node from `.nvmrc`:
  - `export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" && nvm use`
- If the target Node version is missing, install it first:
  - `nvm install`

### Frontend execution rule

- Always run `nvm use` successfully before frontend build/test/dev commands.
- If a frontend command fails with Node engine/runtime errors, retry only after `nvm use`.

### Python baseline

- Do not assume the first `python3` on `PATH` is the correct interpreter for `dataagent/dataagent-backend`.
- In this workspace, prefer a dedicated local virtualenv when it already exists for DataAgent work.
- For this repository, default to `dataagent/dataagent-backend/.venv-py313` when it exists.
- `claude-agent-sdk` requires Python `>=3.10`. Do not try to install or run the DataAgent execution path with the Xcode Python `3.9.6`.
- If no project virtualenv is present, prefer the Xcode Python at `/Applications/Xcode.app/Contents/Developer/usr/bin/python3` for local DataAgent commands, because the host `/usr/local/bin/python3` may point to a different interpreter without the required packages.
- Before running backend commands, verify the interpreter can import the required runtime modules for the touched path, at minimum:
  - `fastapi`
  - `uvicorn`
  - `alembic`
  - `pymysql`
  - `anyio`
- For any execution path that reaches the real agent runtime, also verify:
  - `claude_agent_sdk`
- If the preferred interpreter cannot import the required modules, install the missing packages into the chosen environment before claiming the backend cannot be started.
- If the runtime reports `claude-agent-sdk 未安装`, first verify the active interpreter and Python version before concluding that the package is actually missing. In this repo, that error usually means the wrong interpreter was used.

## Architecture Overview

- `backend/src`: core platform logic for data assets, workflow orchestration, and lineage
- `frontend/src`: main UI for workflows, lineage, Data Studio, and intelligent-query entrypoints
- `dataagent/dataagent-backend`: NL2SQL runtime, session persistence, prompt assembly, and skill integration
- `docs/design`: active technical design documents for medium and large changes
- `docs/plans`: execution plans paired with design documents
- `docs/handbook`: long-lived handbook and feature documentation
- `docs/reports`: reports, validation notes, and post-change summaries

## Working Rules

- Prefer one verified primary path by default.
- Only add compatibility branches when real version or environment differences are confirmed.
- Keep fallback minimal, explicit, and single-layer.
- Avoid repeated or cascading fallback chains and duplicate guard branches.
- Prefer small, targeted changes that preserve existing module boundaries.
- When changing a public API, schema, runtime contract, or deployment behavior, update the related docs in the same change when the impact is medium or large.
- Do not move skill-specific behavior into shared runtime modules unless the behavior is genuinely generic.

## Design & Plan Workflow

- Change sizing:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendata-lab/opendataworks](https://github.com/opendata-lab/opendataworks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
