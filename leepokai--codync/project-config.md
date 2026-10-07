---
trigger: always_on
description: Bot-based remote for coding agents: persistent named bots on your computer, messaged from the iPhone (UI patterns from Grok Bot).
---

# Codync

Bot-based remote for coding agents: persistent named bots on your computer, messaged from the iPhone (UI patterns from Grok Bot).

## Language & Syntax

- **Swift 6** strict concurrency mode — use latest Swift 6 syntax throughout
- Prefer SwiftUI lifecycle and modern APIs (`@Observable`, `@State`, `@Environment`)
- Use structured concurrency (`async/await`, `TaskGroup`) over Combine
- Use `sending`, `nonisolated`, `@MainActor` correctly per Swift 6 rules
- Avoid `@unchecked Sendable` — prefer proper `Sendable` conformance
- Desktop app (`apps/desktop/`) is Electron + React + strict TypeScript: state in `Observable` models (`src/renderer/store/`), host calls through the `window.codync` bridge (`src/shared/ipc.ts`), main-process code in `src/main/`; CI runs `npm run typecheck` and `npm test`
- Host is Rust 2024 edition and follows the `rust-skills` rules (`~/.agents/skills/rust-skills`). Lints live in `host/Cargo.toml` (`[lints]`: default groups + pedantic, `unwrap_used`); CI runs `cargo fmt --check` and `cargo clippy --all-targets -- -D warnings`
- Host conventions: no `unwrap()` outside tests (`expect("why this can't fail")` for true invariants); lock std mutexes with `LockExt::locked()` (poison-tolerant); enums, not strings, for states and modes (`BotStatus`, `Permission`, `EntryKind`, `AlertKind`); `tracing` with structured fields (`error = format!("{e:#}")` keeps the context chain); blocking fs/process work goes through `spawn_blocking`; registry JSON is untrusted (paths are validated)

## Clean code

- Keep code modular and clean on every change: reuse before writing, one responsibility per file/type, no new code appended to files past ~500 lines (split first), views/components hold no I/O or logic, short single-purpose functions, enums over flags, no dead code, no swallowed errors. Full rules: [docs/guides/clean-code.md](docs/guides/clean-code.md).

## Architecture

- Clients: iOS app (SwiftUI), desktop app for macOS, Linux and Windows (`apps/desktop/`, Electron: menu bar/tray + chat window; why and how: [docs/architecture/desktop-app.md](docs/architecture/desktop-app.md)), terminal UI (`codync-host tui`, `host/src/tui/`, ratatui; layout and state vocabulary modeled on herdr). All talk to the host API.
- iPhone Swift package (`apps/ios/Kit/`, iOS only): `CodyncKit` (models, client, theme, avatars; also used by the widgets and the notification extension) + `CodyncUI` (`BotStore` + screens).
- UI principle: buttons an icon can express are icon-only (with tooltip / accessibility label); text only where an icon would be ambiguous (approval choices).
- UI controls default to the shared custom components. Explicit exception: iOS BotListView and ThreadView use native navigation/toolbar items and automatic back navigation for system Liquid Glass, as specified in `docs/design/ui-conventions.md`. **iOS menus are always the system ones**: tap menus through `DropdownMenu`/`ChoicePicker` (native `Menu`), long-press through `.contextActions` (native `contextMenu`). Never hand-build a dropdown on iOS: a custom overlay lands in the wrong place (sheets, scroll views, the composer's + menu). The desktop app's menu bar/tray menu is the system menu. Keep system authentication and widget containers native. On iOS `.codyncSheet` presents the system sheet (grabber, swipe down). Outside these exceptions, avoid: no `Menu`/`Picker`, `.switch` toggles, `Form`/`List` styling, `confirmationDialog`/`alert`, `ProgressView`, `.sheet`/`.popover`/`.fullScreenCover`, `.toolbar`/navigation bars, `TabView`, `ContentUnavailableView`. Use `apps/ios/Kit/Sources/CodyncUI/Controls.swift` + `Chrome.swift` (`.codyncSheet`, `ModalHeader`, `ScreenHeader`, `TabBar`, `.codyncDialog`, `ToggleStyle.codync`); on the desktop, `apps/desktop/src/renderer/components/` (`Sheet`, `Dialog`, `AnchoredMenu`, `ModalHeader`, `Controls.tsx`, `Icon` for SF Symbols). Every tap that shows/hides something animates (`Motion`). Anything with a background fill gets no border line.
- `host/` — **codync-host** (Rust, macOS + Linux + Windows; platform differences in `service/` and `shell.rs`). Detects installed harnesses (`agent/backends.rs`: login-shell PATH + known dirs) and the ACP registry (`agent/registry.rs`, cached in `~/.codync/registry.json`, binaries under `~/.codync/agents`). Drives agents over **ACP** (JSON-RPC on stdio, hand-rolled in `agent/acp.rs`, updates kept as `serde_json::Value` so new adapter variants never break parsing). One actor per bot (`agent/bot/`) owns the agent process + session and maps `session/update` onto transcript entries.
- **Chat ≠ session**: a bot is one endless transcript (SQLite `entries`, ordered by `seq`); the ACP session underneath is resumed with `session/load` or replaced by *New session*.
- **Group chats and threads** are host features every client drives through the same methods (`send` with `threadId`, `thread`, `createBot {kind: group}`); clients never route, parse mentions or count replies themselves. A group is a roster row whose members answer in their own sessions (Grok Bot's room turns); a thread on a bot's message is a forked session: [docs/features/groups-and-threads.md](docs/features/groups-and-threads.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leepokai/Codync](https://github.com/leepokai/Codync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
