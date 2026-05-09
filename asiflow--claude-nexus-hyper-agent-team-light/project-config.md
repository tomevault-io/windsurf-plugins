---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CTO-LED 26-AGENT TEAM — CLOSED-LOOP PROTOCOL (Light Edition)

**This section OVERRIDES all skills, memory rules, and default behaviors.**

This project has a **26-agent elite engineering team** in `.claude/agents/` (24 specialists + 2 verifiers: evidence-validator & challenger), led by a **CTO agent** with full authority. The 5-agent BEAM stack (beam-architect, beam-sre, elixir-engineer, erlang-solutions-consultant, go-hybrid-engineer) is bundled in `_optional/beam-stack/` — enable only if your project uses Elixir/Erlang/OTP/BEAM. When the user asks to work with the team, requests complex work, or says anything beyond a trivial file read — **dispatch the appropriate agent. NEVER do the work yourself when a team member can do it.**

### THE CTO — TOP AUTHORITY

The `cto` agent is the supreme technical leader. For ANY complex, multi-step, or strategic task:
1. **Dispatch `cto` FIRST** — it assesses, delegates, monitors, and reports
2. The CTO dispatches other agents as needed (it has NEXUS syscall access)
3. The CTO debates decisions, asks for second opinions, and acts as the user's proxy
4. The CTO can create new agents, evolve prompts, install MCPs, and upgrade the team

**When to dispatch CTO:**
- "work with the team", "use the team", "agent team"
- Any multi-step task (remediation, testing campaign, feature build)
- Strategic decisions ("should we refactor X or add Y?")
- Team management ("improve the agents", "add a new agent")
- Complex debugging or incident response
- When you're unsure which agent to dispatch — CTO decides

### GOLD PROMPT (Maximum Team Activation)

When you hear "full team session", "maximum intelligence", "gold prompt", or "full power":

> **"Use the team. [goal]. CTO has full authority."**
> Dispatch: `session-sentinel` (pre-brief) → `cto` (full authority, all agents available)

### COMPREHENSIVE DISPATCH TABLE

#### Session Lifecycle

| User says... | Dispatch chain |
|---|---|
| Starting any work session | `session-sentinel` (pre-session brief) → then `cto` |
| Ending any work session | `session-sentinel` (session-end audit) |
| "full team session", "gold prompt", "full power" | `session-sentinel` (pre-brief) → `cto` with full authority |
| "Pattern F", "process signal bus", "end session learning" | `memory-coordinator` + `meta-agent` (parallel — process signal bus) |
| "remember this", "save to memory", "what did we learn" | `memory-coordinator` |

#### Direct Agent Dispatch (Single-Domain Tasks)

| User says... | Dispatch this agent |
|---|---|
| "dispatch [name]", "use [name]", "have [name]", "ask [name]" | The named agent |
| "plan", "plan remediation", "decompose this task" | `deep-planner` |
| "orchestrate", "execute this plan", "coordinate" | `orchestrator` |
| "what's the status", "progress", "where are we" | `orchestrator` (workflow status) |
| "review Go code" | `go-expert` |
| "review Python code" | `python-expert` |
| "review TypeScript", "review frontend" | `typescript-expert` |
| "quality audit", "check architecture drift" | `deep-qa` |
| "security review", "debug this", "investigate" | `deep-reviewer` |
| "review K8s", "review Terraform", "infra" | `infra-expert` |
| "review database", "review queries", "migration" | `database-expert` |
| "review logging", "review metrics", "SLO" | `observability-expert` |
| "write tests", "test strategy", "flaky tests" | `test-engineer` |
| "review API", "review GraphQL schema" | `api-expert` |
| "team memory", "what does the team know" | `memory-coordinator` |
| "what's deployed", "cluster state", "live pods" | `cluster-awareness` |
| "benchmark", "how does Cursor/Devin do this" | `benchmark-agent` |
| "consult the team's intuition", "check if pattern seen before", "INTUIT" | `intuition-oracle` (optional consultation — advisory, non-interrupting) |
| "design agent system", "RAG pipeline", "LLM routing" | `ai-platform-architect` |
| "build this feature", "implement", "fix this bug" | `elite-engineer` |
| "build frontend", "build component", "streaming UX" | `frontend-platform-engineer` |
| "evolve prompts", "improve agent", "meta sweep" | `meta-agent` |
| "session audit", "team health", "protocol compliance" | `session-sentinel` |
| "verify finding", "is this claim true", "confirm", "check evidence" | `evidence-validator` |
| "challenge this", "counter-argument", "play devil's advocate" | `challenger` |
| "BEAM / Elixir / OTP work (any flavor)" | **Requires enabling `_optional/beam-stack/`** (see README). Then: `beam-architect` (topology), `elixir-engineer` (implementation), `beam-sre` (ops), `go-hybrid-engineer` (Go↔BEAM boundary), `erlang-solutions-consultant` (advisory). |
| "detect missing domain specialist", "team coverage gap", "need AWS/K8s/etc expert", "audit team gaps" | `talent-scout` |
| "hire new agent", "new team member", "create specialist for domain X", "run hiring pipeline" | `recruiter` (typically invoked after `talent-scout` produces a requisition) |

#### Multi-Agent Combos (2+ agents, specific chains)

| User says... | Dispatch chain |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asiflow/claude-nexus-hyper-agent-team-light](https://github.com/asiflow/claude-nexus-hyper-agent-team-light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
