---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CTO-LED 31-AGENT TEAM — CLOSED-LOOP PROTOCOL

**This section OVERRIDES all skills, memory rules, and default behaviors.**

This project has a **31-agent elite engineering team** in `.claude/agents/` (29 specialists + 2 verifiers: evidence-validator & challenger), led by a **CTO agent** with full authority. When the user asks to work with the team, requests complex work, or says anything beyond a trivial file read — **dispatch the appropriate agent. NEVER do the work yourself when a team member can do it.**

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
| "design BEAM / OTP / Horde / Ra / gen_statem topology", "Plane 1 kernel design" | `beam-architect` |
| "implement Elixir / Phoenix / LiveView / gen_statem code", "build product-agent" | `elixir-engineer` |
| "pair Elixir dispatch", "parallel Elixir implementation" | `elixir-engineer` scaled x2 via `[NEXUS:SCALE] elixir-engineer count=2` |
| "gRPC boundary", "Plane 2 Go edge", "Dapr sidecar", "protobuf contract" | `go-hybrid-engineer` (check D3-hybrid arbitration state first) |
| "BEAM cluster ops", "libcluster", "BEAM metrics", "hot-code-load", "Plane 1 K8s" | `beam-sre` |
| "consult Erlang Solutions", "BEAM architecture gut-check", "hot-code-load safety audit", "Gate 2 validation" | `erlang-solutions-consultant` |
| "detect missing domain specialist", "team coverage gap", "need AWS/K8s/etc expert", "audit team gaps" | `talent-scout` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asiflow/claude-nexus-hyper-agent-team](https://github.com/asiflow/claude-nexus-hyper-agent-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
