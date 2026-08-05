---
trigger: always_on
description: This guide explains how to use agent persona templates in Codex for specialized development tasks.
---

# Codex Agents Guide

This guide explains how to use agent persona templates in Codex for specialized development tasks.

---

## What Are Codex Agents?

Unlike Claude Code where agents are automatically invoked via the Task tool, Codex agents are **persona templates** designed to be copy-pasted into your Codex session. They provide specialized expertise for different phases of the development workflow.

**Key characteristics:**
- **Not executable** - They're prompt templates, not runnable programs
- **Context-dependent** - You must provide ALL context the agent needs
- **Structured output** - Each agent returns a specific report format
- **Manual integration** - You post results to Linear manually

---

## How to Use Agents in Codex

### Step-by-Step Process

1. **Select the right agent** for your task (see catalog below)
2. **Open a fresh Codex session** to avoid context pollution
3. **Copy the agent template** from `codex/agents/[agent-name].md`
4. **Paste into your Codex session** as the initial prompt
5. **Provide ALL context** the agent needs:
   - Ticket ID and full description
   - Relevant code files or discovery reports
   - Any previous phase outputs
   - Specific questions or focus areas
6. **Let the agent work** and produce a structured report
7. **Copy the report** and post to Linear or take action on recommendations
8. **Close the session** before starting the next phase

### Example Usage

```bash
# 1. View the agent template
cat codex/agents/architect-agent.md

# 2. Copy to clipboard (macOS)
cat codex/agents/architect-agent.md | pbcopy

# 3. Open Codex and paste the template
# 4. Add your context below the template:
#    "Here is the ticket I need you to analyze: APP-123..."
```

---

## Key Differences from Claude Code

| Aspect | Claude Code | Codex |
|--------|-------------|-------|
| **Invocation** | Automatic via Task tool | Manual copy-paste |
| **Linear I/O** | Orchestrator handles via MCP | You post results manually |
| **MCP tools** | `mcp__linear-server__*` available | No MCP tools |
| **Agent execution** | Runs in subprocess | Agent is inline prompt |
| **Context passing** | Task tool provides context | You embed ALL context in prompt |
| **Session management** | Multiple agents in one session | Fresh session per agent recommended |

### The Context-Passing Model

In Claude Code, the orchestrator-agent pattern works like this:
1. Command fetches ticket details via MCP
2. Command invokes agent via Task tool with context
3. Agent returns report
4. Command posts report to Linear via MCP

In Codex, **you are the orchestrator**:
1. **You** fetch ticket details (from Linear web UI or API)
2. **You** paste agent template + context into Codex
3. Agent returns report
4. **You** post report to Linear manually

---

## Agent Catalog

### Available Agents (11 total)

| Agent | Role | Primary Use Cases |
|-------|------|-------------------|
| [architect-agent](#architect-agent) | Senior Technical Architect | Discovery, planning, technical decomposition, doc-truth verification |
| [backend-engineer-agent](#backend-engineer-agent) | Backend Implementation Specialist | Server-side code, APIs, databases |
| [frontend-engineer-agent](#frontend-engineer-agent) | Frontend Implementation Specialist | UI components, frontend logic |
| [qa-engineer-agent](#qa-engineer-agent) | QA & Testing Specialist | Test creation, verification, anti-ballast discipline |
| [code-reviewer-agent](#code-reviewer-agent) | Code Quality Specialist | Code review, pattern compliance, convention guard verification |
| [technical-writer-agent](#technical-writer-agent) | Documentation Specialist | API docs, user guides |
| [security-engineer-agent](#security-engineer-agent) | Security Specialist | OWASP assessment, security review |
| [design-reviewer-agent](#design-reviewer-agent) | UI/UX Design Validator | Design review, accessibility |
| [epic-closure-agent](#epic-closure-agent) | Epic Completion Analyst | Follow-up discipline, convention guard audit, lessons learned |
| [ticket-context-agent](#ticket-context-agent) | Context Gathering Support | Parallel context for large epics |
| [entropy-auditor-agent](#entropy-auditor-agent) | Cross-Epic Entropy Auditor | Recurring consolidation audit, pragmatism-filtered findings |

---

### Architect Agent

**File**: `codex/agents/architect-agent.md`

**Role**: Senior Technical Architect (15+ years experience)

**When to Use**:
- Codebase discovery and service inventory
- Requirements decomposition into tickets
- Technical planning and architecture design
- Implementation guides (adaptation phase)
- Doc-truth verification (project memory vs HEAD) and guard-as-AC for conventions
- Vendor-surface discipline (new dependencies require justification)

**Example Invocation**:
```
[Paste architect-agent.md content]

---

## Your Task

Analyze the following PRD and decompose it into Linear tickets:

**PRD**: User Notification System
[Paste PRD content here]

**Existing Service Inventory**: 
[Paste or reference service inventory]

**Constraints**:
- Must integrate with existing event bus
- Target: 5-7 tickets, each 2-4 hours
```

---

### Backend Engineer Agent

**File**: `codex/agents/backend-engineer-agent.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bdouble/pm-vibecode-ops](https://github.com/bdouble/pm-vibecode-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
