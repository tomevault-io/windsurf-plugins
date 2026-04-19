---
trigger: always_on
description: Run the setup script to initialize submodules and build GhosttyKit:
---

# cmux agent notes

## Initial setup

Run the setup script to initialize submodules and build GhosttyKit:

```bash
./scripts/setup.sh
```

## Local dev

After making code changes, always run the reload script with a tag to build the Debug app:

```bash
./scripts/reload.sh --tag fix-zsh-autosuggestions
```

By default, `reload.sh` builds but does **not** launch the app. The script prints the `.app` path so the user can cmd-click to open it. After a successful build, it always terminates any running app with the same tag (so cmd-clicking launches the freshly-built binary instead of foregrounding the stale instance). Pass `--launch` to open the app automatically after the build:

```bash
./scripts/reload.sh --tag fix-zsh-autosuggestions --launch
```

`reload.sh` prints an `App path:` line with the absolute path to the built `.app`. Use that path to build a cmd-clickable `file://` URL. Steps:

1. Grab the path from the `App path:` line in `reload.sh` output.
2. Prepend `file://` and URL-encode spaces as `%20`. Do not hardcode any part of the path.
3. Format it as a markdown link using the template for your agent type.

Example. If `reload.sh` output contains:
```
App path:
  /Users/someone/Library/Developer/Xcode/DerivedData/cmux-my-tag/Build/Products/Debug/cmux DEV my-tag.app
```

**Claude Code** outputs:
```markdown
=======================================================
[cmux DEV my-tag.app](file:///Users/someone/Library/Developer/Xcode/DerivedData/cmux-my-tag/Build/Products/Debug/cmux%20DEV%20my-tag.app)
=======================================================
```

**Codex** outputs:
```
=======================================================
[my-tag: file:///Users/someone/Library/Developer/Xcode/DerivedData/cmux-my-tag/Build/Products/Debug/cmux%20DEV%20my-tag.app](file:///Users/someone/Library/Developer/Xcode/DerivedData/cmux-my-tag/Build/Products/Debug/cmux%20DEV%20my-tag.app)
=======================================================
```

Never use `/tmp/cmux-<tag>/...` app links in chat output.

After making code changes, always use `reload.sh --tag` to build. **Never run bare `xcodebuild` or `open` an untagged `cmux DEV.app`.** Untagged builds share the default debug socket and bundle ID with other agents, causing conflicts and stealing focus.

```bash
./scripts/reload.sh --tag <your-branch-slug>
```

If you only need to verify the build compiles (no launch), use a tagged derivedDataPath:

```bash
xcodebuild -project GhosttyTabs.xcodeproj -scheme cmux -configuration Debug -destination 'platform=macOS' -derivedDataPath /tmp/cmux-<your-tag> build
```

When rebuilding GhosttyKit.xcframework, always use Release optimizations:

```bash
cd ghostty && zig build -Demit-xcframework=true -Dxcframework-target=universal -Doptimize=ReleaseFast
```

When rebuilding cmuxd for release/bundling, always use ReleaseFast:

```bash
cd cmuxd && zig build -Doptimize=ReleaseFast
```

`reload` = build the Debug app (tag required) and terminate any running app with the same tag. Pass `--launch` to also open the freshly-built app:

```bash
./scripts/reload.sh --tag <tag>
./scripts/reload.sh --tag <tag> --launch
```

`reloadp` = kill and launch the Release app:

```bash
./scripts/reloadp.sh
```

`reloads` = kill and launch the Release app as "cmux STAGING" (isolated from production cmux):

```bash
./scripts/reloads.sh
```

`reload2` = reload both Debug and Release (tag required for Debug reload):

```bash
./scripts/reload2.sh --tag <tag>
```

For parallel/isolated builds (e.g., testing a feature alongside the main app), use `--tag` with a short descriptive name:

```bash
./scripts/reload.sh --tag fix-blur-effect
```

This creates an isolated app with its own name, bundle ID, socket, and derived data path so it runs side-by-side with the main app. Important: use a non-`/tmp` derived data path if you need xcframework resolution (the script handles this automatically).

Before launching a new tagged run, clean up any older tags you started in this session (quit old tagged app + remove its `/tmp` socket/derived data).

## Debug event log

All debug events (keys, mouse, focus, splits, tabs) go to a unified log in DEBUG builds:

```bash
tail -f "$(cat /tmp/cmux-last-debug-log-path 2>/dev/null || echo /tmp/cmux-debug.log)"
```

- Untagged Debug app: `/tmp/cmux-debug.log`
- Tagged Debug app (`./scripts/reload.sh --tag <tag>`): `/tmp/cmux-debug-<tag>.log`
- `reload.sh` writes the current path to `/tmp/cmux-last-debug-log-path`
- `reload.sh` writes the selected dev CLI path to `/tmp/cmux-last-cli-path`
- `reload.sh` updates `/tmp/cmux-cli` and `$HOME/.local/bin/cmux-dev` to that CLI

- Implementation: `vendor/bonsplit/Sources/Bonsplit/Public/DebugEventLog.swift`
- Free function `dlog("message")` — logs with timestamp and appends to file in real time
- Entire file is `#if DEBUG`; all call sites must be wrapped in `#if DEBUG` / `#endif`
- 500-entry ring buffer; `DebugEventLog.shared.dump()` writes full buffer to file
- Key events logged in `AppDelegate.swift` (monitor, performKeyEquivalent)
- Mouse/UI events logged inline in views (ContentView, BrowserPanelView, etc.)
- Focus events: `focus.panel`, `focus.bonsplit`, `focus.firstResponder`, `focus.moveFocus`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manaflow-ai/cmux](https://github.com/manaflow-ai/cmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
