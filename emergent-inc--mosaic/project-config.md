---
trigger: always_on
description: Run the setup script to initialize submodules, build GhosttyKit, and install the pbxproj normalization pre-commit hook:
---

# mosaic agent notes

## Initial setup

Run the setup script to initialize submodules, build GhosttyKit, and install the pbxproj normalization pre-commit hook:

```bash
./scripts/setup.sh
```

## Local dev

After making code changes, always run the reload script with a tag to build the Debug app:

```bash
./scripts/reload.sh --tag fix-zsh-autosuggestions
```

For fast Swift/UI iteration on a tag that already has warmed DerivedData, use the opt-in fast path:

```bash
MOSAIC_DEV_FAST_RELOAD=1 ./scripts/reload.sh --tag fix-zsh-autosuggestions
```

`MOSAIC_DEV_FAST_RELOAD=1` keeps the same Xcode compile graph but skips slow dev packaging work: it skips the Ghostty CLI helper Zig rebuild, reuses an existing `mosaicd` binary when present, and retags the Xcode-built app in place instead of copying the whole `.app` bundle. In local timing, a warmed normal tagged reload took 226s, while warmed fast reloads took 26-36s. Use the normal `reload.sh --tag` path when changing Ghostty, `mosaicd`, helper binaries, signing/bundle packaging, or tag/socket isolation behavior.

By default, `reload.sh` builds but does **not** launch the app. The script prints the `.app` path so the user can cmd-click to open it. After a successful build, it always terminates any running app with the same tag (so cmd-clicking launches the freshly-built binary instead of foregrounding the stale instance). Pass `--launch` to open the app automatically after the build:

```bash
./scripts/reload.sh --tag fix-zsh-autosuggestions --launch
```

`reload.sh` prints an `App path:` line with the absolute path to the built `.app`. Use that path to build a cmd-clickable `file://` URL. Steps:

1. Grab the path from the `App path:` line in `reload.sh` output.
2. Prepend `file://` and URL-encode spaces as `%20`. Do not hardcode any part of the path.
3. Format it as a markdown link using the template for your agent type.

Example. If `reload.sh` output contains:

```text
App path:
 /Users/someone/Library/Developer/Xcode/DerivedData/mosaic-my-tag/Build/Products/Debug/Mosaic DEV my-tag.app
```

**Claude Code** outputs:

```markdown
-------------------------------------------------------
[Mosaic DEV my-tag.app](file:///Users/someone/Library/Developer/Xcode/DerivedData/mosaic-my-tag/Build/Products/Debug/Mosaic%20DEV%20my-tag.app)
-------------------------------------------------------
```

**Codex** outputs:

```markdown
-------------------------------------------------------
[my-tag: file:///Users/someone/Library/Developer/Xcode/DerivedData/mosaic-my-tag/Build/Products/Debug/Mosaic%20DEV%20my-tag.app](file:///Users/someone/Library/Developer/Xcode/DerivedData/mosaic-my-tag/Build/Products/Debug/Mosaic%20DEV%20my-tag.app)
-------------------------------------------------------
```

Never use `/tmp/mosaic-<tag>/...` app links in chat output.

For CLI or socket dogfood against a tagged Debug app, use the tag-bound helper and set `MOSAIC_TAG`.
Do not use `/tmp/mosaic-cli` for tagged dogfood, since that symlink points at the most recently reloaded build and can target the user's main app socket.

```bash
MOSAIC_TAG=<tag> scripts/mosaic-debug-cli.sh list-workspaces
MOSAIC_TAG=<tag> scripts/mosaic-debug-cli.sh send --workspace workspace:1 --surface surface:1 "echo ok"
```

The helper refuses to run without `MOSAIC_TAG`, targets `/tmp/mosaic-debug-<tag>.sock`, and uses the matching tagged CLI from `~/Library/Developer/Xcode/DerivedData/mosaic-<tag>/...`. It also scrubs ambient mosaic terminal context (`MOSAIC_SOCKET`, `MOSAIC_SOCKET_PASSWORD`, workspace/surface/tab/panel IDs, mosaicd socket, and debug log), then sets `MOSAIC_SOCKET_PATH`, `MOSAIC_BUNDLE_ID`, and `MOSAIC_BUNDLED_CLI_PATH` for the selected tag.

After making code changes, always use `reload.sh --tag` to build. **Never run bare `xcodebuild` or `open` an untagged `Mosaic DEV.app`.** Untagged builds share the default debug socket and bundle ID with other agents, causing conflicts and stealing focus.

```bash
./scripts/reload.sh --tag <your-branch-slug>
```

If you only need to verify the build compiles (no launch), use a tagged derivedDataPath:

```bash
xcodebuild -project mosaic.xcodeproj -scheme mosaic -configuration Debug -destination 'platform=macOS' -derivedDataPath /tmp/mosaic-<your-tag> build
```

When rebuilding GhosttyKit.xcframework, always use Release optimizations:

```bash
cd ghostty && zig build -Demit-xcframework=true -Dxcframework-target=universal -Doptimize=ReleaseFast
```

When rebuilding mosaicd for release/bundling, always use ReleaseFast:

```bash
cd mosaicd && zig build -Doptimize=ReleaseFast
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

`reloads` = kill and launch the Release app as "Mosaic STAGING" (isolated from production mosaic):

```bash
./scripts/reloads.sh
```

`reload2` = reload both Debug and Release (tag required for Debug reload):

```bash
./scripts/reload2.sh --tag <tag>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emergent-inc/mosaic](https://github.com/emergent-inc/mosaic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
