---
trigger: always_on
description: **macOS 26 (Tahoe) ONLY.** This is non-negotiable.
---

# Deconstructed - Agent Instructions

## Critical Constraints

**macOS 26 (Tahoe) ONLY.** This is non-negotiable.

- No iOS, iPadOS, visionOS, watchOS, tvOS
- No backwards compatibility (no macOS 15, 14, etc.)
- No `#available` checks or `@available` attributes
- No multi-platform conditionals

### Forbidden APIs (iOS-only, do not use)

- `DocumentGroupLaunchScene`
- `DocumentLaunchView`
- `UIKit` anything
- Any API marked "iOS only" or "iPadOS only" in documentation

### Required APIs (use these)

- `AppKit` / `NSApplication` / `NSWindow` for macOS-specific needs
- `SwiftUI` with macOS idioms (Window scenes, Settings scenes, NSOpenPanel)
- Swift 6.2 concurrency (MainActor isolation by default)
- TCA 1.23.1 modern patterns: `@ObservableState`, `StoreOf`, `@Bindable` (no `WithViewStore`)
- On macOS 26, Observation is native; do not use `WithPerceptionTracking` unless targeting older OSes.

## Project Purpose

Reverse-engineer and clone Reality Composer Pro's functionality:
1. Open `.realitycomposerpro` package files
2. Parse and display project structure
3. Edit scenes and assets
4. Save changes back to the package format

## Dependency Resolution: Workspace vs Package.swift

### The Problem

This project has a split identity:
- **Public open-source repo** (`Deconstructed`) — anyone should be able to clone and build
- **Separate private workflow repo** (`USDTools`) — may exist locally for internal work, but is not required for the public Deconstructed build path

The root `Package.swift` and inner `Packages/DeconstructedLibrary/Package.swift` declare remote URLs for CI/public consumption. But locally, Xcode resolves dependencies at the **workspace level**, overriding what `Package.swift` says.

### How It Actually Works

**All dependency resolution happens through `Deconstructed.xcworkspace`.** The workspace includes local package references that Xcode prefers over remote URLs with the same identity.

This means:
- `Package.swift` remote URLs are **fallbacks for CI / clean clones only**
- The inner `DeconstructedLibrary/Package.swift` may reference `branch: "main"` or pinned revisions — **it doesn't matter locally** because the workspace overrides them
- Editing local first-party package checkouts such as `/Volumes/Plutonian/_Developer/USDInterop` compiles immediately when the workspace is configured to use them
- **You must open `Deconstructed.xcworkspace`**, not the `.xcodeproj`

### Do NOT Try to Build via `swift build` in Inner Package

Running `swift build` inside `Packages/DeconstructedLibrary/` will fail because:
1. SwiftPM resolves deps from `Package.swift` directly (no workspace override)
2. The inner package may point to `branch: "main"` while root pins a revision — SwiftPM cannot reconcile two different revision-based requirements for the same package
3. Local-only packages (like `SelectionOutline` as a sibling) resolve fine, but remote deps conflict

**Always build through Xcode workspace or from the root `Package.swift`:**
```bash
# From repo root — uses root Package.swift which is kept CI-safe
swift build --target ViewportUI
```

### For Agents / CI

- The root `Package.swift` is the source of truth for public/CI builds
- It pins stable versions of remote deps (revisions or semver)
- Local packages (`SelectionOutline`, etc.) use relative paths that work from root
- Never edit the inner `DeconstructedLibrary/Package.swift` dependency URLs to match root — they serve different purposes

## USD Boundary

The current split is:

- public package family: `USDInterop`, `USDInterfaces`, `USDInteropCxx`, `USDOperations`
- private workflow/value layer: `USDTools`

Rules:

- generic scene operations belong in `USDOperations`
- workflows, diagnostics, repair, packaging, conversion, and heuristics do not
- do not reintroduce dependencies on `USDTools` or legacy advanced modules into the public Deconstructed build path

## Reference Implementation

Analyze this real RCP project for format details:
```
/Volumes/Plutonian/_Developer/Deconstructed/references/Base
```

**IMPORTANT**: The document is the **`.realitycomposerpro` bundle**, NOT the parent folder. RCP creates an SPM package wrapper around it for Xcode/Swift integration.

### Package Structure
```
Base/                                      # SPM package (wrapper for integration)
├── Package.swift                          # SPM manifest
├── Package.realitycomposerpro/            # <- THE DOCUMENT (what we open/save)
│   ├── ProjectData/
│   │   └── main.json                      # UUID mappings (paths reference ../Sources/)
│   ├── WorkspaceData/
│   │   ├── Settings.rcprojectdata         # Editor settings (JSON)
│   │   ├── SceneMetadataList.json         # Hierarchy state
│   │   └── <username>.rcuserdata          # Per-user prefs
│   ├── Library/
│   └── PluginData/
└── Sources/                               # Assets (sibling to document)
    └── <ProjectName>/
        ├── <ProjectName>.swift            # Bundle accessor
        └── <ProjectName>.rkassets/
            └── Scene.usda                 # USD scene files
```

### Key Insight
- Double-clicking `.realitycomposerpro` opens RCP
- RCP does NOT display the parent folder structure
- Asset paths in `main.json` like `/Base/Sources/Base/Base.rkassets/Scene.usda` navigate relative to SPM root


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elkraneo/Deconstructed](https://github.com/elkraneo/Deconstructed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
