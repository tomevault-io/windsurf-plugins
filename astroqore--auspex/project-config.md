---
trigger: always_on
description: Operating manual for AI agents (and humans) working in this repository.
---

# AGENTS.md

Operating manual for AI agents (and humans) working in this repository.
Authoritative; `CLAUDE.md` just points here.

## 1. What Auspex Is

A native macOS app that observes every AI coding agent running on the user's
Mac — Claude Code, Codex, Cursor, Grok Build, Antigravity — and shows them on
one live board: who is thinking, calling tools, delegating to sub-agents,
writing files, or waiting for permission. Sessions group by project and task,
and the task board is exposed over MCP.

Pure Swift package, **unsandboxed**, no Xcode workspace, no installer, no
server, and no network except the one it uses to update itself. Builds are
ad-hoc signed until an Apple Developer ID is configured; either way releases
are tagged, signed with the project's Sparkle key, and delivered over two
update channels (§ 9).

**Status: pre-alpha.** The repository is a skeleton; most of
`docs/ARCHITECTURE.md` is target design, not shipped code.

## 2. Repository Map

```
Package.swift              SwiftPM manifest — tools 6.2, macOS 26, Swift 6 language mode
Sources/
  AuspexCore/              Testable library. All logic lives here.
    AuspexPaths.swift      Single source of truth for the ~/.auspex/ tree
    AuspexVersion.swift    Build identity; reads Resources/Info.plist
    Config/UpdateChannel.swift  Stable vs Dev, and what each means to Sparkle
    Store/AuspexStore.swift  GRDB store + DatabaseMigrator
  AuspexApp/               SwiftUI executable. UI glue only.
    main.swift             Subcommand dispatch, then AuspexApp.main()
    AuspexApp.swift        App scene: WindowGroup + MenuBarExtra
    AppEnvironment.swift   Observable dependency holder (placeholder)
    RootView.swift         NavigationSplitView shell
    Updates/AppUpdateController.swift  The one Sparkle updater in the process
Tests/AuspexCoreTests/     swift-testing suites
Resources/
  Info.plist               Bundle metadata; com.astroqore.auspex
  Auspex.entitlements      Deliberately empty — see § 5
Scripts/build_app.sh       swift build → .app bundle → Sparkle → codesign → sandbox assert
Scripts/release_app.sh     Bump, close the changelog, branch, tag. Builds nothing (§ 9)
Scripts/generate_update_feed.sh  One archive → one signed appcast item (§ 9)
RELEASING.md               The release runbook, secrets, and the Sparkle key
docs/ARCHITECTURE.md       Target architecture
```

**The target split is load-bearing.** `AuspexCore` holds every parser, reducer,
adapter, and storage type, and is testable without a running app.
`AuspexApp` holds windows, scenes, and view code. If a piece of logic is worth
a test, it belongs in Core.

## 3. Toolchain

macOS 26 (Tahoe) or newer on Apple silicon, Xcode 26 / Swift 6.2 or newer.
Both targets compile in Swift 6 language mode; keep it that way.

### 3.1 Dependencies, and the `agent-session-kit` pin

Three, all pinned in `Package.swift`:

| Package | Pin | Why |
| --- | --- | --- |
| `GRDB.swift` | `from: 7.0.0` | The local store. |
| `agent-session-kit` | `exact: "0.6.1"` | The harness adapters and the live pipeline. |
| `Sparkle` | `exact: "2.9.4"` | In-app updates (§ 9). |

`Package.resolved` is gitignored, so a release built from a clean checkout of
a tag has nothing but `Package.swift` to tell it which dependency versions to
compile in. That is why the two that matter are `exact:` rather than a range:
the pin *is* the record of what shipped, and two builds of the same Auspex
commit have to contain the same kit.

**To work on Auspex and the kit side by side**, do not edit the pin. Put the
kit in edit mode, which is SwiftPM's own answer and leaves the manifest alone:

```sh
swift package edit agent-session-kit --path ../agent-session-kit
swift build            # now compiles your working tree of the kit
swift package unedit agent-session-kit
```

`.swiftpm/` is gitignored, so the edit-mode state never reaches a commit. When
the kit change is ready, release it there, then bump the pin here — as its own
commit, with a reason in the body.
`.github/workflows/bump-agent-session-kit.yml` opens that pull request on its
own when the kit publishes a newer release. It never merges anything.

## 4. Verification Before Completion

Before claiming a change works, run all four:

```sh
swift build
swift test
./Scripts/build_app.sh release
codesign -dv --entitlements - .build/Auspex.app
```

The `codesign` output must be an empty `<dict/>` plist with no
`com.apple.security.app-sandbox` key. `build_app.sh` also asserts this and
fails the build if the key appears.

### 4.1 Performance budget (first priority, not a nice-to-have)

Auspex runs all day next to the harnesses it watches. Rich motion is wanted;
paying for it with CPU is not. Every change that touches the pipeline, the
board, the scene or the crew must hold these numbers on this machine's
real stores (~600 sessions) and say so in the PR/commit/report:

| Situation | Budget |
| --- | --- |
| Live, window visible, no user input, ≥ 2 min after launch | ≤ 3 % process CPU, main thread idle |
| Live, during a harness burst (a transcript growing every second) | ≤ 10 % process CPU, board updates within 0.5 s |
| Scene or Crew view on screen, 60 animating characters | ≤ 15 % process CPU; offscreen views cost 0 (clocks stop when hidden) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AstroQore/auspex](https://github.com/AstroQore/auspex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
