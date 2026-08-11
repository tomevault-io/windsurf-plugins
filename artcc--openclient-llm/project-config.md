---
trigger: always_on
description: This file is the project-wide operating guide. It resolves facts that would otherwise require reading multiple configuration files; focused implementation rules live in `specs/`.
---

# AGENTS.md

This file is the project-wide operating guide. It resolves facts that would otherwise require reading multiple configuration files; focused implementation rules live in `specs/`.

## Instruction Order

1. Read this file before changing the project.
2. Read every specification relevant to the requested work before editing.
3. If instructions conflict, follow repository-specific instructions over general guidance. When two project instructions conflict, stop and ask for clarification.
4. Keep this file limited to durable, project-wide facts. Put focused or evolving implementation rules in `specs/`.

## Specifications

Each specification must use the `.instructions.md` suffix and start with YAML front matter containing a `description`. Add an `applyTo` pattern when the scope can be expressed by file path. When adding or removing a specification, update this table in the same change.

| File | Read when |
|---|---|
| `agent-tool-calling.instructions.md` | Implementing tool calling, tool UI, or the agent loop. |
| `architecture.instructions.md` | Creating Swift files, features, or changing layer boundaries. |
| `changelog.instructions.md` | Updating `CHANGELOG.md`. |
| `chat-visual-style.instructions.md` | Designing chat-specific SwiftUI. |
| `code-style.instructions.md` | Writing or reviewing Swift style. |
| `concurrency.instructions.md` | Working with async code, isolation, or `Sendable`. |
| `conversation-backup-format.instructions.md` | Exporting, importing, restoring, validating, or versioning conversation backups. |
| `design-ui.instructions.md` | Designing general SwiftUI UI, accessibility, haptics, or animation. |
| `litellm-api.instructions.md` | Changing LiteLLM/OpenAI-compatible API integration. |
| `readme.instructions.md` | Updating `README.md`. |
| `roadmap.instructions.md` | Planning or prioritizing future work. |
| `roadmap-completed.instructions.md` | Reviewing completed roadmap work. |
| `security.instructions.md` | Handling sensitive data, user input, credentials, or security review. |
| `swiftui-multiplatform.instructions.md` | Building shared iOS, iPadOS, or macOS SwiftUI. |
| `testing.instructions.md` | Adding or changing tests and mocks. |
| `web-browsing.instructions.md` | Implementing web search or browsing features. |

## Platform & Services

- Target Swift 6+ with SwiftUI on iOS, iPadOS, and macOS. Minimum deployment is iOS 26 and macOS 26.
- The app connects to a self-hosted LiteLLM server through its OpenAI-compatible API. The base URL is user-configurable.
- Store credentials in `KeychainManager`; store non-sensitive settings in `SettingsManager`.

## Build & Run

```bash
# Build iOS scheme (default)
xcodebuild build -project openclient-llm.xcodeproj -scheme openclient-llm -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max'

# Build macOS scheme
xcodebuild build -project openclient-llm.xcodeproj -scheme openclient-llm-macOS -destination 'platform=macOS'
```

- Use `.xcodeproj` (not `.xcworkspace`). The three SPM packages are SwiftLintPlugins, VoticeSDK, and ConfettiSwiftUI.
- SwiftLint runs on the iOS and macOS app builds. `.swiftlint.yml` sets line-length warning/error limits to
  120/150, function-body limits to 50/80, type-body limits to 300/400, and file-length limits to 500/650;
  force unwraps and force casts are errors.
- CI skips code signing: append `CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO` to `xcodebuild` commands.
- VS Code + XcodeBuildMCP is supported (config at `.xcodebuildmcp/config.yaml`).
- **You must create a `Secrets.xcconfig` before building.** Copy the template from CI:

```bash
cat > Secrets.xcconfig << 'EOF'
VOTICE_API_KEY =
VOTICE_API_SECRET =
VOTICE_APP_ID =
EOF
```

## Concurrency (critical)

The iOS and macOS app targets set `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`; shared code therefore inherits
main-actor isolation when compiled into either app. The test, Share Extension, and WidgetsExtension targets do not set it.

- `@MainActor` annotations on ViewModels are redundant but kept for documentation.
- All test classes **must** be `@MainActor` — otherwise they cannot access `@MainActor`-isolated types synchronously.
- Use `nonisolated` ONLY for genuinely background work (image processing, large JSON parsing).
- `@unchecked Sendable` requires a documented safety invariant comment — never use without justification.
  - Production wrappers: `// Safety: <API> is thread-safe per Apple documentation. All stored properties are immutable (\`let\`).`
  - Test mocks: `// Safety: Only used within serialized @MainActor test methods.`
- No `ObservableObject` / `@Published` — use `@Observable` macro everywhere.

## Test commands

```bash
# Run all tests (iOS scheme)
xcodebuild test -project openclient-llm.xcodeproj -scheme openclient-llm -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max' -test-timeouts-enabled YES -maximum-test-execution-time-allowance 120 CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO

# Run a single test class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artcc/openclient-llm](https://github.com/artcc/openclient-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
