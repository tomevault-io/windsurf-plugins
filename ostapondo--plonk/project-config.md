---
trigger: always_on
description: Rules for AI agents working in this repo.
---

# Agent rules

Rules for AI agents working in this repo.

## Layout

- `App/` — Swift package, the menu bar app. One type per file, files stay under ~300 lines.
- `mcp/` — TypeScript MCP server: `src/server.ts` compiled to `dist/` via `npm run build`. Keep it a thin proxy; logic belongs in the app. Tests are `mcp/test/*.test.js`, run against `dist/` on the Node test runner with no extra dependencies. Tool description rules are in `mcp/AGENTS.md`.
- `scripts/build.sh` — the only build entry point.

Inside `App/Sources/plonk/`, the pieces that are easy to get lost in:

- `Router` owns every HTTP route. `AppDelegate` owns lifecycle, windows and the
  status menu, and nothing else. It is over the line limit and may only shrink,
  so anything new goes in an extension of its own — see `AppDelegate+Shell`.
- `Ink` holds the surfaces, the radius and the accent gradient every page draws
  with, and `SettingRows` the rows a settings card is made of. Pages compose
  those instead of spelling out colours.
- `Resources/en.lproj/Localizable.strings` holds every word the user reads, and
  the `Strings+*.swift` files declare the keys that reach it. Views hold keys,
  never sentences — see "Text" below.
- `AppActions` is the full list of things the UI can ask the app to do.
  `AppModel` is state only; views never reach past it.
- `ScreenIdentity` turns a screen index into the keys config is stored under.

## Adding a module

Plonk is a suite; each capability (workspaces, zones, keep-awake, screenshots, …)
is a module. A new module touches five places, nothing else:

1. A manager type in `App/Sources/plonk/` owning the behavior.
2. Methods on `AppActions`, implemented in the `AppDelegate` extension.
3. A `SettingsPage` entry in `SettingsPages.all`, naming the `SettingsGroup` it
   belongs under as its `parent` (that is what the sidebar draws), plus any
   status-menu items in `StatusMenuController`.
4. HTTP routes under its own path prefix in `Router.handle` (e.g. `/shot/*`).
5. An MCP tool file `mcp/src/tools/<module>.ts` with a `register(server)`
   function, wired in `mcp/src/server.ts`.

Config lives as new fields on `Config` with `decodeIfPresent` defaults so old
config files keep working. Anything stored per monitor is keyed by display UUID
via `ScreenIdentity.keys(forIndex:)`, never by the bare index: indices shift
when a display is unplugged.

## Build & verify

Each line is a subshell, so the block runs as written from the repository root:

```sh
(cd App && swift build)          # must pass before any commit
./scripts/test.sh                # unit tests — must pass
./scripts/lint.sh                # style rules — must pass
(cd mcp && npm test)             # must pass when mcp/ changed
node scripts/check-zone-sets.mjs # must pass when zone-sets/ changed
node scripts/check-strings.mjs   # must pass when any user-facing text changed
./scripts/build.sh               # produces Plonk.app; needs a signing identity
curl -s 127.0.0.1:43917/ping     # smoke test while the app is running
```

The first four need no signing certificate. `build.sh` does, and refuses
rather than produce a bundle that cannot hold its permissions; see
[CONTRIBUTING.md](CONTRIBUTING.md) for why and how to make one.

`scripts/lint.sh` enforces the file-length rule above, plus no emoji, no
trailing whitespace and a final newline. Files already over 300 lines are
recorded in `scripts/line-limit-baseline` with the length they had that day:
they may shrink, never grow, and a new file has to come in under the limit
outright. Shortening one means lowering its number in the same commit.

Anything that decides where a window lands is checked by hand, because none of
it is reachable from the unit suite. `scripts/testbench.sh up 4` opens
throwaway TextEdit windows, `state` prints where each landed as fractions, and
`down` clears them away. Use it instead of moving the user's real windows.

The release number lives in `version.env`, and only there. `scripts/build.sh`
reads `MARKETING_VERSION` and `BUILD_NUMBER` into `Info.plist`. Bump them to cut
a release; never edit a version inside the plist heredoc.

Pure logic lives in `ZoneGeometry`, `Config`, `ImageFit`, `Router` and
`ControlServer.parseIfComplete` so it stays testable without a desktop session.
Put new logic there and cover it in `App/Tests/plonkTests/`.

## Code style

- Swift API design guidelines; match the existing code.
- Comments only for non-obvious constraints (coordinate spaces, AX quirks). No narration, no changelog comments.
- No emoji anywhere, user-facing strings included. Key glyphs (⌃⌥⇧↩) are not emoji and are fine.
- Coordinate spaces are documented in `WindowManager.swift` — read that before touching geometry.
- Annotations are stored in unit coordinates, not view points; see `Annotation.swift`.

## Text

Every string the user reads lives in `App/Sources/plonk/Resources/en.lproj/`,
and nowhere else. A view holds a key:

```swift
Text(.zonesOverlay)                       // a page heading
Button(.commonCancel) { dismiss() }       // a button
HUD.shared.show(.hudNoTextFound)          // a notice
```

Adding one is two edits and no more. Put the English in
`Localizable.strings`, keyed by `<module>.<thing>`; declare it in the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ostapondo/Plonk](https://github.com/ostapondo/Plonk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
