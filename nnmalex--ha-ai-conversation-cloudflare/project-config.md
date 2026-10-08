---
trigger: always_on
description: This project connects Home Assistant's voice assistant to a remote AI agent running on Cloudflare. It consists of two sub-projects (separate repos in this folder):
---

# ha-ai — Home Assistant + Cloudflare AI Agent

## Project Overview

This project connects Home Assistant's voice assistant to a remote AI agent running on Cloudflare. It consists of two sub-projects (separate repos in this folder):

1. **`cf-ha-agent/`** — Cloudflare Agent (TypeScript) that connects to HA's MCP server and uses Workers AI for reasoning
2. **`cloudflare-conversation/`** — Home Assistant custom integration (Python) that bridges HA's Conversation API to the Cloudflare agent

### Why This Architecture

The built-in HA conversation integrations (Anthropic, OpenAI, etc.) send a large context on every request — all exposed entities, areas, floors as YAML. This is token-inefficient and slow.

Our approach: the Cloudflare agent connects directly to HA's MCP server and discovers tools/entities on-demand. The HA integration sends only minimal context per request (~200 bytes: text, area name, language).

## Architecture

```
Voice Satellite (e.g. Living Room)
  |  device_id, satellite_id, text
  v
HA Core -- cloudflare_conversation integration
  |  Resolves: device_id -> area_name, floor_name
  |  Sends: { text, area_name, floor_name, satellite_id, conversation_id, language }
  v
Cloudflare Agent (Durable Object, persistent SQLite state)
  |  System prompt includes area context
  |  Calls Workers AI with MCP tools
  v
Workers AI (glm-4.7-flash or llama-4-scout)
  |  Decides tool calls, e.g. HassTurnOn(domain="light", area="Living Room")
  v
HA MCP Server (/api/mcp, Streamable HTTP, Bearer token auth)
  |  Executes intent with area parameter
  v
Device action completed
```

## Sub-Project: cf-ha-agent (Cloudflare)

### Tech Stack
- Cloudflare Agents SDK (`agents`)
- Workers AI via Vercel AI SDK v6 (`workers-ai-provider`, `ai`)
- Base `Agent` class (MCP client, SQLite via `this.sql`, `onRequest()` for REST)
- TypeScript

### Key Design Decisions
- Uses base `Agent` class (not `AIChatAgent` which is designed for WebSocket chat UIs — our client makes REST POST calls)
- Manages conversation history in SQLite manually (messages table per conversation_id)
- `onStart()` calls `this.addMcpServer()` to connect to HA's MCP server (idempotent, survives restarts)
- `this.mcp.getAITools()` converts MCP tools to Vercel AI SDK format for the model
- Durable Object per conversation — state persists across requests without external DB
- Provider-agnostic via Vercel AI SDK — can swap Workers AI for Anthropic/OpenAI later

### Workers AI Models (with tool calling support)
- `@cf/zai-org/glm-4.7-flash` — 131K context, optimized for multi-turn tool calling, cheapest ($0.06/M input). Best default for home automation.
- `@cf/meta/llama-4-scout-17b-16e-instruct` — 131K context, best overall capability, multimodal
- `@cf/meta/llama-3.3-70b-instruct-fp8-fast` — 24K context, strongest reasoning

### Configuration (secrets / env vars, never hardcoded)
- `HA_MCP_URL` (var) — HA MCP server endpoint, e.g. `https://my-ha.example.com/api/mcp`
- `HA_ACCESS_TOKEN` (secret) — Long-Lived Access Token from HA
- `AGENT_API_KEY` (secret) — API key for the HA integration to authenticate to this agent
- `AI_MODEL` (var) — Workers AI model ID, default `@cf/zai-org/glm-4.7-flash`

### API Endpoint
```
POST /api/chat
Authorization: Bearer <AGENT_API_KEY>
Content-Type: application/json

{
  "text": "turn on the lights",
  "conversation_id": "01JExample",
  "language": "en",
  "context": {
    "area_name": "Living Room",
    "floor_name": "Ground Floor",
    "satellite_id": "assist_satellite.living_room_speaker"
  },
  "extra_system_prompt": "Always respond concisely for voice."
}
```

### MCP Connection Details
- HA MCP Server exposes Assist API tools: HassTurnOn, HassTurnOff, HassGetState, HassLightSet, HassClimateSetTemperature, HassMediaSearchAndPlay, etc.
- Transport: Streamable HTTP (`/api/mcp`) preferred, SSE (`/mcp_server/sse`) as fallback
- Auth: Bearer token in Authorization header
- Stateless per-request on HA side — Agents SDK handles reconnection and caches tool list
- Tools accept `area` parameter — this is how the agent targets the correct room

### Custom Integrations and MCP Tool Discovery

HA's MCP server exposes **only** the Assist API intents — there is no generic `call_service` tool. This is a deliberate HA design decision for safety. However, the tool set is extensible:

#### What's available out of the box
- **Built-in intents** become MCP tools automatically: lights, climate, media, covers, fans, vacuums, timers, shopping lists, weather, date/time
- **Media intents**: `HassMediaPause`, `HassMediaUnpause`, `HassMediaNext`, `HassMediaPrevious`, `HassSetVolume`, `HassSetVolumeRelative`, `HassMediaPlayerMute`, `HassMediaPlayerUnmute`, `HassMediaSearchAndPlay`
- **Calendar/To-do tools**: `calendar_get_events`, `todo_get_items` (if entities are exposed)
- **`GetLiveContext`**: returns real-time state of all exposed entities

#### Extending MCP with HA scripts (the escape hatch)
Any HA script that is **exposed** via Settings > Voice Assistants automatically becomes an MCP tool. This is HA's intended mechanism for adding custom capabilities without code changes on the agent side.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nnmalex/ha-ai-conversation-cloudflare](https://github.com/nnmalex/ha-ai-conversation-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
