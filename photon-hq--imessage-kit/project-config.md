---
trigger: always_on
description: > Context for AI assistants working on this codebase.
---

# CLAUDE.md

> Context for AI assistants working on this codebase.

## Project

`@photon-ai/imessage-kit` — Type-safe macOS iMessage SDK (TypeScript). Reads from `~/Library/Messages/chat.db` via SQLite, sends via AppleScript.

## Commands

```bash
bun test              # Run all tests
npx tsc --noEmit      # Type-check
npx biome check --write src/  # Format + lint
npm run build         # Build via tsup → dist/
```

## Architecture

```
src/
├── index.ts                    # Public API barrel
├── sdk.ts                      # SDK class — composition root
├── sdk-bounds.ts               # Runtime config bounds (standalone, zero deps)
├── domain/                     # Pure business logic — zero I/O, zero external deps
│   ├── attachment.ts           # Attachment interface + TransferStatus
│   ├── chat.ts                 # Chat interface + ChatKind + style constants
│   ├── chat-id.ts              # ChatId value object (parsing, normalization, matching)
│   ├── errors.ts               # IMessageError class + named factories
│   ├── message.ts              # Message interface + enums (Kind, Expire, Share, Schedule)
│   ├── messages-app.ts         # macOS/Messages.app protocol facts (TCC safe dirs, temp prefix)
│   ├── reaction.ts             # Reaction interface + ReactionKind + resolveReactionMeta
│   ├── routing.ts              # MessageTarget + resolveTarget (DM vs group routing)
│   ├── service.ts              # Service type + resolveService
│   ├── timestamp.ts            # MAC_EPOCH + timestamp conversion
│   └── validate.ts             # URL + content validation
├── application/                # Application orchestration — depends on domain + types only
│   ├── send-port.ts            # SendPort interface (implemented by SDK + Sender)
│   └── message-dispatcher.ts   # Incoming event routing (watch → callbacks + plugins)
├── infra/                      # External system adapters
│   ├── platform.ts             # Platform detection, default paths, Darwin version
│   ├── attachments.ts          # Read-only file ops on existing attachments
│   ├── db/                     # SQLite read + watch
│   │   ├── sqlite-adapter.ts   # Runtime-agnostic SQLite (bun:sqlite / better-sqlite3)
│   │   ├── contract.ts         # Query contract + ChatId SQL match helper
│   │   ├── macos26.ts          # macOS 26 query builder (MESSAGE/CHAT/ATTACHMENT fields)
│   │   ├── mapper.ts           # Row → Message/Chat/Attachment conversion
│   │   ├── reader.ts           # High-level database reader facade
│   │   ├── body-decoder.ts     # attributedBody BLOB decoding
│   │   └── watcher.ts          # WAL-based real-time message monitor
│   ├── outgoing/               # Send pipeline
│   │   ├── sender.ts           # Send orchestrator (buddy vs chat method)
│   │   ├── applescript-transport.ts  # AppleScript generation + stdin execution
│   │   └── temp-files.ts       # Sandbox-bypass mktemp file cleanup (~/Pictures/imsg_temp_*)
│   └── plugin.ts               # Plugin lifecycle + hook dispatch (interrupting / sequential / parallel)
├── utils/                      # Shared pure utilities (importable by any layer)
│   └── async.ts                # delay, retry, Semaphore
└── types/                      # Type definitions only — no logic
    ├── config.ts               # IMessageConfig
    ├── query.ts                # MessageQuery, ChatQuery
    ├── send.ts                 # SendRequest
    └── plugin.ts               # Plugin, PluginHooks, hook contexts
```

## Layer Dependency Rules

Enforced by `__tests__/25-architecture-boundaries.test.ts`:

| Layer | May import from |
|-------|----------------|
| `types/` | `types/`, `domain/` types only |
| `domain/` | `domain/`, `types/` |
| `application/` | `application/`, `domain/`, `types/` |
| `infra/` | `infra/`, `domain/`, `types/`, `utils/`, `application/send-port.ts`, `application/message-dispatcher.ts` |
| `utils/` | nothing (pure, zero deps) |
| `sdk.ts` | everything except `index.ts` |
| `sdk-bounds.ts` | nothing |
| `index.ts` | anything (public API barrel) |

## Code Style

- Biome: 4-space indent, single quotes, trailing commas, semicolons as needed, 120 line width
- Section headers: `// -----------------------------------------------`
- Errors: `SendError(msg)` returns `IMessageError` (not `new SendError()`). Use `instanceof IMessageError` in catch; prefer factories over `new IMessageError` so `code` matches intent. Exception: when re-throwing an `IMessageError` with added context, use `new IMessageError(upstream.code, msg, { cause: upstream })` so the original `code` is preserved (see `sender.ts` attachment-precheck re-throw).
- 1 production dependency: `@parseaple/typedstream` (for attributedBody BLOB parsing)
- Dual runtime: `bun:sqlite` (Bun) / `better-sqlite3` (Node.js)

## Key Patterns

- **ChatId value object** (`domain/chat-id.ts`): All chatId parsing/normalization in one place. Supports `any;+;guid` (macOS 26+), `iMessage;+;chatGUID` (legacy), `service;-;address` (DM).
- **Port/Adapter**: `application/send-port.ts` defines `SendPort` (`send(request): Promise<void>`); `infra/outgoing/sender.ts` implements it. `send()` resolves on AppleScript dispatch only — to observe the chat.db row, subscribe via the watcher (see Naming Quirk below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [photon-hq/imessage-kit](https://github.com/photon-hq/imessage-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
