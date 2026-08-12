---
trigger: always_on
description: `@neomei/opencode-feishu` — bridges an OpenCode AI server to Feishu/Lark messaging. Runs standalone (`opencode-feishu start`) or as an OpenCode plugin (`plugins: ["@neomei/opencode-feishu"]`). TypeScript, ESM-only (`"type": "module"`).
---

# AGENTS.md

## Project

`@neomei/opencode-feishu` — bridges an OpenCode AI server to Feishu/Lark messaging. Runs standalone (`opencode-feishu start`) or as an OpenCode plugin (`plugins: ["@neomei/opencode-feishu"]`). TypeScript, ESM-only (`"type": "module"`).

## Commands

```bash
npm run build        # tsc → dist/
npm run dev          # tsc --watch
npm test             # Jest (ts-jest, ESM preset) — single file at test/integration.test.ts
npm run typecheck    # tsc --noEmit
npm run clean        # rm -rf dist
npx jest test/integration.test.ts   # single test
```

`npm run lint` is defined but **will fail** — ESLint is not installed as a devDependency. Do not rely on it.

Runtime entry is `bin/opencode-feishu` → `dist/cli.js`; must build first.

## Architecture

Two entrypoints share the same core wiring. A **single** `FeishuAPI` and `OpenCodeClient` instance is threaded through `SessionManager`, `MessageHandler`, `FeishuEventSource`, and `OpenCodeEventHandler`.

- **Standalone** — `src/cli.ts` → `src/standalone.ts::startStandalone()`. Writes PID to `~/.config/opencode/feishu.pid`. Supports `--daemon`. Reads `OPENCODE_SERVER_PASSWORD` for Basic auth.
- **Plugin** — `src/plugin.ts`. `server()` hook receives host OpenCode `client` + `project` + `directory`.

### Message flow

```
Feishu event (Lark.WSClient + EventDispatcher, autoReconnect)
  → FeishuEventSource (src/feishu/event-source.ts)
  → MessageHandler (dedup / mention / allowlist / group-policy)
  → SessionManager (1 session per chat_id; persisted)
  → OpenCodeClient.sendPrompt / sendCommand

OpenCode event stream
  → OpenCodeEventHandler (src/opencode/event-handler.ts)
  → flushCard() → FeishuAPI sendCard / updateCard (one card per turn, ~0.5 Hz throttle)
```

### Streaming card model

One Feishu interactive card per turn. First `flushCard` creates via `sendCard` (stores `message_id`); subsequent calls PATCH via `updateCard`. `session.idle` flips header to "✅ 完成".

### Card display modes (`showProcess` config)

`'none' | 'tools' | 'thinking' | 'full'`, default `'none'` (quiet mode):

- **`'none'`**: Only final text shown. Thinking animation card sent first, replaced on content arrival. `appendContent` tracks `partID` and resets on new part so only last text part is visible.
- **`'tools'`**: Text + live tool execution list with status icons.
- **`'thinking'`**: Text + reasoning process (inline while streaming, grey/collapsed when done).
- **`'full'`**: Everything — thinking, tools, text.

### OpenCode interactive events

`OpenCodeEventHandler` handles: `message.part.delta`, `message.part.updated`, `session.status`, `session.error`, `session.idle`, `permission.asked/updated/replied`, `question.asked/replied/rejected`, `command.executed`. Pending interactions route next user message through `handleInteractionReply()` before normal processing.

### Slash commands

`MessageHandler.parseSlashCommand()` routes `/`-prefixed messages to `sendCommand()` instead of `sendPrompt()`. Only a whitelist of commands is allowed; unknown commands fall through to `sendPrompt`. Whitelist defined at `src/core/message-handler.ts` ~line 279.

### Session persistence

`SessionManager` persists `chat_id → session_id` mappings to `~/.config/opencode/feishu-sessions.json` (debounced 500 ms, atomic write via tmp+rename). On startup, reconciles against OpenCode (`sessionExists`); stale mappings are dropped.

## Configuration

- **Path**: `~/.config/opencode/feishu.json` (override with `-c`).
- **Multi-profile**: `~/.config/opencode/feishu-profiles/`. Active profile at `~/.config/opencode/feishu-active-profile`.
- **Schema**: Zod in `src/core/config.ts`. `appId` must start with `cli_`. `appSecret` required (config or `FEISHU_APP_SECRET` env var). Resolution: `resolveAppSecret()`.
- **Key fields**: `domain` (`feishu|lark`), `opencodeUrl`, `streaming`, `requireMention`, `groupPolicy` (`allowlist`), `showProcess`, `allowlist`, `workdir`, `hooks`.

## CLI Commands

```
opencode-feishu setup [-c <path>]
opencode-feishu start [-c <path>] [-u <url>] [--daemon|--serve]
opencode-feishu status [--json]
opencode-feishu stop
opencode-feishu doctor [-c <path>] [--json]
opencode-feishu logs [-n <n>] [-f] [--json]
opencode-feishu profile list|add|use|delete|rename|clone|show
```

## Service Layer

Domain services under `src/services/` all extend `BaseService` (`call()` + `validateRequired()`). Untyped SDK calls go through `this.api.getClient().request()`:

| Service | Purpose |
|---------|---------|
| `IMService` | Send/reply/search messages, download resources |
| `DocService` | Fetch/search/convert docs, upload/create documents |
| `ChatService` | Search/create/query chats, manage members |
| `ContactService` | Search users, get department info |
| `CalendarService` | List calendars/events, create/update/delete, free-busy |
| `TaskService` | List/get/create/update/complete/delete tasks |
| `ApprovalService` | List/get/approve/reject/transfer instances |

## Critical Quirks

These are easy to get wrong and cause subtle bugs:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeoMei/opencode-feishu](https://github.com/NeoMei/opencode-feishu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
