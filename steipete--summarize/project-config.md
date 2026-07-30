---
trigger: always_on
description: Summarize can run as a **website automation agent** inside the Chrome side panel. This is **optional** and **gated by a checkbox** in Options.
---


# Automation Agent (Side Panel + Daemon)

Summarize can run as a **website automation agent** inside the Chrome side panel. This is **optional** and **gated by a checkbox** in Options.

Scope:

- **Off (default):** chat is Q&A only (no tools).
- **On:** chat runs a tool-capable agent for web automation.

Explicit exclusions (per product direction): **no update checker**, **no tutorial/welcome flow**, **no proxy config**, **no API key dialog**. All model calls go through the **local daemon**.

## Architecture (High Level)

1. **Side panel** maintains the agent loop + chat UI.
2. **Background** handles tab data, extraction, and tool execution.
3. **Content scripts** handle element picking and native-input bridge.
4. **Daemon** provides `/v1/agent` (SSE stream of chunks + final assistant message).

### Data Flow (Agent Loop)

- User sends a message in the side panel.
- Panel compacts history and sends `panel:agent` to background with:
  - `messages` (pi-ai Message[])
  - `tools` (names)
  - `summary` (optional current summary markdown)
- Background builds `pageContent` using the latest extract (summary + transcript/text + metadata).
- Background calls daemon `POST /v1/agent` (SSE).
- Daemon streams `chunk` events, then an `assistant` event (may include tool calls).
- Panel executes tool calls locally, appends `toolResult` messages, and repeats `/v1/agent` until no tool calls remain.

The daemon **never** executes tools. It only returns the next assistant message.

## Settings + Permissions

### Settings

`automationEnabled` (boolean) lives in `apps/chrome-extension/src/lib/settings.ts`.

- Options UI provides the toggle.
- When disabled, the tool list is empty and the daemon uses the **chat-only** prompt.

### Automation Permissions

Defined in `apps/chrome-extension/wxt.config.ts`:

- `userScripts` – optional; requested via Options for `browserjs()` main-world script execution.
- `debugger` – required only in `build:automation` for **native input** and the **debugger** tool.

Neither permission is needed for summarization. The standard Chrome build omits `debugger` because
[Chrome does not allow it to be optional](https://developer.chrome.com/docs/extensions/reference/api/permissions#optional_declarations),
and it hides debugger-backed tools. The separate automation build declares it as required.

#### Chrome: enable User Scripts (if needed)

1. `chrome://extensions`
2. Open extension details
3. Enable **Allow User Scripts**
4. Reload the tab

## Daemon Endpoint

### `POST /v1/agent` (SSE)

**Headers**

- `Authorization: Bearer <token>`
- `Content-Type: application/json`

**Body**

```
{
  "url": "https://...",
  "title": "Page title",
  "pageContent": "<summary + transcript + metadata>",
  "cacheContent": "<transcript/text used for cache key>",
  "messages": [/* pi-ai Message[] */],
  "model": "auto" | "openai/..." | "anthropic/..." | ...,
  "length": "short" | "xl" | "20k" | ...,
  "language": "auto" | "en" | "de" | ...,
  "tools": ["navigate", "repl", "ask_user_which_element", "skill", "debugger"],
  "automationEnabled": true
}
```

**Response (SSE, default)**

```
event: chunk
data: { "text": "..." }

event: assistant
data: { /* AssistantMessage */ }

event: done
data: {}

event: error
data: { "message": "..." }
```

**Response (JSON)**
Use `Accept: application/json` or `?format=json`.

```
{ "ok": true, "assistant": { /* AssistantMessage */ } }
```

### `POST /v1/agent/history`

Returns cached chat history for the same cache key as `/v1/agent`.

**Body**

```
{
  "url": "https://...",
  "pageContent": "<summary + transcript + metadata>",
  "cacheContent": "<transcript/text used for cache key>",
  "model": "auto" | "openai/..." | "anthropic/..." | ...,
  "length": "short" | "xl" | "20k" | ...,
  "language": "auto" | "en" | "de" | ...,
  "automationEnabled": true
}
```

**Response**

```
{ "ok": true, "messages": [/* Message[] */] }
```

### Model Resolution (Daemon)

- **Fixed model** (explicit `model`): parsed as `<provider>/<model>`. Provider base URL overrides come from config/env (OpenAI, Anthropic, Google, xAI, ZAI). OpenRouter uses OpenAI-compatible completions.
- **Auto model**: uses existing auto-selection logic (`buildAutoModelAttempts`), preferring API-key transports and then CLI fallback when available.
- **Synthetic models**: created for OpenAI-compatible base URLs (local/openrouter).
- `maxOutputTokens` defaults to 2048 or `maxOutputTokens` override.
- CLI models are supported as auto fallback and via explicit `cli/<provider>` or `cli/<provider>/<model>` overrides.
- If the daemon still says no model is available after key/install changes, restart or reinstall it so the saved environment snapshot refreshes.

## Page Content Payload

`pageContent` is built from the latest extract and includes:

- **Summary** (optional): current summary markdown (truncated to settings cap).
- **Transcript/text**: timed transcript when available, otherwise extracted text.
- **Metadata**: URL/title, source (`page` vs `url`), extraction strategy, markdown provider, Firecrawl usage, transcript provider + cache status, media duration, word counts, truncation flags.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/summarize](https://github.com/steipete/summarize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
