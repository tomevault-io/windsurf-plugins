---
trigger: always_on
description: macOS menu-bar app for real-time audio transcription using MLX on Apple Silicon.
---

# AGENTS.md — OpenSuperMLX

macOS menu-bar app for real-time audio transcription using MLX on Apple Silicon.
Swift 5 / SwiftUI, Xcode project, targeting macOS 14.0+ (Sonoma), ARM64 only.

## Agent Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Always apply — bias toward caution over speed. For trivial tasks, use judgment.

*Source: [Karpathy guidelines](https://github.com/forrestchang/andrej-karpathy-skills/blob/main/.cursor/rules/karpathy-guidelines.mdc)*

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Build Commands

```bash
# Prerequisites
brew install cmake libomp rust ruby && gem install xcpretty
git submodule update --init --recursive

# Full build (Rust dylib + WeTextProcessing + patches + Xcode)
./run.sh build

# Build and run
./run.sh
```

### Fast Incremental Build (~3-5 seconds)

For **Swift-only changes**, skip `run.sh` — the **preferred build command during development**:

```bash
xcodebuild -scheme OpenSuperMLX -configuration Debug -jobs 8 \
  -derivedDataPath build -quiet -destination 'platform=macOS,arch=arm64' \
  -skipPackagePluginValidation -skipMacroValidation \
  -clonedSourcePackagesDirPath SourcePackages CODE_SIGNING_ALLOWED=NO build
```

**Fall back to `./run.sh build`** when: first build on fresh clone, after modifying `asian-autocorrect/` (Rust), `patches/*.patch`, `WeTextProcessing/` (ITN binary), SPM dependencies, or if `build/` was deleted.

**`VendoredPackages/mlx-audio-swift/`** — edit Swift source directly; changes are picked up by the incremental xcodebuild above (no `./run.sh` needed).

## Tests

```bash
# All unit tests
xcodebuild test -scheme OpenSuperMLX -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath build -clonedSourcePackagesDirPath SourcePackages \
  CODE_SIGNING_ALLOWED=NO -only-testing:OpenSuperMLXTests

# Single test class
xcodebuild test -scheme OpenSuperMLX -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath build -clonedSourcePackagesDirPath SourcePackages \
  CODE_SIGNING_ALLOWED=NO -only-testing:OpenSuperMLXTests/ITNProcessorTests

# Single test method
xcodebuild test -scheme OpenSuperMLX -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath build -clonedSourcePackagesDirPath SourcePackages \
  CODE_SIGNING_ALLOWED=NO \
  -only-testing:OpenSuperMLXTests/ITNProcessorTests/testCleanDuplicateChinesePunctuation
```

`ClipboardUtilPasteIntegrationTests` require accessibility permissions and an active display — they `XCTSkip` when unavailable.

## CLI Test Harness

The binary doubles as a CLI test harness — subcommand runs headlessly, no subcommand launches the GUI. 10 commands: `transcribe`, `stream-simulate`, `correct`, `config`, `recordings`, `queue`, `mic`, `model`, `benchmark`, `diagnose`. All accept `--json`, `--quiet`, `--verbose` flags after the subcommand.

```bash
BINARY=build/Build/Products/Debug/OpenSuperMLX.app/Contents/MacOS/OpenSuperMLX
$BINARY diagnose --json          # environment snapshot
$BINARY help <command>            # per-command usage
```

For full command reference, test commands, and error codes, see [`docs/cli.md`](docs/cli.md).

## Patches


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axot/OpenSuperMLX](https://github.com/axot/OpenSuperMLX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
