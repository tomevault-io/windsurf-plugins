---
trigger: always_on
description: A modern, native desktop mail client built in Rust that stands out through deep **Nextcloud integration** — targeting both businesses and end users. The goal is to be more appealing and capable than existing alternatives by combining standard email protocols with modern APIs and tight collaboration features.
---

# Unkai Mail

## Vision

A modern, native desktop mail client built in Rust that stands out through deep **Nextcloud integration** — targeting both businesses and end users. The goal is to be more appealing and capable than existing alternatives by combining standard email protocols with modern APIs and tight collaboration features.

## Key Differentiators

- **Nextcloud Talk integration** — create and join Talk rooms directly from the mail client (similar to Teams integration in Outlook)
- **Nextcloud Files integration** — attach, share, and browse files from Nextcloud directly within the client
- **Contact & Calendar sync** — full sync with Nextcloud Contacts and Calendar
- **Modern protocol support** — JMAP and direct API calls alongside traditional protocols

## Tech Stack

- **Language:** Rust (core logic, protocol handling, backend)
- **UI Framework:** Tauri 2 (native desktop app with Rust backend + system webview for UI)
- **Frontend:** Svelte 5 + TypeScript + Vite
- **UI Library:** Skeleton UI v3 (Tailwind CSS-based component library, theme: cerberus)
- **Platform targets:** Windows, macOS, Linux

## Project Structure

```
unkai-mail/
├── Cargo.toml              # Workspace root
├── crates/
│   ├── unkai-core/        # Shared types, models, error handling
│   ├── unkai-imap/        # IMAP mail retrieval
│   ├── unkai-smtp/        # SMTP mail sending
│   ├── unkai-jmap/        # JMAP modern mail access
│   ├── unkai-caldav/      # CalDAV calendar sync
│   ├── unkai-carddav/     # CardDAV contact sync
│   ├── unkai-nextcloud/   # Nextcloud API (Talk, Files, OCS)
│   ├── unkai-store/       # Local storage, caching, keychain
│   ├── unkai-discovery/   # Account autoconfiguration (SRV, autoconfig)
│   ├── unkai-crypto/      # OpenPGP + S/MIME primitives
│   ├── unkai-mcp/         # Local MCP server (#438)
│   └── unkai-commands/    # Transport-agnostic application layer (#476)
├── src-tauri/              # Tauri desktop shell (command shims + chrome)
└── ui/                     # Frontend (Svelte 5 + TypeScript + Vite)
    ├── src/
    │   ├── lib/            # Svelte components
    │   ├── app.css         # Global styles (Tailwind + Skeleton)
    │   ├── App.svelte      # Root component
    │   └── main.ts         # Entry point
    └── public/             # Static assets
```

## Protocols & Integrations

| Protocol/API | Purpose | Crate |
|---|---|---|
| IMAP | Mail retrieval | `unkai-imap` |
| SMTP | Mail sending | `unkai-smtp` |
| JMAP | Modern mail access (where supported) | `unkai-jmap` |
| CalDAV | Calendar sync (Nextcloud + others) | `unkai-caldav` |
| CardDAV | Contact sync (Nextcloud + others) | `unkai-carddav` |
| Nextcloud OCS/API | Talk rooms, file sharing, app integrations | `unkai-nextcloud` |

## Architecture Principles

- **Separation of concerns** — Rust core library handles all protocol/business logic; UI layer is a thin presentation layer
- **Offline-first** — local caching and sync so the client works without constant connectivity
- **Security-first** — TLS everywhere, credential storage via OS keychain, no plaintext secrets
- **Modular design** — each protocol as its own crate for testability and reuse

## Frontend ↔ backend IPC: the `api/` layer (#473)

All backend IPC in the frontend goes through the typed layer in [`ui/src/lib/api/`](ui/src/lib/api/) — **never import `@tauri-apps/*` directly in a component.** A vitest guard (`ui/src/lib/api/noDirectIpc.test.ts`) fails the build on violations.

- **Commands**: one typed wrapper per `#[tauri::command]`, grouped into domain modules (`api/mail`, `api/compose`, `api/accounts`, `api/contacts`, `api/calendar`, `api/nextcloud`, `api/talk`, `api/notes`, `api/tasks`, `api/crypto`, `api/settings`, `api/system`), all funnelling through `call()` in `api/core.ts`. Components do `import * as api from './api'` and call `api.mail.fetchEnvelopes({ accountId, folder, limit })`. **When you add/rename/change a Rust command, update its wrapper in the matching domain module in the same PR** — that's the point of the layer: the compiler finds every affected call site.
- **Events**: every event name (backend push channels + popout↔main handoffs) is registered in `AppEventPayloads` in `api/events.ts`. Subscribe with `api.onAppEvent('new-mail', handler)` (handler gets the Tauri `Event`, payload under `.payload`), emit with `api.emitAppEvent(...)`. Adding a new event = adding it to the registry first.
- **Platform affordances** (native dialogs, plugin notifications, autostart, `convertFileSrc` asset URLs) live in `api/platform.ts` — this file is the canonical list of desktop-only surface.
- **DTO types**: `api/types.ts` holds placeholder `any` aliases for backend DTOs. Tightening them is **lazy**, like the i18n migration: replace an alias with a real interface whenever you touch code that consumes it; don't open a bulk-typing PR.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firn-labs/unkai-mail](https://github.com/firn-labs/unkai-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
