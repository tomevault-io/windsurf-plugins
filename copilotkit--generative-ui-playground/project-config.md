---
trigger: always_on
description: Generative UI playground showcasing three protocols for AI-powered interfaces.
---

# UI Protocols Demo

Generative UI playground showcasing three protocols for AI-powered interfaces.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  FRONTEND (Next.js)                                                  │
│  ├── Protocol tabs: [Static+MCP] [A2UI]                             │
│  ├── CopilotKitProvider with agent switching                        │
│  ├── renderActivityMessages: A2UIRenderer (a2ui mode only)          │
│  ├── useRenderToolCall: WeatherCard, StockCard                      │
│  ├── useHumanInTheLoop: TaskApprovalCard                            │
│  └── CopilotSidebar                                                 │
└────────────────────────────┬────────────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│ "default" Agent         │     │ "a2ui" Agent            │
│ BasicAgent + MCP Apps   │     │ HttpAgent → Python A2A  │
│ Port: 3001 (MCP)        │     │ Port: 10002             │
└─────────────────────────┘     └─────────────────────────┘
```

## Three Protocols

### 1. Static GenUI
Pre-built React components rendered by frontend hooks.
- **useRenderToolCall**: Display-only rendering (WeatherCard, StockCard)
- **useHumanInTheLoop**: Interactive approval (TaskApprovalCard)
- **Files**: `src/app/components/static-tools/*.tsx`

### 2. MCP Apps
HTML/JS apps served by MCP server, rendered as sandboxed iframes.
- 6 apps: Flights, Hotels, Trading, Kanban, Calculator, Todo
- MCP server registers tools with UI resources (`mimeType: "text/html+mcp"`)
- MCPAppsMiddleware bridges MCP to AG-UI events
- **CRITICAL**: Requires `public/sandbox.html` — the iframe sandbox page that MCPAppsActivityRenderer loads app content into. Without it, iframes show 404.
- **Files**: `mcp-server/apps/*.html`, `mcp-server/server.ts`, `public/sandbox.html`

### 3. A2UI (Agent-to-UI)
Agent-composed declarative JSON UI, rendered dynamically.
- Python agent (Google ADK) generates A2UI JSON at runtime
- General-purpose UI generator: forms, lists, cards, confirmations
- No external data dependencies - generates UI from user descriptions
- A2UIRenderer processes activity messages
- **Files**: `a2a-agent/agent/*.py`

## Widget Builder

The "Widget Builder" link in the header opens the official A2UI Composer at https://a2ui-composer.ag-ui.com/.

## Development

### Start All Services

```bash
# Terminal 1: MCP Server
cd mcp-server && npm run dev

# Terminal 2: Python A2A Agent
cd a2a-agent && python -m agent

# Terminal 3: Next.js Frontend
npm run dev
```

### URLs
- Frontend: http://localhost:3000
- MCP Server: http://localhost:3001/mcp
- A2A Agent: http://localhost:10002

## Environment Variables

```bash
# .env
OPENAI_API_KEY=sk-...          # OpenAI API key for gpt-5.2
MCP_SERVER_URL=http://localhost:3001/mcp
A2A_AGENT_URL=http://localhost:10002
```

## Production URLs (Railway)

Live deployment on Railway:
- **Frontend**: https://frontend-production-456e.up.railway.app
- **MCP Server**: https://mcp-server-production-5419.up.railway.app
- **A2A Agent**: https://a2a-agent-production.up.railway.app

Railway Project: `ui-protocols-demo`

## Key Packages

```json
{
  "@copilotkit/react-core": "Frontend provider and hooks",
  "@copilotkit/react-ui": "CopilotSidebar component",
  "@copilotkit/a2ui-renderer": "A2UI message renderer",
  "@copilotkitnext/runtime": "CopilotRuntime backend",
  "@copilotkitnext/agent": "BasicAgent class",
  "@ag-ui/mcp-apps-middleware": "MCP Apps → AG-UI bridge"
}
```

## File Structure

```
ui-protocols-demo/
├── src/app/
│   ├── page.tsx                 # Main page with agent switching
│   ├── theme.ts                 # A2UI v0.8 theme
│   ├── api/
│   │   ├── copilotkit/          # CopilotRuntime API route (default agent)
│   │   └── copilotkit-a2ui/     # CopilotRuntime API route (A2UI agent)
│   └── components/
│       ├── CopilotContextProvider.tsx   # Static tool hooks
│       ├── static-tools/        # Weather, Stock, TaskApproval cards
│       ├── protocol-cards/      # Educational protocol cards
│       ├── ComparisonTable.tsx  # Protocol comparison
│       └── PromptPill.tsx       # Clickable suggestion pills
├── mcp-server/                  # MCP Apps server (see mcp-server/CLAUDE.md)
└── a2a-agent/                   # Python A2A agent (see a2a-agent/CLAUDE.md)
```

## Agent Switching

Frontend uses `useState` to toggle between agents:
- `"default"`: Static GenUI + MCP Apps (BasicAgent + MCPAppsMiddleware)
- `"a2ui"`: General-purpose UI generator (HttpAgent → Python A2A backend)

The `agent` prop on CopilotKitProvider determines which backend agent handles requests.

### Pending Message Pattern

Protocol card pills can trigger automatic mode switches. When clicking a pill for a different mode than currently active, the app:

1. Sets `pendingMessage` state in `Home` component (outside provider)
2. Switches `activeAgent` state, triggering provider remount
3. `PageContent` (inside provider) has `useEffect` that watches `pendingMessage`
4. After remount, `useEffect` sends the message and clears state


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CopilotKit/generative-ui-playground](https://github.com/CopilotKit/generative-ui-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
