---
trigger: always_on
description: Claude Code skill library for penetration testing and CTF work.
---

# red-run

Claude Code skill library for penetration testing and CTF work.

## Engagement Workflow

The orchestrator is invoked via `/red-run-ctf` slash command only — not by
natural language triggers. It contains all routing logic, approval gates,
and state management rules. **If you are a teammate** (spawned by a team
lead), **do NOT invoke the orchestrator skill.** Load technique skills via
`mcp__skill-router__get_skill()` instead — never via the Skill tool.

## Token Budget

Every token costs money and latency. Consider token impact when making ANY
change — agent templates, skill text, MCP responses, orchestrator prompts.
Prefer designs that minimize per-invocation token usage without sacrificing
needed functionality. Put hints in tool responses (loaded only when called)
rather than agent templates (loaded every invocation).

**No inline file templates.** Never embed file contents (YAML, shell scripts,
JSON, config files) directly in skill files, agent templates, or orchestrator
prompts. Store templates in `operator/templates/` and reference them by path.

## Architecture

### Orchestrator Variants

| Variant | Status | Execution Model |
|---------|--------|-----------------|
| `/red-run-ctf` | **Active** (default) | Agent teams — persistent teammates, peer messaging |
| `/red-run-legacy` | **Legacy** | Subagents — ephemeral, one skill per invocation |
| `/red-run-notouch` | Planned | DLP-safe — operator runs commands, reports sanitized output |
| `/red-run-train` | Planned | Training mode — guided walkthrough with explanations |

All variants share state.db, MCP servers, and technique skills. An engagement
started with one can be resumed with another. Invoke via slash command only.

### Agent Teams (`/red-run-ctf`)

The lead session runs the orchestrator skill, creates a team via `TeamCreate`,
spawns persistent domain teammates via `Agent` with `team_name`, assigns tasks
via `TaskCreate`/`TaskUpdate`, and chains vulnerabilities toward impact.
Teammates communicate via `SendMessage` and write to state.db through
state-mgr. All technique skills (67+) are served on-demand via the MCP
skill-router. Teammate spawn templates live in `teammates/`.

### MCP Servers

| Server | Location | Purpose |
|--------|----------|---------|
| skill-router | `tools/skill-router/` | Semantic skill discovery and loading (ChromaDB + embeddings) |
| nmap-server | `tools/nmap-server/` | Dockerized nmap scanning with input validation |
| shell-server | `tools/shell-server/` | TCP listener, reverse shell, local interactive process manager |
| state | `tools/state-server/` | Full read/write engagement state (SQLite) |
| browser-server | `tools/browser-server/` | Headless browser automation |
| rdp-server | `tools/rdp-server/` | Headless RDP automation via aardwolf |
| sliver-server | `tools/sliver-server/` | Sliver C2 gRPC wrapper (optional) |
| state-viewer | `operator/state-viewer/` | Read-only web dashboard for state.db (not MCP) |

In agent teams mode, **state-mgr** is the sole writer to state.db (LLM-level
dedup + graph coherence). **shell-mgr** owns shell lifecycle (listeners,
processes, upgrades) — teammates message shell-mgr for setup, then interact
with the MCP directly after session handoff. See each server's `README.md`.

## Skill Routing

The orchestrator makes every routing decision. Skills report findings
generically — they do not name next skills. The orchestrator calls
`search_skills(query)` to find technique skills, then spawns/messages the
appropriate teammate.

**Mandatory skill loading**: Never execute a technique without loading the
matching skill via `get_skill()`. Skills contain methodology, payloads, and
troubleshooting that general knowledge does not.

**Built-in sub-agents** (Explore, Plan, general-purpose) do NOT have MCP
access — use them only for local processing, never for target-level work.

## State Management

Engagement state lives in `engagement/state.db` (SQLite, managed by
state-server MCP). Tables: targets, ports, credentials, credential_access,
access, vulns, pivot_map, blocked, tunnels, state_events.

- `get_state_summary()` produces a compact markdown summary for consumption
- Teammates read state directly; all writes go through state-mgr
- The orchestrator polls `poll_events()` for real-time visibility
- Orchestrator uses state summary + pivot map to chain vulns toward impact

## Teammate Protocol

This section applies to all domain teammates spawned during engagements.
Infrastructure teammates (state-mgr, shell-mgr) have their own protocols
defined in their templates.

### Task Workflow

1. The lead assigns a task via `SendMessage` starting with `[TASK]`,
   including: skill name, target, and context.
2. Load the skill via `mcp__skill-router__get_skill(name="<skill-name>")`
   — call it directly, not via a subagent. If not callable yet, run
   `ToolSearch("select:mcp__skill-router__get_skill")` first. The full
   skill text MUST be in YOUR context window. **Never use the Agent tool
   or Skill tool to load skills.**
3. Execute the skill's methodology end-to-end.
4. Message state-mgr with findings using `[action]` protocol.
5. Message the lead with a structured summary.
6. Mark the task complete. **Wait for next assignment — never self-claim.**

### State Writes via state-mgr


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blacklanternsecurity/red-run](https://github.com/blacklanternsecurity/red-run) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
