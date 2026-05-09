---
trigger: always_on
description: Multi-agent orchestrator for Claude with layered security. Agents run in containers with no secrets, per-tool permissions, and a credential-injecting MITM proxy.
---

# Stockade — Agent Platform

Multi-agent orchestrator for Claude with layered security. Agents run in containers with no secrets, per-tool permissions, and a credential-injecting MITM proxy.

## Architecture

```
User (Terminal / Discord)
  ↓
Orchestrator  →  Router (scope → agent)  →  Dispatcher (query / HTTP)
  ↓                                            ↓
RBAC (user identity flows through chain)   Agent (Claude Code SDK session)
  ↓                                            ↓
Permissions (first-match-wins rules)       Credential Proxy (MITM, inject per-route)
  ↓                                            ↓
Gatekeeper (AI risk review for ask rules)  Network Policy (deny-by-default allowlist)
```

Key flow: message arrives on a channel → router resolves scope to agent ID → RBAC checks user access → dispatcher launches agent (in-process or container) → agent runs tools with permission checks → gatekeeper reviews `ask` rules → credential proxy injects secrets on the wire.

## Package Structure

```
packages/
├── orchestrator/   — Core: config, routing, dispatch, permissions, RBAC, gatekeeper, channels
│   └── src/
│       ├── index.ts          — Entry point, starts channels + dispatch loop
│       ├── config.ts         — YAML config loader, env substitution, path resolution
│       ├── router.ts         — Scope → agent ID resolution
│       ├── dispatcher.ts     — Agent dispatch (SDK query() or HTTP to container)
│       ├── permissions.ts    — First-match-wins permission engine (allow/deny/ask)
│       ├── rbac.ts           — User identity + role-based access control
│       ├── gatekeeper.ts     — AI risk assessment for tool invocations
│       ├── sessions.ts       — SQLite session persistence
│       ├── skills.ts         — Platform skills directory setup (~/.stockade/.claude/skills/)
│       ├── watch.ts          — Config hot-reload watcher
│       ├── agent-mcp.ts      — ask_agent MCP server (sub-agent delegation)
│       ├── channels/         — Terminal, Discord channel adapters
│       ├── containers/       — Docker container lifecycle management
│       ├── api/              — HTTP API (server, sessions endpoint)
│       ├── workers/          — Host worker process management
│       └── scheduler/        — Cron-based scheduled agent tasks
├── proxy/          — Credential proxy: MITM HTTPS, gateway API, network policy
│   └── src/
│       ├── index.ts          — Proxy entry point
│       ├── shared/           — Config, credentials, types (shared between HTTP/gateway)
│       ├── http/             — MITM proxy: header injection, cache markers, audit logging
│       ├── gateway/          — Gateway API: token auth, ref token issuance
│       ├── ssh/              — SSH proxy (port forwarding)
│       └── cli/              — apw CLI tool + read-claude-oauth.py utility
└── worker/         — Container agent worker (receives HTTP dispatch)
    └── src/
        ├── agent.ts          — Wraps Claude Code SDK session
        ├── server.ts         — HTTP server for dispatch
        ├── channel.ts        — Worker channel abstraction
        ├── session.ts        — Worker session state
        └── entrypoint.sh     — Docker entrypoint
```

## Config Location

Live config lives in `~/.stockade/` (decoupled from the source repo):
- `~/.stockade/config.yaml` — agents, channels, RBAC, containers, gatekeeper
- `~/.stockade/proxy.yaml` — credential provider, network policy, HTTP/SSH routes
- `~/.stockade/secrets/` — file-based credential store
- `~/.stockade/proxy/` — TLS certs (ca.crt, ca.key, ssh-ca)

Example configs in `config/` are templates for new users. Never commit real config or secrets.

## Permission System

First-match-wins rules per agent. Default when no rule matches: `ask` (HITL approval).

```yaml
permissions:
  - "deny:Write(/config/**)"    # /prefix = platform root (~/.stockade)
  - "ask:Bash(rm *)"            # Glob match on command
  - "allow:*"                   # Catchall — allow everything else
```

Path prefixes: `/` = platform root, `~/` = home, `./` = agent cwd, `//` = absolute POSIX.

## Skills

All skills live in one shared directory: `~/.stockade/.claude/skills/`. Changes are immediately visible to all agents — no restart or copy step needed.

- **Host agents** (`sandboxed: false`): pick up skills automatically because `sdkCwd = platformRoot` (`~/.stockade`), so the SDK loads `~/.stockade/.claude/skills/` natively.
- **Sandboxed agents** (Docker): the skills directory is volume-mounted read-write into the container at `/workspace/.claude/skills`.

**Skill visibility is controlled by permission rules**, not by which skills exist on disk. Use `allow:Skill(name)` / `deny:Skill(*)` in an agent's `permissions:` list:

```yaml
permissions:
  - "allow:Skill(tavily-search)"
  - "allow:Skill(goplaces)"
  - "deny:Skill(*)"       # hide everything else
  - "allow:*"
```

This maps to two enforcement layers:
1. **SDK-native** (`sdkSettings.permissions.allow/deny`) — denied skills are stripped from the agent's context entirely (0 tokens consumed).
2. **PreToolUse hook** — blocks invocation at runtime for any skill not explicitly allowed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dragooon/stockade](https://github.com/Dragooon/stockade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
