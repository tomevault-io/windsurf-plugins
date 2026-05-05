---
trigger: always_on
description: - ADOPT: project exists, docs are missing/weak; scan the project and build docs + alignment phases
---

# AGENTS.md

<!--
AGENTS_STATE values:
- ADOPT: project exists, docs are missing/weak; scan the project and build docs + alignment phases
- BOOTSTRAP: conversation.md exists; decompress into .unpack/docs/
- BUILD: docs + phases exist; execute phases
-->
<!-- AGENTS_STATE: BOOTSTRAP -->
<!-- UNPACK_VERSION: 1.0.0 -->

## Purpose

This repository is operated through documentation and phases.

There are two entry modes:
- **ADOPT** (existing project, no docs yet): scan the project, reverse-engineer docs, and create alignment phases.
- **BOOTSTRAP** (new project or from reference, `conversation.md` exists): read the conversation, decompress it into `.unpack/docs/`, and generate phases. If reference code was specified during init, scan it to ground specs in existing patterns.

After bootstrap or adopt, the agent switches to **BUILD** mode and uses `.unpack/docs/phases/*` to drive all work.

## Non-negotiable rules

1. **If unsure, ask.** Do not guess requirements or business intent.
2. **No hidden scope.** Any material change in scope/constraints becomes a **new steering phase**.
3. **Phases drive work.** Only do work that is in the current phase scope.
4. **Dependencies matter.** A phase cannot start until all `depends_on` phases are `done`.
5. **Keep `.unpack/docs/index.md` current.** It must index all docs + phases + statuses at all times.
6. **Inference labeling.** Anything derived from code without confirmation must be labeled as "inferred".
7. **Project memory is binding.** The agent must read `.unpack/docs/_meta/project-memory.md` (if it exists) before planning or executing a phase.
8. **Decision logging is mandatory.** Any new decision or changed assumption must be recorded (append-only) in `.unpack/docs/_meta/project-memory.md`, and if stable, promoted to an ADR.

---

## Mode detection

- If `.unpack/docs/_meta/guide-config.md` does not exist → **project not initialized**. Do not enter any mode. Do not list the repo contents or full workflow. Just welcome the user in 2-3 sentences: Unpack turns AI research conversations into structured, buildable projects — you chat freely, drop the conversation here, and the agent turns it into specs, phases, and code. Then offer to run `/up-init` to get started.
- If `conversation.md` exists at repo root → run **BOOTSTRAP**.
- Else if `.unpack/docs/index.md` is missing OR `.unpack/docs/phases/` is empty → run **ADOPT**.
- Else → run **BUILD**.

## Skill deployment

Skills are stored in `.unpack/skills/` as the canonical, agent-agnostic source. Each agent discovers skills from its own path:

- **Claude Code**: `.claude/skills/<name>/SKILL.md`
- **Codex**: `.agents/skills/<name>/SKILL.md`

Before entering any mode, check if skills are deployed to your path. If not, copy each `.unpack/skills/<name>/SKILL.md` to the corresponding location under your skills directory. This is a one-time operation — after deployment, skills are available as slash commands.

During `/up-init`, the user is asked which agent(s) they'll use, and skills are deployed to all selected paths.

---

## ADOPT — Adopt an existing project

### Goal

Create an Unpack-ready documentation system and an **alignment phase plan** that upgrades the project to a healthy, compliant state.

### Output required

- `.unpack/docs/` scaffold with:
  - `.unpack/docs/index.md` fully populated
  - `.unpack/docs/discovery/*` filled from repo scan
  - `.unpack/docs/specs/*` seeded (with "TBD / needs confirmation" where required)
  - `.unpack/docs/phases/*` alignment phases with dependencies, criteria, and test plans
- `.unpack/docs/_meta/project-memory.md` initialized (if decisions are made during bootstrap)
- Switch `AGENTS_STATE` to `BUILD`

### Steps

1. **Create docs scaffold**
   - Ensure `.unpack/docs/_meta`, `.unpack/docs/discovery`, `.unpack/docs/specs`, `.unpack/docs/phases`, `.unpack/docs/decisions` exist.
   - Create templates if missing.

2. **Repo scan (no refactors yet)**
   - Build a repo inventory from files:
     - languages/frameworks detected
     - entrypoints (main binaries / server start points)
     - dependency manifests (`package.json`, `pyproject.toml`, `go.mod`, etc.)
     - test/lint/typecheck commands (or absence)
     - CI configs (or absence)
     - deployment manifests (or absence)
   - Write results into:
     - `.unpack/docs/discovery/repo-inventory.md`
     - `.unpack/docs/discovery/runtime-and-commands.md`

3. **Inferred architecture + debt**
   - Derive a best-effort component map from folder structure + key files.
   - Label all unverified conclusions as **inferred**.
   - Write into:
     - `.unpack/docs/discovery/architecture-inferred.md`
     - `.unpack/docs/discovery/risks-and-debt.md`

4. **Seed stable specs**
   - Populate `.unpack/docs/specs/*` with what is confidently knowable.
   - Anything unclear becomes:
     - a "Needs confirmation" section inside the relevant spec, and
     - an "Open question" in the next phase file.

5. **Create alignment phases**
   - Generate `.unpack/docs/phases/phase-0.md` as the setup phase and mark it `done` only after docs are generated and indexed.
   - Create a minimal default alignment plan (phase-1+), then tailor it to what the repo scan found.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apresmoi/unpack](https://github.com/apresmoi/unpack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
