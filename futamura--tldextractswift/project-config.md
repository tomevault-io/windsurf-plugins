---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

TLDExtractSwift — a pure Swift library that extracts the top-level domain, second-level domain, subdomain, and root domain from a hostname or URL using the Public Suffix List (PSL), including IDNA/internationalized domains. Depends on PunycodeSwift (`Punycode` module) for IDNA encoding. Supports macOS, iOS, tvOS, watchOS, visionOS, and Linux (SPM builds). Distributed via SPM. Carthage compatibility is best-effort (not CI-verified); CocoaPods distribution has ended (3.0.0 is the last version published under the pod name `TLDExtractSwift`).

## Commands

```bash
# Build and test (SPM)
swift build
swift test

# Run a single test method
swift test --filter TLDExtractSwiftTests/<testMethodName>

# Lint (enforced in CI via swift-format, not SwiftLint)
swift-format lint --ignore-unparsable-files --configuration .swift-format --recursive Sources Tests

# Update the bundled Public Suffix List (rewrites Resources/public_suffix_list.dat)
python update-psl.py

# Fastlane lanes (require `bundle install` first; Ruby/Python managed by mise)
bundle exec fastlane tests            # xcodebuild tests on all platforms (macOS/iOS/tvOS/watchOS/visionOS) + slather coverage
bundle exec fastlane lint_swift       # swift-format lint
bundle exec fastlane build_spm        # swift build + test
bundle exec fastlane build_carthage   # Carthage builds per platform
bundle exec fastlane gen_docs         # DocC static site into docc-site/ (via scripts/gen_docs.sh)
bundle exec fastlane set_version      # prompt for version; sets MARKETING_VERSION in the pbxproj
bundle exec fastlane set_version version:4.0.2   # non-interactive form
bundle exec fastlane bump_version     # patch/minor/major bump; same effect
bundle exec fastlane bump_version type:patch     # non-interactive form

# Interactive job menu (fzf)
./run.sh
```

Xcode-based test for a specific platform (what CI runs):

```bash
xcodebuild -project TLDExtractSwift.xcodeproj -scheme TLDExtractSwift -destination "platform=macOS" clean test
```

Note: `swift test` requires network access — the test suite initializes `TLDExtract(useFrozenData: false)`, which downloads the live PSL (see below).

## Architecture

Source files in `Sources/`:

- `TLDExtract.swift` — the public API: `TLDExtract` class (`init(useFrozenData:)` loads and parses the PSL; `parse(_:quick:)` returns a `TLDResult?`), the `TLDExtractable` protocol with `URL`/`String` conformances (hostname extraction via regex), and the `TLDResult` struct (`rootDomain` / `topLevelDomain` / `secondLevelDomain` / `subDomain`).
- `Parser.swift` — `PSLParser` (parses raw PSL text into a `PSLDataSet` of exceptions / wildcards / normals) and `TLDParser` (`parseExceptionsAndWildcards` matches rule-based entries; `parseNormals` matches plain suffixes by iterating host components from the right).
- `Model.swift` — `PSLDataSet`, `PSLData` (one PSL rule: exception flag, dot-split parts, priority for rule precedence), `PSLDataPart` (`.wildcard` / `.characters`).
- `SPMPSL.swift` — `SPM_PSL`, a frozen copy of the PSL embedded as a Swift string literal (~10k lines, compiled only under `SWIFT_PACKAGE`).
- `Extension.swift` — internal helpers (`Bundle.current`, `String.isComment`).
- `TLDExtractError.swift` — `TLDExtractError.pslParseError`.
- `TLDExtractSwift.h` — umbrella header for framework (non-SPM) builds.

### PSL data loading (two build paths)

`TLDExtract.init` branches on `#if SWIFT_PACKAGE`:

- **SPM build**: `useFrozenData: true` uses the embedded `SPM_PSL` string; `false` (default) synchronously downloads https://publicsuffix.org/list/public_suffix_list.dat at init. During parsing, each PSL line is additionally registered in its IDNA-encoded form via PunycodeSwift (`line.idnaEncoded`).
- **Framework build** (Xcode/Carthage): loads `Resources/public_suffix_list.dat` (or `public_suffix_list_frozen.dat` when `useFrozenData: true`) from the framework bundle via `Bundle.current`. No IDNA pass at parse time — punycoded rules are pre-baked into the `.dat` file.

`update-psl.py` downloads the latest PSL, strips comments and blank lines, inserts a punycode-encoded variant after each internationalized rule, and writes the result to all three bundled copies: `Resources/public_suffix_list.dat`, `Resources/public_suffix_list_frozen.dat`, and the `SPM_PSL` literal in `Sources/SPMPSL.swift` (substituted in place, so the file's header is preserved). The three therefore hold identical data; `useFrozenData` selects between a bundled snapshot and a live download only on the SPM path.

`.github/workflows/update-psl.yml` runs the script weekly (Monday 03:00 UTC, plus manual dispatch). When the list changed it runs `swift test` in the `swift:6.2` container against the refreshed data and opens a pull request against `develop` only if those tests pass. That in-workflow run is the real gate: a pull request opened with `GITHUB_TOKEN` does not trigger workflows, so the pull request itself usually shows no checks (close and reopen it to start them, or set the `PSL_UPDATE_TOKEN` secret to a PAT — the workflow prefers that secret when present).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [futamura/TLDExtractSwift](https://github.com/futamura/TLDExtractSwift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
