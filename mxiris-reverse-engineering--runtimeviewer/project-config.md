---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md
This file provides guidance to coding agents when working with code in this repository.

## Project Overview

Runtime Viewer is a macOS/iOS document-based (NSDocument) application for inspecting Objective-C and Swift runtime interfaces. It serves as a modern alternative to RuntimeBrowser with features like Swift interface support, type-defined jumps, Xcode-style syntax highlighting, code injection capabilities, and MCP (Model Context Protocol) integration for LLM clients.

## Build Commands

**Workspace preference**: Before running any `xcodebuild` / `swift build` / `swift test`, check whether `../MxIris-Reverse-Engineering.xcworkspace` (sibling of this repo) exists. If it does, **use that workspace** via `xcodebuild -workspace ../MxIris-Reverse-Engineering.xcworkspace -scheme <scheme> ...` — it wires this repo together with local checkouts of MachOKit / MachOObjCSection / MachOSwiftSection / swift-capstone / swift-demangling / swift-semantic-string / swift-syntax that may contain in-progress fixes not yet published upstream. Building against the remote SPM resolution can hit stale errors (e.g. the MachOSwiftSection `@Mutex` macro expansion bug) that the workspace's local checkout already fixes. Only fall back to the standalone commands below when the workspace is absent.

**Catalyst helper build order**: For native macOS builds, build
`RuntimeViewerCatalystHelper` first, then build `RuntimeViewer macOS` /
`RuntimeViewerUsingAppKit` in the same Xcode/DerivedData session. Do not model
this as a direct target dependency: Xcode treats the Mac Catalyst helper as
iOS-family embedded content and rejects it from the macOS app target.
`RunScript.sh` and `ArchiveScript.sh` both already handle this by building/
archiving the helper before the main app.

Recommended Xcode order:
1. Build `RuntimeViewerCatalystHelper` for `My Mac (Mac Catalyst)`.
2. Build `RuntimeViewer macOS` for `My Mac`.

```bash
# Debug build + launch (configuration "Debug-arm64e", workspace
# RuntimeViewer-Debug.xcworkspace, scheme "RuntimeViewer macOS"; builds
# RuntimeViewerCatalystHelper first, then the main app). This is the only
# working path for Debug-arm64e — the Xcode GUI fails to compile under
# iOSPackagesShouldBuildARM64e=true. Product: RuntimeViewer-Debug-arm64e.app
# under /Volumes/DerivedData/RuntimeViewer/Debug-arm64e when that volume
# exists (project-relative DerivedData otherwise).
./RunScript.sh
./RunScript.sh --no-launch         # build only
./RunScript.sh --update-packages   # refresh SPM pins before building
./RunScript.sh --dry-run           # print commands without running

# Release build (archives Catalyst helper + main app, notarizes, and optionally
# generates appcast + uploads GitHub Release). Uses scheme "RuntimeViewer macOS".
# Omit the distribution flags for a local signed zip only.
./ArchiveScript.sh
# Cut a full release (appcast + GitHub Release + commit docs/appcast.xml):
./ArchiveScript.sh --update-appcast --upload-to-github --commit-push --version-tag vX.Y.Z

# Build RuntimeViewerServer XCFramework (all platforms)
./BuildRuntimeViewerServerXCFramework.sh
```

**Workspaces**: `RuntimeViewer-Debug.xcworkspace` (used by `RunScript.sh`)
already wires the local sibling checkouts of MachOKit / MachOObjCSection /
MachOSwiftSection / swift-demangling / swift-semantic-string plus the
precompiled swift-syntax, so Debug builds pick up in-progress fixes in those
repos without touching remote SPM pins. `RuntimeViewer-Distribution.xcworkspace`
serves release archives.

**Build Schemes**:
- `RuntimeViewer macOS` — main app; Debug-arm64e via `RunScript.sh`, Release archives via `ArchiveScript.sh`
- `RuntimeViewerCatalystHelper` — Mac Catalyst helper, always built before the main app
- `RuntimeViewerUsingAppKit` — plain Debug builds of the AppKit app target

## Architecture

### Package Structure

The project uses three Swift Package Manager packages:

**RuntimeViewerCore** (`RuntimeViewerCore/`):
- `RuntimeViewerCore` — Runtime inspection engine using MachOObjCSection (ObjC) and MachOSwiftSection (Swift)
- `RuntimeViewerCommunication` — XPC/TCP-based IPC layer for cross-process inspection
- `RuntimeViewerCoreObjC` — Objective-C interop utilities (internal target)

**RuntimeViewerPackages** (`RuntimeViewerPackages/`):
- `RuntimeViewerArchitectures` — MVVM + Coordinator pattern with RxSwift
- `RuntimeViewerApplication` — ViewModels and business logic (Sidebar, Inspector, Content, Theme, FilterEngine)
- `RuntimeViewerUI` — AppKit UI components (MinimapView, StatefulOutlineView, skeleton effects)
- `RuntimeViewerService` — XPC service helpers and code injection
- `RuntimeViewerServiceHelper` — Helper utilities
- `RuntimeViewerHelperClient` — Helper client for XPC communication
- `RuntimeViewerSettings` — Settings models and dependency values
- `RuntimeViewerSettingsUI` — Settings UI (SwiftUI)
- `RuntimeViewerCatalystExtensions` — Mac Catalyst support

**RuntimeViewerMCP** (`RuntimeViewerMCP/`) — MCP integration (macOS 15+ only):
- `RuntimeViewerMCPShared` — Shared protocols and transport types
- `RuntimeViewerMCPBridge` — Bridge server that runs inside the main app

### Application Targets

- `RuntimeViewerUsingAppKit` — Main macOS application (AppKit, document-based)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MxIris-Reverse-Engineering/RuntimeViewer](https://github.com/MxIris-Reverse-Engineering/RuntimeViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
