---
trigger: always_on
description: Native macOS AppKit GUI client for remote [Herdr](https://herdr.dev) servers. The chrome is cmux-like (sidebar, attention rings, GPU terminal) and maps 1:1 onto Herdr's own nouns: **sidebar folders are hosts, their children are spaces (Herdr workspaces), the strip above the terminal is the space's tabs, and a tab's panes are drawn as splits.** Herdr stays the multiplexer; this app owns no panes, agents, or layouts of its own — it renders what the server reports and asks the server to change it.
---

# Herdglass

Native macOS AppKit GUI client for remote [Herdr](https://herdr.dev) servers. The chrome is cmux-like (sidebar, attention rings, GPU terminal) and maps 1:1 onto Herdr's own nouns: **sidebar folders are hosts, their children are spaces (Herdr workspaces), the strip above the terminal is the space's tabs, and a tab's panes are drawn as splits.** Herdr stays the multiplexer; this app owns no panes, agents, or layouts of its own — it renders what the server reports and asks the server to change it.

This project is under the [Business Source License 1.1](LICENSE), which is not an open source
license; do not add dependencies whose terms it cannot absorb. Do not fork or copy
[cmux](https://github.com/manaflow-ai/cmux) (AGPL). Rendering uses [libghostty](https://github.com/ghostty-org/ghostty) (MIT), built by `Scripts/libghostty.sh` from the commit pinned in `Vendor/ghostty` and linked as the `GhosttyKit` binary target. The AppKit glue in `Sources/Herdglass/Ghostty/` started as [GhosttyKit](https://github.com/briannadoubt/GhosttyKit) (MIT) and is ours now.

## Layout

- `Sources/HerdrClient/` — SSH attach, Unix JSON-RPC, `terminal session control` bridge, models, split-layout tree (`Layout.swift`)
- `Sources/Herdglass/` — AppKit window, sidebar (hosts → spaces), tab strip, split container, connect sheet, Ghostty host view, ghostty-config reader (`GhosttyConfig.swift`), the chrome's type scale (`ChromeMetrics.swift`), the opt-in space keys (`SpaceKeys.swift`), settings window and macOS notifications (`AgentNotifications.swift`)
- `Sources/Herdglass/Ghostty/` — the libghostty glue: `TerminalHost` (the `ghostty_app_t` and its callbacks), `TerminalSession` (one surface), `TerminalSurfaceView` (the `NSView` and its input)
- `Tests/HerdrClientTests/` — parsing, framing, and RPC-transport tests
- `Scripts/libghostty.sh` — build `Vendor/GhosttyKit.xcframework` from the pinned ghostty; `--check` compares it with the pin
- `Scripts/dev.sh` — build `.build/Herdglass.app` and optionally `--run` (extra args pass through)
- `Scripts/release.sh` — optimized build at `.build/release-app/Herdglass.app`, `--install` to `/Applications`, optionally `--run`
- `Scripts/appicon.swift` — draws `Resources/Herdglass.icns` with CoreGraphics; the `.icns` is in git, so this runs only when the icon design changes

Entry: `HerdglassMain.swift`. Flags: `--bridge` (PTY child for libghostty), `--connect <host>` (skip the connect sheet), `--self-test <host>`, `--show-ghostty-config` (what the app took from the user's ghostty config, and the menu it produced).

## Architecture

```
GUI  --session.snapshot / events.subscribe / pane.focus-->  forwarded herdr.sock (API)
GUI  --spawns-->  Herdglass --bridge  --herdr terminal session control-->  forwarded herdr-client.sock
```

`herdr --remote` is TUI-only (cannot prefix CLI subcommands). Attach is our SSH ControlMaster:

1. `BatchMode` SSH (key must already be in `ssh-agent`)
2. Find remote `herdr` at Homebrew/mise/Nix/`~/.local/bin` — non-interactive PATH has no Homebrew
3. Ensure `herdr server`, parse `socket:` from `herdr status server`
4. Forward **both** `herdr.sock` (API) and `herdr-client.sock` (direct terminal attach)
5. Focused pane: Ghostty launches `Herdglass --bridge --target <pane> …`, which speaks NDJSON `terminal.frame` / `terminal.input` / `terminal.resize` / `terminal.scroll` / `terminal.release`
6. Scroll wheel: the GUI writes `terminal.scroll` to a per-pane FIFO (`PaneControlChannel`, path passed as `--control-pipe`) that the bridge forwards

The app is a ghostty client as much as a Herdr one, so it reads the user's own
`ghostty` config rather than inventing settings of its own: `GhosttyConfig`
snapshots the keys that describe a *window* — colours, opacity, titlebar,
`window-theme`, `confirm-close-surface`, `focus-follows-mouse` — plus the
`keybind`s for the actions this app can perform, and `GhosttyRuntime` owns the
`ghostty_config_t` everything reads from. `--show-ghostty-config` prints the lot,
including which menu shortcuts came from the config and which are ours.

`ConnectionsController` owns the window's hosts — one `SessionController` per host, each with its own SSH master, event stream and selection (space → tab → pane). Only the selected host is `isVisible`, so only its panes have bridges. `MainWindowController` turns all of that into a `SidebarModel`, a `TabBarModel` and a `SplitContainerView.Model`; `StatusStyle` is the only place agent status becomes a color or a word.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buldezir/Herdglass](https://github.com/buldezir/Herdglass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
