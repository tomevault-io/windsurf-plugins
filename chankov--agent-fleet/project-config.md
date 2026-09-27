---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository. pi is the coding agent Agent Fleet installs for; Claude Code takes part only as a coms peer (see `docs/claude-code-coms-bridge.md`).
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository. pi is the coding agent Agent Fleet installs for; Claude Code takes part only as a coms peer (see `docs/claude-code-coms-bridge.md`).

## Terminal File Links

- In terminal responses, link local files using descriptive Markdown link text and an absolute `file:///` URL, e.g. `[Agent guide](file:///absolute/path/to/AGENTS.md)`. Do not use relative Markdown link targets or show bare paths instead of named links. This avoids Zed/macOS opening errors.

## Repository Overview

Agent Fleet — a Pi-centered multi-agent orchestration system (agent-hub dispatcher, herdr fleet control plane, coms peer messaging, Hermes remote control) plus a library of lifecycle skills and agent personas for pi. Skills live in two roots: fleet-native `skills/` and the vendored upstream import `vendor/agent-skills-upstream/skills/` (native wins on name collisions — see `docs/UPSTREAM-SKILLS.md`).

## Project Philosophy

Read [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md) before proposing or revising Agent
Fleet plans, architecture, or process changes. It is the canonical project
direction: deterministic lifecycle control, proportional workflows, adaptive
model assistance, and evidence-based completion. Plans and design reviews must
include a proportionate alignment note and distinguish current guarantees from
proposed behavior. Surface conflicts for a maintainer decision rather than
silently changing these principles.

## Skill-Driven Execution

Agent Fleet runs on a **skill-driven execution model** powered by this repository's `/skills` directory.

### Core Rules

- If a task matches a skill, you MUST invoke it
- Skills are located in `skills/<skill-name>/SKILL.md`
- Never implement directly if a skill applies
- Always follow the skill instructions exactly (do not partially apply them)

### Intent → Skill Mapping

The agent should automatically map user intent to skills:

- Feature / new functionality → `spec-driven-development`, then `incremental-implementation`, `test-driven-development`
- Planning / breakdown → `planning-and-task-breakdown`
- Bug / failure / unexpected behavior → `debugging-and-error-recovery`
- Code review → `code-review-and-quality`
- Refactoring / simplification → `code-simplification`
- API or interface design → `api-and-interface-design`
- UI work → `frontend-ui-engineering`

### Lifecycle Mapping

The agent should internally follow this lifecycle and invoke the matching skill at each phase:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery`
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

This lifecycle is normally implicit — the agent maps user intent to the right skill without being asked.

The repo also ships slash commands as explicit lifecycle entry points, `af-`-prefixed
in `.pi/prompts/`:

- `/af-spec` → `spec-driven-development`
- `/af-plan` → `planning-and-task-breakdown`
- `/af-build` → `incremental-implementation` + `test-driven-development`
- `/af-test` → `test-driven-development`
- `/af-review` → `code-review-and-quality`
- `/af-code-simplify` → `code-simplification`
- `/af-ship` → `shipping-and-launch`

Workspace installation is outside the prompt/skill layer: use the deterministic
`agent-fleet setup`, `agent-fleet doctor`, and `agent-fleet uninstall` CLI commands.

Whether triggered implicitly or via a command, the agent MUST invoke the underlying skill — never inline the steps.

### Execution Model

For every request:

1. Determine if any skill applies (even 1% chance)
2. Invoke the appropriate skill
3. Follow the skill workflow strictly
4. Only proceed to implementation after required steps (spec, plan, etc.) are complete

### Anti-Rationalization

The following thoughts are incorrect and must be ignored:

- "This is too small for a skill"
- "I can just quickly implement this"
- "I’ll gather context first"

Correct behavior:

- Always check for and use skills first

This keeps workflow enforcement identical across every supported runtime.

## Orchestration: Personas, Skills, and Commands

This repo has three composable layers. They have different jobs and should not be confused:

- **Skills** (`skills/<name>/SKILL.md`) — workflows with steps and exit criteria. The *how*. Mandatory hops when an intent matches.
- **Personas** (`agents/<role>.md`) — roles with a perspective and an output format. The *who*.
- **Slash commands** (`.pi/prompts/af-*.md`) — user-facing entry points. The *when*. The orchestration layer.

Composition rule: **the user (or a slash command) is the orchestrator. Personas do not invoke other personas.** A persona may invoke skills.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chankov/agent-fleet](https://github.com/chankov/agent-fleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
