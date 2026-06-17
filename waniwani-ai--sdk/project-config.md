---
trigger: always_on
description: SDK for [app.waniwani.ai](https://app.waniwani.ai) — open-source flow engine plus optional hosted tier for tracking, KB, and chat.
---

# WaniWani SDK

SDK for [app.waniwani.ai](https://app.waniwani.ai) — open-source flow engine plus optional hosted tier for tracking, KB, and chat.

## Public API tiers

The SDK ships three tiers. **When recommending APIs for new code, always use OSS or Free Tier — never Legacy.**

### OSS (no API key required)

Truly open source. Works standalone with any state backend.

- `createFlow`, `StateGraph`, `START`, `END`, `redacted`, `createFlowTestHarness` from `@waniwani/sdk/mcp`
- `KvStore` interface, `MemoryKvStore` from `@waniwani/sdk/mcp`

`createFlow().compile()` requires either an explicit `{ store }` OR `WANIWANI_API_KEY`. With neither, it throws at compile time with a clear migration message.

### Free tier (one env var: `WANIWANI_API_KEY`)

Same SDK, hosted features added when the key is present.

- `WaniwaniKvStore` (hosted flow state) — used by `createFlow` default when key is set
- `waniwani()`, `tracking/*`, `withWaniwani`, `createTrackingRoute`, `widget-token`, `scoped-client` from `@waniwani/sdk` and `@waniwani/sdk/mcp`
- `createKbClient` from `@waniwani/sdk/kb`
- `useWaniwani` from `@waniwani/sdk/mcp/react` (also OSS — degrades to no-op without config; BYO endpoint also supported)
- `WaniwaniChat` (hosted React chat — recommended), themes, `embed.js` (IIFE for non-React hosts), `styles.css` from `@waniwani/sdk/chat`
- `ChatEmbed` from `@waniwani/sdk/chat` — bare-bones bring-your-own-backend primitive. Exposed but **not** the recommended path for new code; reach for it only when self-hosting the chat backend.

`withWaniwani` is no-key-safe: it wraps tools and bridges session metadata even without an API key, and its own auto-captured `tool.called` events are internally guarded (`safeTrack`). User-initiated tracking calls are **not**: `client.track.*`, `identify()`, and the scoped client throw `WANIWANI_API_KEY is not set` when no key is configured.

### Legacy (preserved, undocumented, marked `@deprecated`)

Still used by ~14 internal customer MCPs. Kept exported for back-compat. **Never suggest these for new code.** They will move to dedicated `@waniwani/sdk/legacy*` entry points in a future minor release.

- `createTool`, `createResource`, `registerTools` from `@waniwani/sdk/mcp`
- `toNextJsHandler` (`@waniwani/sdk/next-js`), `toExpressJsHandler` (`@waniwani/sdk/express-js`), `createApiHandler` (`@waniwani/sdk/chat/server`)
- `ChatCard` (and `ChatCardProps`) — canonical import is now `@waniwani/sdk/legacy`. Still re-exported from `@waniwani/sdk/chat` for back-compat; that re-export will be removed in a future minor release. Superseded by `WaniwaniChat`.
- All MCP-widget React hooks except `useWaniwani`: `WidgetProvider`, `useWidgetClient`, `useDisplayMode`, `useToolOutput`, `useSafeArea`, `useMaxHeight`, `useTheme`, `useLocale`, `useCallTool`, `useSendFollowUp`, `useFlowAction`, `useUpdateModelContext`, `useRequestDisplayMode`, `useToolResponseMetadata`, `useWidgetState`, `useIsChatGptApp`, `useOpenExternal`
- `InitializeNextJsInIframe`, `LoadingWidget`, `DevModeProvider`, mocks, `detectPlatform`, `isMCPApps`, `isOpenAI`

### Internal (not part of the public API)

`@waniwani/sdk/internal` is a private entry point for the WaniWani platform (app.waniwani.ai) to reuse SDK primitives that should not be exposed to third-party consumers. **Never document these in user-facing docs. Never suggest them for new code outside the WaniWani monorepo.**

- `replayScenario`, `ConversationTurnResult`, `ConversationResult`, `EvalScenario`, `ChatResult`, `ToolCallTrace`, `TurnAssertion`, `EvalScenarioType` from `@waniwani/sdk/internal` — replay a recorded UIMessage conversation against an MCP-backed chat server. Used by the compliance/evals features in the app.

The old `@waniwani/sdk/evals` public entry (with `chat`, `conversation`, `saveScenario`, `loadScenarios`, `braintrust`/`autoevals` scorers) was removed and is **not** restored at `/internal` — only the surface the app actually uses.

## Project structure

```
src/
├── index.ts              # waniwani() client, defineConfig, WaniWaniError
├── waniwani.ts
├── tracking/             # Event tracking (free tier)
├── kb/                   # Knowledge base (free tier)
├── internal/             # Private surface for app.waniwani.ai (replayScenario)
├── legacy/               # LEGACY entry points (createTool, createResource, chat-server adapters, ChatCard)
├── mcp/
│   ├── index.ts          # Public exports for @waniwani/sdk/mcp
│   ├── server/
│   │   ├── flows/        # OSS: createFlow, StateGraph
│   │   ├── kv/           # OSS interface + MemoryKvStore + WaniwaniKvStore
│   │   ├── tools/        # LEGACY: createTool
│   │   ├── resources/    # LEGACY: createResource
│   │   ├── with-waniwani/# Free tier wrapper (no-key safe)
│   │   ├── tracking-route.ts
│   │   ├── widget-token.ts
│   │   └── scoped-client.ts
│   └── react/            # Mostly LEGACY (only useWaniwani is non-legacy)
└── chat/
    ├── web/              # Free tier chat widget (WaniwaniChat, ChatEmbed, embed.js IIFE)
    └── server/           # Back-compat shim — re-exports from `src/legacy/chat/server/`
```

## Usage

OSS — no API key:

```ts
import { createFlow, MemoryKvStore, START, END } from "@waniwani/sdk/mcp";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WaniWani-AI/sdk](https://github.com/WaniWani-AI/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
