---
trigger: always_on
description: Expo React Native mobile app that connects to a Claude Code Remote (CCR) server over Tailscale VPN. Manages Claude Code sessions from your phone: create sessions, stream output live, approve/deny tool use, and receive push notifications.
---

# Claude Code Remote App

## What This Is

Expo React Native mobile app that connects to a Claude Code Remote (CCR) server over Tailscale VPN. Manages Claude Code sessions from your phone: create sessions, stream output live, approve/deny tool use, and receive push notifications.

## Architecture

```
Expo App (this repo)
├── app/             Expo Router file-based routing
│   ├── _layout      Root layout + providers (QueryClient, GestureHandler, SafeArea)
│   └── (tabs)/      Tab navigator: Sessions, Projects, Cron, Settings
├── components/      Shared UI components
├── constants/       Theme (colors, spacing) + slash command registry
└── lib/             API client, WebSocket, Zustand store, types, notifications
```

### State Management

- **Server state**: TanStack React Query (`lib/api.ts`) — sessions, templates, projects, cron jobs, push settings
- **Local state**: Zustand with AsyncStorage persistence (`lib/store.ts`) — host config, transient session messages
- **Real-time**: WebSocket per active session (`lib/websocket.ts`) — streams messages, auto-reconnects on close

### Theming

Dual light/dark theme matching system appearance. Pattern:

```typescript
const colors = useColors();                        // returns LightColors or DarkColors
const styles = useThemedStyles(colors, makeStyles); // memoized StyleSheet factory

const makeStyles = (c: ColorPalette) => StyleSheet.create({ ... });
```

All color tokens live in `constants/theme.ts`. The `ColorPalette` interface has tokens for background, card, text, code blocks, tool cards, tab bar, etc.

**Important**: `app.json` must have `"userInterfaceStyle": "automatic"` for system theme switching. This is a native-level config — changing it requires a dev client rebuild.

### API Client

`lib/api.ts` exports React Query hooks for every endpoint. Base URL comes from Zustand store (`hostConfig.address` + `hostConfig.port`). All requests use `http://` (Tailscale handles encryption).

Key polling intervals: sessions list 5s, individual session 5s, server status 10s, templates/projects/cron jobs 30s.

### WebSocket

`lib/websocket.ts` — `useSessionStream(sessionId)` hook connects to `ws://{host}/ws/sessions/{id}`. Returns `{ messages, isConnected, disconnect }`. Auto-reconnects after 3s. Filters out ping messages.

**Deduplication**: The WebSocket replays the full session message history on each new connection. To prevent duplicates on re-entry, `useSessionStream` calls `clearMessages(sessionId)` on mount before connecting. The WS stream is the single source of truth — stale messages in the Zustand store are wiped each time.

## File Overview

| File | Purpose |
|------|---------|
| `app/_layout.tsx` | Root layout — providers, StatusBar style |
| `app/(tabs)/_layout.tsx` | Tab navigator (Sessions, Projects, Cron, Settings) |
| `app/(tabs)/sessions/index.tsx` | Session list with filter chips |
| `app/(tabs)/sessions/[id].tsx` | Session detail — message stream, input bar, action menu |
| `app/(tabs)/sessions/[id]/mcp.tsx` | Session MCP servers list with health status |
| `app/(tabs)/sessions/[id]/skills.tsx` | Session available skills list |
| `app/(tabs)/sessions/create.tsx` | Full-screen session creation form |
| `app/(tabs)/sessions/workflows/index.tsx` | Workflow list |
| `app/(tabs)/sessions/workflows/[id].tsx` | Workflow detail with step form and DAG view |
| `app/(tabs)/cron/_layout.tsx` | Cron tab stack navigator |
| `app/(tabs)/cron/index.tsx` | Cron job list with filters and FAB |
| `app/(tabs)/cron/[id].tsx` | Cron job detail with run history |
| `app/(tabs)/projects/index.tsx` | Project list with FAB for creating projects |
| `app/(tabs)/projects/[id].tsx` | Project detail with sessions, cloning/error states |
| `app/(tabs)/projects/create.tsx` | Create project — blank (git init) or clone from URL |
| `app/(tabs)/settings/index.tsx` | Settings — server config, push settings |
| `app/(tabs)/settings/templates/index.tsx` | Template list |
| `app/(tabs)/settings/templates/[id].tsx` | Template edit form |
| `app/(tabs)/settings/analytics.tsx` | Usage analytics dashboard |
| `app/(tabs)/settings/mcp.tsx` | MCP server list and health |
| `app/(tabs)/settings/rules.tsx` | Auto-approval rules management |
| `app/(tabs)/settings/usage.tsx` | Claude API usage and rate limits |
| `components/MessageCard.tsx` | Message router — renders correct card by message type |
| `components/AssistantTextCard.tsx` | Markdown renderer for assistant responses |
| `components/ApprovalCard.tsx` | Tool approval/denial UI |
| `components/ToolUseCard.tsx` | Tool invocation display |
| `components/ToolResultCard.tsx` | Tool output display |
| `components/BashOutputCard.tsx` | Bash command output display |
| `components/InputBar.tsx` | Text input + send button + attachment picker |
| `components/CommandAutocomplete.tsx` | Slash command dropdown above input |
| `components/SessionCard.tsx` | Session list item |
| `components/SessionInfoBar.tsx` | Session info bar (project, cost, model, context %) |
| `components/CreateSessionSheet.tsx` | Bottom sheet session creation (BottomSheetScrollView) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gldc/claude-code-remote-app](https://github.com/gldc/claude-code-remote-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
