---
trigger: always_on
description: This is a standalone, client-side WebUI designed to interact with OpenAI-compatible APIs, specifically optimized for `llama.cpp`. It features a rich chat interface with conversation branching, file attachments, MCP (Model Context Protocol) tool integration, agentic reasoning, and mobile support via Capacitor.
---

# Standalone llama.cpp WebUI

## Project Overview
This is a standalone, client-side WebUI designed to interact with OpenAI-compatible APIs, specifically optimized for `llama.cpp`. It features a rich chat interface with conversation branching, file attachments, MCP (Model Context Protocol) tool integration, agentic reasoning, and mobile support via Capacitor.

**Key Characteristics:**
- **Client-Side Only:** No backend server for the UI itself. All data (history, keys) is stored locally in the browser via IndexedDB (Dexie).
- **Backend Agnostic:** Works with any OpenAI-compatible endpoint (local `llama-server`, external APIs like Vultr, DeepInfra, OpenRouter).
- **MCP Support:** Connect to MCP servers (SSE/WebSocket/StreamableHTTP) for tool calling (web search, file access, etc.).
- **Agentic:** Models can call tools, receive results, and continue reasoning — full tool execution loop.
- **Mobile Support:** Packaged as an Android app using Capacitor.

## Tech Stack
- **Framework:** SvelteKit 2 (Svelte 5.36 with Runes)
- **Language:** TypeScript (strict)
- **Styling:** TailwindCSS 4
- **UI Components:** bits-ui 2.14.4, tailwind-variants
- **Persistence:** Dexie.js (IndexedDB wrapper)
- **Build Tool:** Vite 7
- **Mobile:** Capacitor (Android)
- **MCP:** @modelcontextprotocol/sdk, zod
- **Testing:** Vitest (Unit), Playwright (E2E)
- **UI Development:** Storybook 10

## Architecture

### Core Services (`src/lib/services/`)
- **`ChatService` (`chat.ts`):** Stateless API layer. Handles HTTP requests, streaming responses, parsing `<think>` tags (reasoning models), tool call parsing, and formatting messages for the API. Supports `tools` and `tool_choice` parameters for MCP integration. Automatically injects current date/time and timezone into system messages. Has a `systemPromptOverride` field that, when set, replaces the global system message from settings — used by the search route to inject a search-focused system prompt.
- **`MCPService` (`mcp.service.ts`):** MCP protocol client. Handles WebSocket, SSE, and StreamableHTTP transports. Manages server connections, tool discovery, and tool execution. Auto-detects SSE endpoints by URL path (`/sse`).
- **`BuiltinToolsService` (`builtin-tools.service.ts`):** Provides browser-local tools (calculator, date math) that work without any MCP server. Tools are prefixed with `builtin_` and executed locally.
- **`DatabaseService` (`database.service.ts`):** Encapsulates all Dexie.js interactions. Manages persistence for conversations and messages.
- **`PropsService` (`props.service.ts`):** Fetches server properties endpoint (for llama.cpp server capabilities).
- **`SlotsService` (`slots.ts`):** Monitors server slot usage/capacity (specific to `llama.cpp` server).

### State Management (`src/lib/stores/`)
- **`ChatStore` (`chat.svelte.ts`):** The central store for UI state. Orchestrates message sending, agentic tool execution loop (send → model calls tool → execute → send results back → model may call more tools → loop up to `maxToolIterations`), regeneration, and manages the active conversation. `createConversation()` accepts an optional `navigateTo` parameter (string or callback `(convId) => string`) for custom post-creation navigation (used by search route).
- **`ConversationsStore` (`conversations.svelte.ts`):** Manages conversation lifecycle — CRUD, title updates, branching, navigation, import/export. Extracted from ChatStore.
- **`MCPStore` (`mcp.svelte.ts`):** MCP server state orchestration. Manages connections, health checks, tool definitions, and tool execution. Provides `getToolDefinitionsForLLM()` for injecting tools into API requests and `executeTool()` for running tools.
- **`MCPResourcesStore` (`mcp-resources.svelte.ts`):** MCP resource caching.
- **`AgenticStore` (`agentic.svelte.ts`):** Manages agentic sections in messages (reasoning blocks, tool calls).
- **`SettingsStore` (`settings.svelte.ts`):** Manages user preferences (API URL, model parameters, MCP server configs).
- **`ServerStore` (`server.svelte.ts`):** Server connection state and props.

### Data Model
- **Conversations:** stored in `conversations` table.
- **Messages:** stored in `messages` table. Supports a **tree structure** (branching) where each message points to a parent.
- **Attachments:** Stored as base64 strings within the message `extra` field.
- **MCP Server configs:** Stored in localStorage via settings (key: `mcpServers`).

## Key Directories
- `src/lib/components/app/`: Core application UI components organized by category:
  - `chat/`: Chat UI (messages, form, sidebar, settings)
  - `mcp/`: MCP server management UI (server cards, forms, connection logs, resource browser)
  - `actions/`: Action buttons and controls
  - `badges/`: Status badges
  - `content/`: Content renderers (markdown, syntax highlighting, collapsible blocks)
  - `forms/`: Form components
  - `navigation/`: Navigation components
  - `models/`: Model selection UI
  - `dialogs/`: Dialog components
  - `misc/`: Miscellaneous components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NandaIda/standalone_llamacpp_webui](https://github.com/NandaIda/standalone_llamacpp_webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
