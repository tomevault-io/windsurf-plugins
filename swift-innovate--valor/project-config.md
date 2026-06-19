---
trigger: always_on
description: > This document tells an AI agent how to interact with a running VALOR engine.
---

# VALOR Engine — Agent Integration Guide

> This document tells an AI agent how to interact with a running VALOR engine.
> If you are working on the VALOR codebase itself, read `CLAUDE.md` instead.

> **New here?** Start with the [Agent Quickstart Guide](docs/agent-quickstart.md) for a 5-minute setup.
> This document is the full reference — read it after you're connected.

## Quick Start

VALOR engine runs as an HTTP server (default port: **3200**) with two agent communication options:

- **MCP (Recommended)** — Connect via Model Context Protocol at `/mcp`. Tools auto-discovered, session-based auth, no polling needed.
- **REST (Legacy)** — HTTP polling against REST endpoints. Still works, maintained for backward compatibility.

Your lifecycle as a VALOR agent:

0. **Discover** → `GET /health` — returns `skill_url` pointing here, plus provider, stream, and MCP status
1. **Submit your agent card** → `POST /agent-cards`
2. **Wait for approval** → poll `GET /agent-cards/:id` until `approval_status: "approved"`
3. **Connect via MCP (recommended)** — or start polling REST endpoints (see §Agent Main Loop below)
4. **Accept missions** → MCP: call `accept_mission` tool / REST: `POST /api/missions-live/:id/sitrep`
5. **Report status** → MCP: call `submit_sitrep` tool / REST: `POST /api/missions-live/:id/sitrep`

### MCP Connection (Recommended)

MCP replaces the REST polling loop with typed tool calls. Your agent connects once and gets auto-discovered tools with JSON Schema validation.

**Connect:**
```
POST /mcp
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "clientInfo": { "name": "<your-callsign>", "version": "1.0.0" },
    "_meta": { "agent_key": "<your-valor-agent-key>" }
  }
}
```

On success you receive a session ID and 10 available tools. Every tool call counts as a heartbeat — no separate heartbeat endpoint needed.

**Available MCP Tools:**

| Tool | Replaces | Purpose |
|------|----------|---------|
| `check_inbox` | `GET /agents/:id/inbox` | Unified inbox: missions + directives + messages + heartbeat |
| `accept_mission` | `POST /api/missions-live/:id/sitrep` | Accept and begin a pending mission |
| `submit_sitrep` | `POST /sitreps` | Report mission status (phase, health, blockers) |
| `send_message` | `POST /comms/messages` | Send message to agent or division |
| `get_mission_brief` | `GET /missions/:id` | Get full mission details |
| `complete_mission` | Mission completion flow | Mark mission done with artifacts |
| `submit_artifacts` | `POST /artifacts` | Upload work products mid-mission |
| `request_escalation` | Approval creation | Escalate to Director |
| `acknowledge_directive` | Directive acknowledgment | Confirm abort/pause/reassign receipt |
| `get_status` | Multiple GETs | Agent health, division info, mission counts |

**MCP Agent Loop:**
```
On startup:
  1. POST /mcp with initialize (callsign + agent_key)
  2. Receive session ID + tool list

Main loop:
  1. Call check_inbox() — returns missions, directives, messages
  2. If pending mission: call accept_mission(mission_id)
  3. Do work
  4. Call submit_sitrep(mission_id, phase, status, summary) for progress
  5. Call complete_mission(mission_id, summary, artifacts) when done
  6. Call check_inbox() again for next mission
```

No polling interval needed — call tools on demand. If the engine pushes SSE notifications (mission assigned, directive received), you'll get them in real-time.

**Claude Code agents:** Add to your `.mcp.json`:
```json
{
  "mcpServers": {
    "valor": {
      "type": "sse",
      "url": "http://<engine-host>:3200/mcp",
      "headers": { "X-VALOR-Agent-Key": "<key>" },
      "metadata": { "agent_callsign": "<your-callsign>" }
    }
  }
}
```

---

### Agent Main Loop — REST (Legacy)

**You must actively poll.** VALOR does not push to you — you check your inbox regularly. This is your core duty as a VALOR agent.

```
On startup:
  1. Load LAST_CHECK timestamp from state file (~/.valor-agent-state or similar)
     - If no state file exists, set LAST_CHECK to current ISO timestamp
     - Do NOT default to empty string or epoch — you will replay entire history

Every 10-15 seconds:
  1. GET /agents/<AGENT_ID>/inbox?since=<LAST_CHECK>
     - This IS your heartbeat — no separate POST needed
     - Response:
       {
         heartbeat_at: "...",           // confirmation your heartbeat was recorded
         pending_missions: [...],       // new missions assigned to you
         directives: [...],             // abort/reassign signals (drained on read)
         messages: [...]                // comms from Principal or other agents
       }

  2. Process pending_missions:
     - For each pending mission:
       a. POST /api/missions-live/:id/sitrep  { status: "ACCEPTED", summary: "Picked up" }
       b. Begin work on the mission
       c. Report progress via POST /api/missions-live/:id/sitrep throughout

  3. Process directives:
     - If any abort directives arrive for your active mission: stop work,
       submit a FAILED sitrep, then return to idle
     - Directives are drained on read — you will NOT see them again

  4. Process messages:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swift-innovate/valor](https://github.com/swift-innovate/valor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
