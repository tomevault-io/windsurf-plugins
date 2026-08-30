---
trigger: always_on
description: `./scripts/setup.sh` initializes submodules, builds GhosttyKit, and installs the pbxproj normalization pre-commit hook.
---

# cmux agent notes

## Setup

`./scripts/setup.sh` initializes submodules, builds GhosttyKit, and installs the pbxproj normalization pre-commit hook.

## Build and reload

Always build with a tag. **Never run bare `xcodebuild` or `open` an untagged `cmux DEV.app`**: untagged builds share the default debug socket and bundle ID with other agents, causing conflicts and stealing focus.

```bash
./scripts/reload.sh --tag <branch-slug>            # build Debug, kill same-tag app, do not launch
./scripts/reload.sh --tag <branch-slug> --launch   # also open it
```

A tag gives the app its own name, bundle ID, socket, and derived data path, so it runs side-by-side with the user's main app. Report the build to the user as a markdown link to `http://127.0.0.1:17320/<tag>`. Never put a `file://` URL, a raw `.app` path, or `/tmp/cmux-<tag>/...` in chat output.

Other variants: `reloadp.sh` (Release), `reloads.sh` (Release as isolated "cmux STAGING"), `reload2.sh --tag <tag>` (both).

Compile-only check, no launch:

```bash
xcodebuild -project cmux.xcodeproj -scheme cmux -configuration Debug -destination 'platform=macOS' -derivedDataPath /tmp/cmux-<tag> build
```

Rebuild GhosttyKit.xcframework with Release optimizations:

```bash
cd ghostty && zig build -Demit-xcframework=true -Dxcframework-target=universal -Doptimize=ReleaseFast
```

Clean up older tags you started this session (quit the app, remove its `/tmp` socket and derived data) before launching a new one.

## Tag-bound debug CLI

For CLI or socket dogfood against a tagged Debug app, set `CMUX_TAG` and use the helper. Do not use `/tmp/cmux-cli`, which points at the most recently reloaded build and can target the user's main app socket.

```bash
CMUX_TAG=<tag> scripts/cmux-debug-cli.sh list-workspaces
CMUX_TAG=<tag> scripts/cmux-debug-cli.sh send --workspace workspace:1 --surface surface:1 "echo ok"
```

The helper refuses to run without `CMUX_TAG`, targets `/tmp/cmux-debug-<tag>.sock`, and uses the matching tagged CLI from DerivedData. It scrubs ambient cmux terminal context (`CMUX_SOCKET`, `CMUX_SOCKET_PASSWORD`, workspace/surface/tab/panel IDs, cmuxd socket, debug log), then sets `CMUX_SOCKET_PATH`, `CMUX_BUNDLE_ID`, and `CMUX_BUNDLED_CLI_PATH` for the tag.

## iOS builds open on the iPhone by default

Any work verified by opening the iOS app installs BOTH an isolated-simulator build AND the same build on the user's iPhone. Never stop at simulator-only. Use `ios/scripts/reload-cloud.sh --tag <tag>` (or `ios/scripts/reload.sh --tag <tag>`); with a default iPhone configured (`CMUX_IPHONE_DEVICE_ID` or `~/.config/cmux/iphone-device-id`) the device leg is automatic, and `--device-id <id>` still overrides (`xcrun devicectl list devices`). Auto sign-in and auto-pair apply as usual; launch the app so it is immediately open on the phone. The simulator leg uses the tag's own isolated device `cmux-dev-<slug>`, created on demand; do not target a shared or user-visible simulator.

**Every phone install MUST be authenticated before handoff — installed-but-signed-out is a failed install.** A tagged bundle id is a fresh identity with no session, so after EVERY install (first install or rebuild of an existing tag), the app must be past the login screen and paired. This is mechanically enforced, not advisory: device launches default to the full `--ensure-mac` flow, and the **iPhone auth gate** inside `scripts/mobile-dev-launch.sh` exits non-zero (with the exact retry command) unless the tagged Mac observes a signed-in + paired `mobile.rpc.ready` session from the phone. `scripts/verify-iphone-auth.sh --tag <tag> [--device-id <id>]` is the required post-install verification verb when you installed any other way or need to re-check later: it relaunches the app WITHOUT injecting credentials and passes only if persisted sign-in + pairing produce a usable RPC session (PASS/FAIL with reason, no screenshots). Never install with raw `devicectl device install app` (it skips sign-in entirely and the local-build-guards wrapper refuses it for the configured personal iPhone), and never pass `--no-sign-in`/`--no-attach`/`--no-setup` for a dogfood build — the scripts refuse them for device installs unless `CMUX_ALLOW_UNAUTHENTICATED_INSTALL=1` is set, and that variable is **human-only: agents never set it** (same convention as `CMUX_ALLOW_LOCAL_XCODEBUILD`). If sign-in cannot be completed (missing `~/.secrets/cmuxterm-dev.env`, web API down, phone locked), the gate's failure output is the truth: put its reason and retry command in the handoff instead of handing off a logged-out app.

Every phone build requires the same-tag Mac dev build (the iOS app is unusable without its Mac). The reload scripts build the Mac tag first when it is missing and refuse to ship a phone-only build if that fails; do not bypass this with `CMUX_IOS_SKIP_MAC_BUILD_CHECK` in normal work.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tankxu/fleet](https://github.com/tankxu/fleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
