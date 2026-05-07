---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture

```
HA Assist (Voice/Text) → HA Custom Component (Python) → Home Mind Server (Express/TS) → LLM API (Anthropic/OpenAI/Ollama) + Shodh Memory + HA REST API
```

**Single path, no fallbacks.** All interactions flow through home-mind-server. Shodh Memory is the only memory backend (required, no SQLite fallback). LLM provider is selected via `LLM_PROVIDER` env var (default: `anthropic`).

### Home Layout Index

`TopologyScanner` (`ha/topology-scanner.ts`) runs at startup and every 30 minutes. It uses the HA template API (`POST /api/template`) with a single Jinja2 query that calls `floors()`, `floor_name()`, `floor_areas()`, `area_name()`, `area_entities()`, and `areas()` (available since HA 2024.4). Unassigned areas are derived by exclusion — areas not returned by any `floor_areas()` call.

Builds a compact `floor → room → [entity_ids]` text section and injects it into every system prompt alongside the device cheat sheet. This gives the LLM spatial awareness without tool calls — it knows which floor and room every device belongs to before reasoning begins.

If the template API fails (older HA, network error), the scanner logs a warning and injects nothing — the rest of the system works normally.

### Device Capability Index

`DeviceScanner` (`ha/device-scanner.ts`) runs at startup and every 30 minutes. It fetches all `light.*` entities, reads `supported_color_modes` attributes, and builds `DeviceCapabilityProfile` objects with pre-computed `whiteMethod` and `colorMethod`. These are formatted as a markdown cheat sheet and injected into every system prompt via `buildSystemPrompt()` / `buildSystemPromptText()`.

**White method precedence**: `rgbw`/`rgbww` → `rgbw_color: [0,0,0,255]`; `color_temp` → `color_temp_kelvin`; `rgb`/`xy`/`hs` → `rgb_color: [255,255,255]`; else → none.

**`DEVICE_OVERRIDES`** env var (JSON) allows per-entity overrides for devices with incorrect HA-reported modes (e.g. Gledopto GL-C-008P always reports `color_temp+xy` regardless of wiring). Overrides are applied after auto-detection. Fields: `whiteMethod` and/or `colorMethod`.

### Request Flow (IChatEngine.chat)

1. Load user's facts from Shodh via semantic search (query = current message)
2. Refresh DeviceScanner + TopologyScanner if stale (both run in parallel via `Promise.all`)
3. Build system prompt: static part (cached via `cache_control: ephemeral` for Anthropic, plain string for OpenAI) + dynamic part (facts + datetime + home layout + device cheat sheet)
3. Load conversation history from `IConversationStore` (keyed by conversationId)
4. Stream response with tool loop (parallel tool execution)
5. Fire-and-forget fact extraction (extracts facts, replaces conflicting old ones)
6. Return response to caller

### Two LLM Calls Per Request

- **Chat**: `IChatEngine` — handles conversation + HA tool calls. Implementations: `LLMClient` (Anthropic), `OpenAIChatEngine` (OpenAI/Ollama)
- **Extraction**: `IFactExtractor` — extracts facts from conversation (async, non-blocking). Implementations: `FactExtractor` (Anthropic), `OpenAIFactExtractor` (OpenAI/Ollama)

Provider is selected at startup by `llm/factory.ts` based on `LLM_PROVIDER` config.

### Memory Architecture

- **Long-term facts**: Shodh Memory (external service, semantic search, Hebbian learning, natural decay)
- **Conversation history**: `IConversationStore` with two backends: `InMemoryConversationStore` (default, lost on restart) and `SqliteConversationStore` (persistent via `better-sqlite3`). Controlled by `CONVERSATION_STORAGE` env var (`memory` | `sqlite`). Max 20 messages/conversation.
- **Entity cache**: 10-second TTL in HomeAssistantClient (invalidated after service calls)
- **Fact categories**: baseline, preference, identity, device, pattern, correction
- **Smart replacement**: Extractor identifies existing facts that new facts supersede (via `replaces` field)

## Development Commands

All commands run from `src/home-mind-server/`:

```bash
npm run dev          # tsx watch (hot reload), starts at localhost:3100
npm run build        # tsc → dist/
npm run typecheck    # tsc --noEmit
npm run lint         # eslint src/
npm test             # vitest run
npm run test:watch   # vitest (watch mode)
npm run test:coverage # vitest with v8 coverage

# Single test file
npm test -- src/memory/shodh-client.test.ts

# Single test by name
npm test -- -t "can check health"
```

Requires Shodh Memory running at SHODH_URL. For local dev: `cp .env.example .env` and fill in credentials.

## Testing Patterns

Vitest with `globals: true`. Tests mock constructors using `class` syntax in `vi.mock()` factories (not `vi.fn().mockImplementation()`) because the code uses `new`. Config tests use `vi.resetModules()` + dynamic `import()` to test different env var configurations.

Integration tests (e.g., `shodh-client.test.ts`) only run when `SHODH_TEST_URL` and `SHODH_TEST_API_KEY` are set — otherwise `describe.skip`.

## Code Patterns

**ES Modules with `.js` extensions** in TypeScript imports (required by `moduleResolution: "NodeNext"` in tsconfig):
```typescript
import { loadConfig } from "./config.js";
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoornet/home-mind](https://github.com/hoornet/home-mind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
