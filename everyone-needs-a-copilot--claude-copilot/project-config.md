---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Claude Copilot framework.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the Claude Copilot framework.

---

## Main Session Guardrails

**These rules prevent context bloat — the framework's core purpose.**

| Rule | What To Do Instead | Enforcement |
|------|-------------------|-------------|
| Never write implementation code | Delegate to `@agent-me` | Hook: force-delegate |
| Never create detailed plans | Delegate to `@agent-ta` | Hook: force-delegate |
| Never use `Explore`, `Plan`, or `general-purpose` agents | Use framework agents (they integrate with Task Copilot) | Advisory |
| Avoid reading >8 files directly | Delegate to framework agent | Hook: force-delegate (triggers at 5 consecutive same-tool calls) |
| Keep responses short | Store details via `tc wp store` | Advisory |

**Mechanical enforcement:** The force-delegate rule, QA-gate rule, and session-cap advisory are enforced by hooks in `.claude/hooks/` — not just policy. Attempting >5 consecutive Bash/Read/Edit calls will be blocked automatically. After `@agent-me` completes, all main-session tools are gated until `@agent-qa` provides a pass verdict. See `.claude/hooks/README.md` for escape hatches and debug tools.

**Framework agents:** ta, me, qa, do, sd, doc, design (plus `kc` for knowledge repo setup)

---

## Overview

**Claude Copilot** solves five challenges:

| Challenge | Solution | Component |
|-----------|----------|-----------|
| Lost memory, wasted tokens | Persistent memory + semantic search | **Memory Copilot** |
| Generic AI lacks expertise | Specialized agents for complex tasks | **Agents** |
| Manual skill management | Native @include + optional MCP | **Skills** |
| Context bloat from agents | Ephemeral task/work product storage | **Task Copilot** |
| Inconsistent processes | Battle-tested workflows | **Protocol** |

### Feature Comparison

| Feature | Invocation | Persistence | Best For |
|---------|------------|-------------|----------|
| **Memory** | Auto | Cross-session | Context preservation, decisions, lessons |
| **Agents** | Protocol | Session | Expert tasks, complex work |
| **Skills** | @include or MCP | On-demand | Reusable patterns, marketplace |
| **Tasks** | CLI (`tc`) | Per-initiative | PRDs, task tracking, work products |
| **Commands** | Manual | Session | Quick shortcuts, workflows |
| **Extensions** | Auto | Permanent | Team standards, custom methodologies |

---

## Quick Decision Guide

### Command Selection Matrix

| Command | When to Use | Scope |
|---------|-------------|-------|
| `/setup` | First time on machine | Machine |
| `/setup-project` | New project initialization | Project |
| `/update-project` | Sync project with latest framework | Project |
| `/update-copilot` | Update framework itself | Machine |
| `/knowledge-copilot` | Create shared knowledge repo | Machine/Team |
| `/protocol [task]` | Start fresh work session | Session |
| `/continue [stream]` | Resume previous work | Session |
| `/pause [reason]` | Context switch, save state | Session |
| `/map` | Analyze codebase structure | Project |
| `/memory` | View memory state and recent activity | Session |
| `/orchestrate` | Set up parallel stream orchestration | Project |

### Use Case Mapping

| I want to... | Start with | What Happens |
|--------------|------------|--------------|
| Fix a bug | `/protocol fix the login bug` | Defect flow: qa → me → qa |
| Build a feature | `/protocol add dark mode UI` | Experience flow: sd → design → ta → me → qa |
| Refactor code | `/protocol refactor auth module` | Technical flow: ta → me → qa |
| Deploy / infra work | `/protocol deploy to staging` | Infra flow: do → me → qa |
| Improve something | `/protocol improve dashboard` | Clarification flow (asks intent) |
| Skip design stages | `/protocol --skip-sd add feature` | Jumps to specified stage |
| Resume yesterday's work | `/continue` | Memory loads automatically |
| Run parallel work streams | `/orchestrate generate` then `/orchestrate start` | Create PRD + tasks → set up worktrees |
| Search past decisions | `cc memory search "<query>"` | Semantic search across sessions |
| Load local skill | `@include .claude/skills/NAME/SKILL.md` | Direct file include |

---

## The Five Pillars

### 1. Memory Copilot

Persistent memory across sessions with semantic search.

**Storage:** `.claude/memory/entries/<uuid>.md` (committed, travels with repo)
**Commands:** `cc memory store`, `cc memory search`, `cc memory get`, `cc memory list`
**Index:** `cc memory index --rebuild` (local SQLite cache, gitignored)

**Location:** `tools/cc/`

### 2. Agents

7 specialized agents + 1 setup agent (kc). Every agent embeds named industry methodology — IDEO (sd), Nielsen + Rams + Atomic Design (design), ADR/Fitness Functions (ta), Kent Beck (me), Diátaxis (doc), 12-Factor/SRE (do), Meszaros (qa). Security (STRIDE/DREAD), copywriting (MailChimp Voice & Tone), and creative direction (Litmus Test) are available as @include skills.

**Location:** `.claude/agents/`

### 3. Skills

Load via native @include (recommended), `cc skill` CLI, or Skills Copilot MCP server (optional).

**Native:** `@include .claude/skills/NAME/SKILL.md`

**cc CLI:** `cc skill get <name>`, `cc skill search "<query>"`, `cc skill list`, `cc skill evaluate`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Everyone-Needs-A-Copilot/claude-copilot](https://github.com/Everyone-Needs-A-Copilot/claude-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
