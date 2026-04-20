---
trigger: always_on
description: A cross-platform Rust TUI for managing Azure Service Bus (queues, topics, subscriptions, messages). Built with `ratatui` and direct REST API integration — **no official Azure SDK**.
---

# Service Bus Explorer TUI — Claude Instructions

## Project Overview

A cross-platform Rust TUI for managing Azure Service Bus (queues, topics, subscriptions, messages). Built with `ratatui` and direct REST API integration — **no official Azure SDK**.

## Architecture

### No Azure SDK
All client code in `src/client/` uses `reqwest` against the REST API directly. Management plane uses ATOM XML feeds parsed with targeted string extraction (not full serde XML). Data plane uses JSON/HTTP. Auth is HMAC-SHA256 SAS tokens or Azure AD Bearer tokens via `azure_identity`.

### Async + Sync Hybrid Event Loop
The main loop in `main.rs` polls `crossterm` events synchronously at 100ms intervals. Azure operations are dispatched as `tokio::spawn` tasks that communicate results back via an unbounded `mpsc` channel (`app.bg_tx` / `app.bg_rx`). The main loop drains `bg_rx.try_recv()` each tick.

### Status Sentinel Dispatch
Action dispatch in `run_app()` uses **string matching on `app.status_message`** to trigger async operations. `event.rs` sets the sentinel; `main.rs` matches it and spawns the task. Known sentinels:
- `"Peeking messages..."` → peek, `"Refreshing..."` → tree reload
- `"Submitting..."` → send/create/edit (disambiguated by `app.modal` or `app.detail_editing`)
- `"Deleting..."` → entity delete, `"Bulk resending..."` / `"Bulk deleting..."` → bulk ops
- `"Clearing (delete)..."` / `"Clearing (delete DLQ)..."` / `"Clearing (resend)..."` → purge/resend

When adding new operations, set a unique sentinel in `event.rs`, then match it in `main.rs`.

### ATOM XML Parsing
Azure returns inconsistent ATOM feed schemas. Parsing in `management.rs` uses raw string search (`extract_element`, `extract_element_value`, `extract_entries`) rather than serde XML deserialization. `CountDetails` child elements may be prefixed (`d2p1:ActiveMessageCount`) or unprefixed — the parser tries both.

## Module Structure

```
src/
├── main.rs              # Entry point, event loop, status-sentinel → async task dispatch
├── app.rs               # App state, BgEvent enum, form builders, tree construction
├── event.rs             # Input routing: global → modal → panel handlers
├── config.rs            # TOML persistence (connections, settings, OS-specific paths)
├── client/
│   ├── auth.rs          # SAS token gen, Azure AD token, connection string parsing
│   ├── management.rs    # Management plane: ATOM XML CRUD + raw XML parsing helpers
│   ├── data_plane.rs    # Data plane: send, peek-lock, receive-delete, purge, bulk ops
│   ├── models.rs        # Entity descriptions, message models, TreeNode/FlatNode
│   └── error.rs         # ServiceBusError (thiserror) with Api, Auth, Xml variants
└── ui/
    ├── layout.rs        # Top-level 3-panel layout (tree | detail | messages)
    ├── tree.rs          # Entity tree with inline message/DLQ counts
    ├── messages.rs      # Message list + detail view + inline edit rendering
    ├── modals.rs        # Connection, form, confirm, clear-options, peek-count dialogs
    ├── detail.rs        # Entity properties/runtime info panel + sparkline previews
    ├── metrics_detail.rs # Full-screen metrics overlay with braille line charts
    ├── status_bar.rs    # Bottom status bar
    ├── help.rs          # Full keyboard shortcut overlay (`?` key)
    └── sanitize.rs      # Terminal escape injection prevention (CSI/OSC stripping)
```

## Key Patterns

### Entity Path Handling
- Queues: `"queue-name"` — Topics: `"topic-name"` — Subscriptions: `"topic-name/Subscriptions/sub-name"`
- Management API uses `/Subscriptions/` (PascalCase); data plane requires `/subscriptions/` (lowercase). `DataPlaneClient::normalize_path()` handles conversion.
- DLQ: append `/$deadletterqueue` to any entity path
- **Send path**: subscriptions must route to parent topic — use `send_path()` in `main.rs` to strip `/Subscriptions/...`

### Topic Fan-out
Operations on a **Topic** entity automatically fan out across all its subscriptions:
- Peek DLQ on topic → peeks all subscription DLQs and merges results
- Purge/clear on topic → purges each subscription (or each subscription's DLQ)
- Bulk resend on topic → resends from each subscription's DLQ to the topic

This pattern appears in every topic-aware spawn block in `main.rs`. It calls `mgmt.list_subscriptions()` first, then iterates.

### Peek Implementation
The REST API's `PeekOnly=true` has no cursor (always returns the same first message). Peek is implemented as **peek-lock + abandon**: lock N messages sequentially, collect them, then abandon all locks. This makes messages available again but **increments `DeliveryCount`** on each peek.

### Input Routing Priority
`event.rs:handle_events()` routes keys in this order:
1. **Background running + Esc** → cancel operation
2. **Modal open** (`app.modal != None`) → `handle_modal_input()`
3. **Inline editing** (`app.detail_editing`) → `handle_detail_edit_input()` (Esc exits edit, all other keys go to field editor)
4. **Global keys** (q, Ctrl+C, ?, c, Tab)
5. **Panel-specific** handler based on `app.focus` (Tree/Detail/Messages)

### Form Editing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CosX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
