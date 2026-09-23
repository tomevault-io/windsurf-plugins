---
trigger: always_on
description: Deckle is a focused macOS menu-bar app that overlays deterministic paper texture on every display. Keep it small, native, energy-efficient, and compatible with macOS 13.
---

# AGENTS.md

Deckle is a focused macOS menu-bar app that overlays deterministic paper texture on every display. Keep it small, native, energy-efficient, and compatible with macOS 13.

## Project facts

- Language: Swift 5.9 package
- UI: SwiftUI plus AppKit where window behavior matters
- Minimum platform: macOS 13
- Dependencies: none outside Apple frameworks
- Product shape: menu-bar accessory app; no Dock or app-switcher entry
- Build system: Swift Package Manager plus the root `Makefile`

Run commands from this repository root. Do not assume tools or dependencies from sibling projects in the parent workspace.

## Commands

```sh
swift build                         # Debug compile
swift test                          # Full test suite
swift test --filter PaperComfortTests
swift run                           # Unbundled development run
make app                            # Release build + ad-hoc signed dist/Deckle.app
make build UNIVERSAL=1              # arm64 + x86_64 release compile
make dmg                            # Local DMG
```

Before shipping a code change, run `swift test`. For release-sensitive changes, also run `make build UNIVERSAL=1`: the release workflow uses Xcode 15.4 and builds both architectures, which can expose actor-isolation errors that a local single-architecture build misses.

## Source map

| Area | Files | Responsibility |
|---|---|---|
| App entry and state | `DeckleApp.swift`, `AppState.swift` | MenuBarExtra, application lifecycle, persisted settings, transient preview state |
| Overlay | `OverlayController.swift`, `OverlayWindow.swift` | One click-through retained-mode window per display |
| Rendering | `TexturePreset.swift`, `TextureRenderer.swift` | Versioned texture recipes, spectral/legacy engines, bounded caches |
| Main menu | `MenuView.swift`, `HeroCardView.swift`, `PresetCardView.swift`, `QuickControlsView.swift`, `FeaturePromoCard.swift` | Status, search, paper library, controls, discovery |
| Paper creation | `PaperMill.swift`, `PaperComfort.swift` | Custom paper editing, live preview, comfort estimates, import/export |
| Community and updates | `CommunityBrowser.swift`, `UpdateManager.swift` | Community paper index, download/install, GitHub release updates |
| Automation | `HotKey.swift`, `URLCommands.swift` | Global shortcut and `deckle://` commands |
| Assets and packaging | `Icons.swift`, `Support/Info.plist`, `scripts/GenerateIcon.swift`, `Makefile` | Menu glyph, app metadata, app bundle, DMG |

## Architectural invariants

### State and persistence

- `AppState.shared` is the single source of truth for user-facing settings.
- Persistent properties write through to `UserDefaults` in `didSet`; keep keys stable unless a migration is implemented.
- Transient state must not be persisted. `previewPaper` exists only while Paper Mill previews an unsaved draft.
- `texture` means the saved selection. `effectiveTexture` may temporarily resolve to the Paper Mill preview. Do not conflate them in menu chrome or automation.
- Keep `shouldShowOverlay` tied to enabled/snoozed state. Preview visibility is handled separately by `OverlayController`.

### Overlay windows

- Maintain one `OverlayWindow` per connected display.
- Overlay windows must remain borderless, transparent, click-through, retained-mode, and at `.screenSaver` level.
- Preserve `canJoinAllSpaces`, `.stationary`, `.fullScreenAuxiliary`, and `.ignoresCycle` behavior.
- Per-display exclusions always win, including during Paper Mill preview.
- A Paper Mill preview may override enabled, snoozed, and app-rule visibility so an unsaved paper can be judged on screen.
- Do not allocate display-sized texture bitmaps. `TextureView` must continue using a small tiled layer pattern.

### Renderer compatibility and performance

- Renderer versions are a compatibility boundary:
  - `.legacy` must remain byte-compatible for version-less historical custom papers.
  - `.spectral` remains available for v2 custom papers and compatibility fixtures.
  - `.spectralPlus` (v3) is used by all built-in presets and newly created papers; it layers darkening oriented-fiber bundles (Gabor-modulated) and Perlin surface roughness over the v2 spectrum.
- Stable seeds must round-trip through JSON. Never use Swift `hashValue` for deterministic output.
- Preserve Hermitian symmetry, inverse-FFT normalization, seamless wrapping, and backing-scale behavior in the spectral engine. The v3 pass builds on `spectralField` unchanged; it only darkens the resulting field, so it inherits these invariants.
- Renderer caches are bounded LRUs and intentionally main-thread-only. Do not call them concurrently without redesigning synchronization.
- Cache keys must include every render-relevant input and exclude irrelevant metadata such as a paper name.
- Avoid uncached 2x spectral rendering directly in a parent SwiftUI body that observes unrelated state. Isolate expensive thumbnails in equatable child views, use 1x while dragging, and debounce full overlay pushes.
- Any renderer change needs deterministic tests. Legacy changes also need the pinned byte hash to remain unchanged unless compatibility is intentionally broken.

### Paper Mill


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YellowFoxH4XOR/deckle](https://github.com/YellowFoxH4XOR/deckle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
