---
trigger: always_on
description: - `RepoPrompt/`: macOS app source (Swift/SwiftUI), features, services, models, views.
---

# Repository Guidelines

## Project Structure & Module Organization
- `RepoPrompt/`: macOS app source (Swift/SwiftUI), features, services, models, views.
- `repoprompt-mcp/`: CLI client for MCP connection and discovery.
- `RepoPromptTests/` and `RepoPromptUITests/`: XCTest suites.

## Build, Test, and Development
- Target: macOS 14+ (Xcode 15+).
- New tests live in `RepoPromptTests/` with `*Tests.swift` naming.
- Use the standard `xcodebuild` CLI directly for Xcode builds and tests.

### Build and Test Commands

Discover project and scheme names when needed:

```bash
xcodebuild -list -project RepoPrompt.xcodeproj
```

Build and launch:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build
APP_PATH="$(xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug -showBuildSettings | awk -F ' = ' '/TARGET_BUILD_DIR/ { dir=$2 } /FULL_PRODUCT_NAME/ { name=$2 } END { print dir "/" name }')"
open "$APP_PATH"
```

Show build settings or clean build products:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -showBuildSettings
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt clean
```

Testing rules:

- Never run the full test suite.
- Never run the full UI test target.
- Prefer `RepoPromptTests/` selectors by default.
- Use a targeted `RepoPromptUITests/` selector when UI or integration behavior is the most relevant validation path.
- Selectors should be target-prefixed, for example:
  - `RepoPromptTests/ApplyEditsCoreTests`
  - `RepoPromptTests/ApplyEditsCoreTests/testExactMatch`
  - `RepoPromptUITests/RepoPromptUITests/testAgentChatStressHarnessKeepsAutoFollowAndProducesGrouping`

Examples:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/ApplyEditsCoreTests
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/ApplyEditsCoreTests/testExactMatch
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt test -only-testing:RepoPromptTests/PathMatcherTests -only-testing:RepoPromptTests/ApplyEditsCoreTests/testExactMatch
```

Notes:

- Prefer a single blocking `xcodebuild` command first.
- Use `-resultBundlePath <path>` when you need a result bundle for detailed failure analysis.
- Do not inspect PIDs or kill processes unless explicitly asked.

### Validation

Validate when a change can affect runtime behavior, buildability, packaging, MCP behavior, or CLI behavior.

- If you are already running a relevant `xcodebuild ... test ...`, do not run a separate build first.
- Otherwise run `xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build`.
- Then run the smallest relevant test scope.
- Skip validation for docs, comments, prompt text, copy-only tweaks, or formatting-only changes.
- If changes affect the MCP server, CLI, or any feature that depends on the running app, validate with a local app launch plus the smallest useful CLI/MCP smoke check.

Use this flow when a feature needs live app interaction testing:

1. Build and launch the debug app:

```bash
xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug build
APP_PATH="$(xcodebuild -project RepoPrompt.xcodeproj -scheme RepoPrompt -configuration Debug -showBuildSettings | awk -F ' = ' '/TARGET_BUILD_DIR/ { dir=$2 } /FULL_PRODUCT_NAME/ { name=$2 } END { print dir "/" name }')"
open "$APP_PATH"
```

2. Connect with the debug CLI and run the smallest useful check for the feature under test:

```bash
rp-cli-debug -e 'windows'
rp-cli-debug -w 1 --call get_file_tree --json '{"type":"roots"}'
```

Then use `agent_run` to test agent functionality in the debug app, or use other targeted tools as needed to test non-agent functionality.

### Live Agent Mode / Claude Investigations

Before live Claude or Agent Mode smokes, enable debug-only diagnostics through the MCP `app_settings` surface of the debug app:

```bash
rp-cli-debug -w 1 --call app_settings --json '{"op":"list","group":"agent_mode","detailed":true}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.claude_raw_event_logging_enabled","value":true}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.claude_raw_event_log_file_path","value":"/tmp/repoprompt-claude-raw-events"}'
rp-cli-debug -w 1 --call app_settings --json '{"op":"set","key":"agent_mode.perf_diagnostics_enabled","value":true}'
```

These settings are intentionally DEBUG-only; if a key is unavailable, confirm you are connected to a current debug build before using lower-level UserDefaults fallbacks. Never hard-code investigation preferences in Swift source; keep them runtime-configurable and document exact keys/values in investigation notes.

### Symbolication Archives

For crash or performance investigations that need symbolication, check Xcode archives under:

```bash
~/Library/Developer/Xcode/Archives/<YYYY-MM-DD>/
```

Recent production archives may include paths like:

```bash
~/Library/Developer/Xcode/Archives/2026-05-13/RepoPrompt 2026-05-13, 10.55 AM.xcarchive
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [repoprompt/repoprompt-classic](https://github.com/repoprompt/repoprompt-classic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
