---
trigger: always_on
description: Context for Claude Code when working with this repository.
---

# CLAUDE.md

Context for Claude Code when working with this repository.

## Commands

```bash
npm run bundle    # Build: clean + tsup + make executable + create .tgz
npm test          # Run unit tests (vitest + happy-dom)
npm run dev       # Development server (tsx, no build)
npm start         # Run built server from lib/server.js
```

## Architecture

```
src/
├── server.ts          # MCP server entry — registers all tools + resources
├── session/           # Session state (state.ts) + lifecycle (lifecycle.ts)
├── providers/         # SessionProvider implementations
│   ├── registry.ts    # getProvider() — routes to local or cloud provider
│   ├── local-browser.provider.ts  # Chrome/Firefox/Edge/Safari
│   ├── local-appium.provider.ts   # iOS/Android via Appium
│   └── cloud/
│       └── browserstack.provider.ts  # BrowserStack (browser + App Automate)
├── tools/             # One file per MCP tool (see Tool Pattern below)
├── resources/         # One file per MCP resource (see Recording below)
├── recording/         # step-recorder.ts (withRecording HOF) + code-generator.ts
├── scripts/           # Browser/mobile scripts executed via browser.execute() — no try/catch, raw data only
├── locators/          # Element detection, selector generation, XML parsing (mobile)
├── config/            # appium.config.ts — iOS/Android capability builders
├── utils/             # parse-variables.ts, zod-helpers.ts (coerceBoolean)
└── types/             # ToolDefinition, ResourceDefinition, RecordedStep interfaces
```

### Session State

Single active session model in `src/session/state.ts`:

```typescript
// Private state — access via getState() or getBrowser()
export function getBrowser(): WebdriverIO.Browser { ... }
export function getState() { return state; }
export interface SessionMetadata {
  type: 'browser' | 'ios' | 'android';
  capabilities: Record<string, unknown>;
  isAttached: boolean;
  provider?: 'local' | 'browserstack';   // set at session start; used by lifecycle to call provider hooks
  tunnelHandle?: unknown;                 // opaque handle returned by provider.startTunnel(), passed back to onSessionClose()
}
```

Session lifecycle managed via `src/session/lifecycle.ts`:
- `registerSession()` — registers browser + metadata + history, handles transition sentinel; calls `provider.onSessionClose()` on orphaned sessions
- `handleSessionTransition()` — appends `__session_transition__` step to outgoing session
- `closeSession()` — terminates or detaches, marks endedAt, calls `provider.onSessionClose()`, cleans up maps

### Tool Pattern

All tools follow this structure:

```typescript
// 1. Definition with Zod schema
export const myToolDefinition: ToolDefinition = {
  name: 'my_tool',
  description: 'What it does',
  inputSchema: {
    param: z.string().describe('Parameter description'),
  },
};

// 2. Implementation
export const myTool: ToolCallback = async ({ param }: { param: string }) => {
  try {
    const browser = getBrowser();
    // ... implementation
    return { content: [{ type: 'text', text: `Result` }] };
  } catch (e) {
    return { content: [{ type: 'text', text: `Error: ${e}` }] };
  }
};

// 3. Register in server.ts via the registerTool helper
registerTool(myToolDefinition, myTool);
```

### Recording

Selected tools are wrapped with `withRecording()` in `server.ts`. Steps accumulate in `state.sessionHistory` (keyed by sessionId).

MCP resources expose live session data — all at fixed URIs discoverable via ListResources:

**Session history:**
- `wdio://sessions` — index of all sessions
- `wdio://session/current/steps` — current session step log
- `wdio://session/current/code` — generated WebdriverIO JS for current session
- `wdio://session/{sessionId}/steps` — step log for any session (URI template)
- `wdio://session/{sessionId}/code` — generated JS for any session (URI template)

**Live page state (current session):**
- `wdio://session/current/elements` — interactable elements (viewport-only; use `get_elements` tool with `inViewportOnly: false` for all)
- `wdio://session/current/accessibility` — accessibility tree
- `wdio://session/current/screenshot` — screenshot (base64)
- `wdio://session/current/cookies` — browser cookies
- `wdio://session/current/tabs` — open browser tabs
- `wdio://session/current/contexts` — native/webview contexts (mobile)
- `wdio://session/current/context` — currently active context (mobile)
- `wdio://session/current/app-state` — mobile app state
- `wdio://session/current/geolocation` — device geolocation
- `wdio://session/current/capabilities` — resolved WebDriver capabilities for the active session
- `wdio://browserstack/local-binary` — platform-specific download URL and daemon start command for BrowserStack Local binary

### Build

- **tsup** bundles `src/server.ts` → `lib/server.js` (ESM)
- Shebang preserved for CLI execution
- `zod` externalized

## Key Files

| File                                               | Purpose                                       |
|----------------------------------------------------|-----------------------------------------------|
| `src/server.ts`                                    | MCP server init, tool + resource registration |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webdriverio/mcp](https://github.com/webdriverio/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
