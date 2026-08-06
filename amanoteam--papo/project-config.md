---
trigger: always_on
description: **Papo** is an unofficial GTK4/libadwaita WhatsApp client written in Rust using [Relm4](https://relm4.org/) 0.11. It targets GNOME 49 (libadwaita 1.8+, GTK 4.20+).
---

# Agent Guidelines for Papo

**Papo** is an unofficial GTK4/libadwaita WhatsApp client written in Rust using [Relm4](https://relm4.org/) 0.11. It targets GNOME 49 (libadwaita 1.8+, GTK 4.20+).

## Architecture

- **Framework**: Relm4 with async components (`AsyncComponent`). UI is declared in `view!` macros.
- **Pattern**: Components communicate via typed `Input`/`Output` messages. The root `Application` component orchestrates child components (`Login`, `ChatList`, `ChatView`, `Client`).
- **Async**: Heavy work (database, network) must be offloaded via `sender.oneshot_command()` or `relm4::spawn()`. Never block the main thread.
- **State**: Shared state uses `Arc<Database>` and `Arc<RuntimeCache>`. Chat/message state lives in `src/state/`.
- **Database**: libsql (with encryption feature). Migrations and queries in `src/store/`.
- **WhatsApp API**: `wacore`/`waproto`/`whatsapp-rust` crates (from crates.io, v0.5).

## Project Structure

```
src/
├── main.rs                  # Entry point: logger, i18n, resource loading, RelmApp init
├── application.rs           # Root AsyncComponent: orchestrates pages, state machine, action wiring
├── config.rs / config.rs.in # Build-time constants (APP_ID, VERSION, PROFILE, paths)
├── utils.rs                 # Shared helpers: QR generation, date formatting, phone number parsing
│
├── components/              # Relm4 UI components (AsyncComponent/SimpleAsyncComponent)
│   ├── mod.rs               # Re-exports ChatList, ChatView, Login and their I/O types
│   ├── chat_list.rs         # Sidebar chat list with AdwToggleGroup filters, TypedListView rows
│   ├── chat_view.rs         # Chat history with bidirectional infinite scroll, message input, read receipts
│   └── login.rs             # QR-code + phone-number pairing flow, pair-code cells
│
├── modals/                  # SimpleComponent dialogs launched from Application actions
│   ├── mod.rs
│   ├── about.rs             # AdwAboutDialog with app metadata
│   └── shortcuts.rs         # AdwShortcutsDialog with keyboard shortcuts
│
├── session/                 # WhatsApp client runtime and caches
│   ├── mod.rs
│   ├── client.rs            # AsyncComponent wrapping whatsapp-rust Client (connection, sync, events)
│   └── cache.rs             # AvatarCache (disk) and RuntimeCache (in-memory Moka caches)
│
├── state/                   # Plain data models (no UI logic)
│   ├── mod.rs               # Re-exports Chat, ChatMessage, Media, MessageStatus
│   ├── chat.rs              # Chat struct with DB save/load, participants, unread helpers
│   ├── message.rs           # Message struct with status enum (Sent/Read/Delivered/etc.), reactions
│   └── media.rs             # Media attachment with download info, MIME type, dimensions
│
├── store/                   # Database layer (libsql with encryption)
│   ├── mod.rs               # Re-exports Database, Contact
│   └── database.rs          # Schema creation, CRUD for chats/messages/contacts, search queries
│
└── widgets/                 # Custom GTK widgets reused in components
    ├── mod.rs               # Re-exports PairStep, PairingCell
    ├── pair_step.rs         # Single character cell for phone-number pair code
    └── pairing_cell.rs      # Character display widget for pair code grid

data/
├── com.amanoteam.Papo.desktop.in.in      # Desktop entry template
├── com.amanoteam.Papo.metainfo.xml.in.in # AppStream metadata template
├── com.amanoteam.Papo.gschema.xml.in     # GSettings schema (preferences)
├── com.amanoteam.Papo.service.in         # D-Bus service file
├── icons/                                # App icon (SVG, symbolic)
└── resources/
    ├── style.scss                        # Compiled SCSS → CSS gresource
    └── stylesheet/                       # Partial SCSS files (chat_list, chat_view, login, etc.)

po/                        # Gettext translations (pt_BR.po, POTFILES.in, LINGUAS)
build-aux/                 # Flatpak manifests (JSON) and dist-vendor script
```

## Code Quality — Non-Negotiable

The project enforces **extremely strict** linting. Code will fail CI if these are violated.

- **Clippy**: `#![deny(clippy::all)]`, `#![deny(clippy::cargo)]`, `#![deny(clippy::pedantic)]`, plus specific denied lints (see `src/main.rs` lines 5–21). **Do not suppress with `#![allow(...)]` or `#[allow(...)]` unless specifically marked as a FIXME/WIP.**
- **No `println!`/`eprintln!`**: Use the `tracing` crate (`tracing::info!`, `tracing::error!`, etc.). These are denied by clippy.
- **No dead code**: `#![allow(dead_code)]` exists temporarily for WIP features. Do not add new dead code.
- **Formatting**: `rustfmt`. A pre-commit hook is installed in development builds.
- **Rust edition**: 2024. Requires Rust nightly for certain dependencies.

## Internationalization (i18n)

**Every user-facing string must be translatable.**

- Use the macros: `i18n!("Text")`, `i18n_f!("Hello {0}", name)`, `ni18n!("1 item", "{n} items", count)`.
- Do not hardcode English strings in UI code.
- Translations live in `po/`. Brazilian Portuguese is currently supported.

## UI / HIG Rules

- **Container**: `adw::ApplicationWindow` + `adw::HeaderBar` for main windows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmanoTeam/Papo](https://github.com/AmanoTeam/Papo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
