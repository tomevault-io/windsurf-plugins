---
trigger: always_on
description: <!-- spec_version: 1.4.2 -->
---

<!-- spec_version: 1.4.2 -->

# AI Context Kit Agent Guide

## Purpose
AI Context Kit is a cross-provider instruction-layer repository for context-aware AI collaboration.

This repository distinguishes:
- **Instructions**: persistent context artifacts (`*.instructions.md` for user context and `AGENTS.md` for project context) that define who the user is, what the project is, and how collaboration should run.
- **Prompts/queries**: day-to-day requests made inside that instructed environment.

## Source Of Truth And Precedence
Use this order when files differ:
1. **Specification (authoritative, v1.4.2):** `specs/context_aware_ai_session_spec.md`
2. **Templates (canonical structures):** `templates/*.instructions.md` and `templates/skill_template/SKILL.md`
3. **Skills (canonical operational workflows):** `skills/*/SKILL.md` and skill-local references
4. **Prompts:** `prompts/skills/*.prompt.md` (compatibility wrappers — must defer detailed logic to skills); `prompts/loop/*.prompt.md` (implementation loop steps — self-contained workflow prompts)
5. **Samples and validation artifacts (illustrative records):** `usercontexts/*.instructions.md`, related `*.validation.md`

## Repository Map
| Path | Purpose |
| --- | --- |
| `specs/` | Normative session-model specification and terminology |
| `templates/` | Canonical instruction templates aligned to the spec |
| `skills/` | Canonical workflow skills (`SKILL.md` folders) and skill-local resources |
| `prompts/skills/` | Compatibility wrappers that route workflows to canonical skills |
| `prompts/loop/` | Numbered step prompts for the implementation loop (readiness-check → implementation → self-review → learnings → human-in-the-loop); invoke in order, learnings is optional |
| `usercontexts/` | User-context instruction examples and validation reports |

## Scope And Precedence For AGENTS.md Files
- An `AGENTS.md` file applies to the directory it is in and all subdirectories.
- If multiple `AGENTS.md` files apply, the closest (deepest) one wins for files in its subtree.
- Keep root `AGENTS.md` global and nested `AGENTS.md` files folder-specific.
- If instructions conflict or remain unclear after precedence, ask before proceeding.

## Session-State Contract
### Session State Summary
Active session state includes:
- Project
- Role/Mode
- Phase
- Output Style
- Tone
- Interaction Mode

### Persistence And Transitions
- Session state persists across turns until explicitly changed or reset.
- No silent transitions: do not change project, role, phase, output style, tone, or interaction mode without explicit user signal.
- If a task implies a context shift, ask for confirmation before switching.

### Cross-Session Persistence (spec section 4.4)
- When the user signals session end or explicitly requests one, propose creating a checkpoint artifact using the `create-checkpoint` skill.
- Never create a checkpoint silently. User approval is required before writing.
- When a checkpoint artifact is provided at session start, apply the `restore-checkpoint` skill to restore state and surface any conflicts with active instruction files before proceeding.

### Context Compression (spec section 4.5)
- When context window saturation is evident, propose compression explicitly — describe what will be retained and what will be dropped before asking for confirmation.
- Never apply compression silently. The user must confirm before any context is dropped.
- Before applying compression, offer to export the current state to a checkpoint file using the `create-checkpoint` skill.
- After compression, do not imply that dropped context is recoverable.

### Ambiguity Rule
- If assumptions, state, or intent are ambiguous, ask clarifying questions before acting.

### Default State For This Repo
Defaults are defined directly in this `AGENTS.md` to keep project context self-contained and AGENTS-first.

| Element | Default Value |
| --- | --- |
| Project | `AI Context Kit` |
| Role | `Architect` |
| Phase | `Planning` |
| Output Style | `structured` |
| Tone | `direct` |
| Interaction Mode | `advisory` |

## Command Namespace Policy
Use namespaced commands for explicit state control.

| Command | Description |
| --- | --- |
| `/ack.context` | Show active session state summary |
| `/ack.mode <role>` | Change assistant role/mode |
| `/ack.phase <phase>` | Change current work phase |
| `/ack.style <style>` | Change output style/verbosity |
| `/ack.tone <tone>` | Change communication tone |
| `/ack.interact <mode>` | Change interaction mode (`advisory`, `pair`, `driver`) |
| `/ack.reset` | Reset session state (keep user context and project context unless user says otherwise) |

Alias policy:
- Namespaced `/ack.*` commands are the default.
- Unprefixed aliases are allowed only when no command conflict exists.

## Repository Project Context
### Overview
**AI Context Kit** is a template repository for building instruction-based AI collaboration across providers.

- Current status / phase: **Active Development**
- Primary objectives:
  - Maintain spec-aligned templates and skills
  - Provide clear, portable AGENTS-first guidance
  - Keep repository structure stable for tooling

### Role Definitions
| Role | When To Use | Assistant Behavior | Typical Outputs |
| --- | --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MSiccDev/ai-context-kit](https://github.com/MSiccDev/ai-context-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
