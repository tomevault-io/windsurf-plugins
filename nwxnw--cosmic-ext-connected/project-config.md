---
trigger: always_on
description: Guidance for Claude Code when working with cosmic-ext-connected.
---

# CLAUDE.md

Guidance for Claude Code when working with cosmic-ext-connected.

## Project Overview

Connected is a panel applet for the COSMIC™ desktop environment providing phone-to-desktop connectivity. It uses KDE Connect's daemon (`kdeconnectd`) as a D-Bus backend while providing a native libcosmic UI.

**Key Principle:** This project does NOT modify KDE Connect. It consumes kdeconnectd as a D-Bus service.

## Release Management

- **`main`** — sole development branch (no stable release branches yet)
- **`v0.1.0`** tag exists as a historical marker for the first GitHub release
- All work (features and fixes) goes to `main` directly or via `feature/*` branches
- A release branch will be created when the project is published to a Flatpak repository or package archive

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Connected Applet (Rust)                                        │
│  ├── cosmic-ext-connected/     (UI layer - libcosmic)          │
│  └── kdeconnect-dbus/          (D-Bus client - zbus)           │
└──────────────────────┬──────────────────────────────────────────┘
                       │ D-Bus (org.kde.kdeconnect.*)
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  kdeconnectd (system package: apt install kdeconnect)          │
└──────────────────────┬──────────────────────────────────────────┘
                       │ TCP/UDP/Bluetooth
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│  Android Phone (KDE Connect app)                                │
└─────────────────────────────────────────────────────────────────┘
```

## Build Commands

```bash
cargo build                              # Build all crates
cargo build --release                    # Build release
cargo run -p cosmic-ext-connected        # Run (requires COSMIC)
cargo test && cargo clippy               # Test and lint
just install                             # Install to system
just uninstall                           # Uninstall
```

**Development cycle:** `cargo build --release && sudo just install && killall cosmic-panel`

**Flatpak build:**
```bash
flatpak-builder --user --install --force-clean build-dir io.github.nwxnw.cosmic-ext-connected.json
gtk-update-icon-cache -f ~/.local/share/flatpak/exports/share/icons/hicolor/  # Force icon cache refresh
killall cosmic-panel                     # Reload panel
```

**Flatpak sandbox permissions** (in `finish-args` of manifest):
- `--filesystem=xdg-config/cosmic:rw` — read/write COSMIC config
- `--filesystem=xdg-data/kpeoplevcard:ro` — read contacts for SMS name resolution
- `--filesystem=xdg-cache/kdeconnect.daemon:ro` — read MMS attachment cache (daemon uses Qt app name `"kdeconnect.daemon"` → cache at `~/.cache/kdeconnect.daemon/`)

**Debug logs:** `journalctl --user SYSLOG_IDENTIFIER=cosmic-ext-connected -f` (logs land here directly via `tracing_journald`; see `cosmic-applets/Logging and Diagnostics in COSMIC Applets.md` in the Auxis vault for routing details)

## Project Structure

```
cosmic-ext-connected/
├── cosmic-ext-connected/src/
│   ├── app.rs              # Core: ConnectApplet, Message enum, update()
│   ├── config.rs           # User preferences (cosmic_config)
│   ├── notifications.rs    # Cross-process notification deduplication
│   ├── subscriptions.rs    # D-Bus signal subscriptions
│   ├── device/             # Device fetch and actions
│   ├── sms/                # SMS conversations, views, subscriptions
│   │   ├── send.rs                       # SMS sending (replyToConversation for replies, sendWithoutConversation for new messages)
│   │   ├── conversation_subscription.rs  # Dual D-Bus request + incremental conversation loading
│   │   ├── fetch.rs                      # Conversation fetching and caching
│   │   └── views.rs                      # SMS conversation list and thread views
│   ├── media/              # Media player controls
│   └── views/              # Shared UI components
│
├── data/
│   ├── io.github.nwxnw.cosmic-ext-connected.desktop
│   ├── io.github.nwxnw.cosmic-ext-connected.metainfo.xml
│   └── icons/hicolor/scalable/apps/
│       ├── io.github.nwxnw.cosmic-ext-connected.svg                          # App icon (128x128, #BEBEBE fill)
│       ├── io.github.nwxnw.cosmic-ext-connected-symbolic.svg                  # Panel: connected state
│       └── io.github.nwxnw.cosmic-ext-connected-disconnected-symbolic.svg     # Panel: disconnected state
│
├── io.github.nwxnw.cosmic-ext-connected.json  # Flatpak manifest
│
├── kdeconnect-dbus/src/
│   ├── daemon.rs           # org.kde.kdeconnect.daemon proxy
│   ├── device.rs           # Device interface proxy
│   ├── contacts.rs         # ContactLookup: vCard parsing, name resolution, group display names
│   └── plugins/            # Per-plugin D-Bus proxies
│
└── docs/                   # Detailed implementation docs
    ├── DBUS.md             # D-Bus interface reference
    ├── SMS.md              # SMS implementation details
    ├── NOTIFICATIONS.md    # Notification systems
    ├── MEDIA.md            # Media controls
    ├── UI_PATTERNS.md      # libcosmic UI patterns
    ├── LOGGING.md          # Tracing/journald routing for diagnostics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nwxnw/cosmic-ext-connected](https://github.com/nwxnw/cosmic-ext-connected) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
