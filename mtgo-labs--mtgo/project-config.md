---
trigger: always_on
description: Build Telegram bots and userbots in Go using mtgo — a fast, idiomatic MTProto client. Use for any Telegram-related Go project — bots with inline keyboards and callbacks, userbots acting on behalf of a user account, session management, media upload/download, authentication via bot token or phone number or QR code or session strings, group creation and management, middleware chains, plugins, i18n, MTProxy, business connections, paid media, secret chats, gifts, live broadcasting, and multi-client s
---


# mtgo — Telegram MTProto Client for Go

mtgo is a Go library for building Telegram bots and userbots using the MTProto 2.0 protocol. It provides a high-level client API with handlers, filters, middleware, plugins, and storage backends.

## Quick Reference

**Module:** `github.com/mtgo-labs/mtgo`
**API Reference:** <https://pkg.go.dev/github.com/mtgo-labs/mtgo>

Use `go doc` to look up types and methods:
```
go doc github.com/mtgo-labs/mtgo/telegram Client
go doc github.com/mtgo-labs/mtgo/telegram Filter
go doc github.com/mtgo-labs/mtgo/telegram/params SendMessage
```

**Key packages:**
- `telegram` — high-level client, handlers, filters, middleware, keyboards
- `telegram/types` — Message, User, Chat, CallbackQuery, media types
- `telegram/params` — SendMessage, SendPhoto, Download, ProgressInfo, entities
- `tg` — generated TL types and RPC methods (low-level)
- `tgerr` — generated error types and error constants
- `session` — session string import/export (Telethon, Pyrogram, GramJS, mtcute)

**Ecosystem packages:**
- `github.com/mtgo-labs/storage/sqlite` — SQLite storage
- `github.com/mtgo-labs/storage/postgres` — PostgreSQL storage
- `github.com/mtgo-labs/storage/mongodb` — MongoDB storage
- `github.com/mtgo-labs/storage` — storage.NewAdapter() wrapper
- `github.com/mtgo-labs/plugins/conversations` — conversation/state machine plugin
- `github.com/mtgo-labs/plugins/i18n` — internationalization plugin
- `github.com/mtgo-labs/middlewares/floodwait` — flood wait auto-retry middleware
- `github.com/mtgo-labs/middlewares/ratelimit` — rate limiting middleware

For advanced topics (full Config reference, userbot auth, group management, BotFather, testing), see `references/advanced.md`.
For newer features (business connections, secret chats, gifts, live broadcasting, TDLib JSON, account privacy, lifecycle handlers), see `references/new-features.md`.

## Client Creation

```go
import "github.com/mtgo-labs/mtgo/telegram"

// Bot with token
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    BotToken:    os.Getenv("BOT_TOKEN"),
    SessionName: "my_bot",
    SavePeers:   true,
})

// Bot with in-memory session
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    BotToken:    botToken,
    SessionName: "my_bot",
    InMemory:    true,
    SavePeers:   true,
    ParseMode:   telegram.HTML,
})

// Userbot (phone number) — terminal prompts for code/password
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    PhoneNumber: "+1234567890",
    SessionName: "my_userbot",
})

// Session string import (auto-detects format)
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    SessionString: sessionStr,
    InMemory:      true,
    SavePeers:     true,
})

// With storage backend
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    BotToken:    botToken,
    SessionName: "my_bot",
    Storage:     sqlite.New(),
})

// Auto-connect on first use (no explicit Connect call needed)
client, err := telegram.NewClient(apiID, apiHash, &telegram.Config{
    BotToken:     botToken,
    SessionName:  "my_bot",
    AutoConnect:  true,
})
```

The `apiID` is `int32` and `apiHash` is `string`, obtained from https://my.telegram.org. The `NewClient` signature is `NewClient(apiID int32, apiHash string, cfg *Config) (*Client, error)`.

### Common Config fields

| Field | Type | Purpose |
|---|---|---|
| `BotToken` | `string` | Bot authentication |
| `PhoneNumber` | `string` | Userbot authentication |
| `SessionString` | `string` | Import existing session |
| `SessionName` | `string` | Session identifier |
| `InMemory` | `bool` | No session file on disk |
| `SavePeers` | `bool` | Cache peer info |
| `ParseMode` | `params.ParseMode` | Default parse mode (HTML/MarkdownV2) |
| `Storage` | `storage.Storage` | Storage backend |
| `AutoConnect` | `bool` | Lazy connect on first RPC/handler registration |
| `NoUpdates` | `bool` | Skip receiving updates |
| `MTProxy` | `*MTProxyConfig` | MTProxy config |
| `WebSocket` | `bool` | MTProto over WebSocket |
| `Device` | `DeviceConfig` | Device identity (model, version, lang) |
| `ReconnectEnabled` | `bool` | Auto-reconnect (default true) |
| `Retries` | `int` | RPC retry count |
| `ReqTimeout` | `time.Duration` | Default RPC timeout (60s) |

For the full Config reference (50+ fields including reconnect, health, dispatch, update recovery), see `references/advanced.md`.

## Lifecycle

Three ways to run a client:

```go
// Option 1: Start() — Connect + Idle in one call (simplest)
client.Start() // blocks until Stop()
// Note: Start() returns error, check it:
if err := client.Start(); err != nil {
    log.Fatal(err)
}

// Option 2: Connect + Idle (manual control)
client.Connect(0) // 0 = auto-detect nearest DC
defer client.Stop()
client.Idle() // blocks until Stop()


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtgo-labs/mtgo](https://github.com/mtgo-labs/mtgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
