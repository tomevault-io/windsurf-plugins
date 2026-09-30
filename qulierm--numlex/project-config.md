---
trigger: always_on
description: Orientation and operating guide for coding agents. Applies to the **whole
---

# AGENTS.md — working in the Numlex repository

Orientation and operating guide for coding agents. Applies to the **whole
repository** unless a future nested `AGENTS.md` overrides it (closest file wins
for its subtree).

## Read this first

- Read the code before changing it; the source and tests are the primary
  sources of truth. Change the smallest correct surface.
- Never release, bump the version, move a tag, edit release assets, publish an
  appcast, or push the Homebrew tap without explicit, per-task authorization
  (section 3.7).
- `git status` first — and again before committing. Two untracked items
  (`Examples/`, root file `Numlex`) are intentional and must never be
  added, modified or deleted.
- Prefer existing precedent: most contracts are pinned by tests, usually pure
  engine cases or source-contract cases.

## Snapshot (2026-09-25)

These facts **age** — re-verify with git and the release endpoints before
relying on them, especially before any release task.

| Fact | Value |
| --- | --- |
| App version (`Sources/NumlexApp/Resources/Info.plist`) | `4.9.5` (`CFBundleShortVersionString` = `CFBundleVersion`) |
| App release commit | `43b09398d9d757506946b318145e35ac61c9786c` (release commit of annotated tag `4.9.5`) |
| Current release | GitHub [Qulierm/Numlex `4.9.5`](https://github.com/Qulierm/Numlex/releases/tag/4.9.5), annotated tag `4.9.5` (peels to `43b0939`), release id `RE_kwDOMGYWT84Xoloc`, published `2026-09-25T10:34:41Z` |
| Release DMG | `Numlex-4.9.5-macOS-arm64.dmg`, 10,796,720 bytes, SHA-256 `47228bb2ce0e45be294b529a5b879df2b51d07a7d9c34ea898176729c48861b9` |
| `SHA256SUMS` asset | 95 bytes, digest `sha256:c3506f90aa034013f1f45bc5cc53743020929febd95aa141f9690dfdb2647306` |
| Engine suite | 1429/1429 (observed 2026-09-25; runner output is authoritative) |
| Website (`../NumlexWeb`) HEAD | `01d0650822d818fbd669cd54ecd7b9a79d3f1abf` (private repo) |
| Homebrew tap HEAD | `011bedf8006c9b4766d019efc62e6d814a177584` |
| Release notes / feed | <https://numlex.tech/Numlex-4.9.5-macOS-arm64.md> · <https://numlex.tech/appcast.xml> |
| Previous release (historical) | `4.9.4.1` at `92db3d3d5f4a2787610c6d086cbb68927f91db31`, DMG `744fc564df6802db8ebb6fa9d3da5081a28d17b8a73593b5e4b006497ea7b4ee` (10,792,648 bytes) — immutable |

Verify with: `git rev-parse HEAD && git ls-remote origin main`,
`gh release view 4.9.5 --repo Qulierm/Numlex --json assets`,
`swift run NumlexTests | tail -1`.

---

# 1. Project scope and repository map

## 1.1 What Numlex is

A **native macOS notepad calculator**: plain-text lines evaluate as you type,
with results in a separate answer column.

- **Native**: SwiftUI shell + Swift 6.2 (package tools-version 6.0); the editor
  core is an AppKit `NSTextView` bridged via `NSViewRepresentable`. No web
  view, no Electron, no JavaScript, no runtime scripting.
- **arm64, macOS 26+**, ad-hoc signed and **not notarized** (deliberate).
- **Offline-first**: calculations and catalogs are local, from versioned
  hash-checked resource bundles. Network is limited to background currency
  rates and the Open-Meteo lookups that explicit `weather in …` /
  geography lines request.
- **Deterministic**: strict typed parser/evaluator, no silent coercion.

## 1.2 Package products, targets and dependencies

| Product | Kind | Path | Role |
| --- | --- | --- | --- |
| `NumlexApp` | executable | `Sources/NumlexApp` | The macOS app (SwiftUI + AppKit bridge, settings, export UI). Links Sparkle. |
| `NumlexCore` | library | `Sources/NumlexCore` | Pure dependency-free engine: parser, evaluator, models, catalogs, persistence, services, export. |
| `NumlexTestKit` | library | `Tests/NumlexTestKit` | Portable `EngineCase` arrays shared by both runners. |
| `NumlexCoreTests` | test target | `Tests/NumlexCoreTests` | Swift Testing suite (`swift test`, full Xcode toolchain). |
| `NumlexTests` | executable | `Tests/NumlexTests` | Standalone runner (`swift run NumlexTests`, CLT-only friendly). |

The only external dependency is **Sparkle, pinned exactly to `2.9.6`**; only
`NumlexApp` links it and the core stays dependency-free. `Package.resolved` is
gitignored.

`NumlexCore` declares five `.copy` resource directories under
`Sources/NumlexCore/Resources/` (`NumlexTimezones`, `NumlexHolidays`,
`NumlexIncomeTax`, `NumlexCPI`, `NumlexTax`). `NumlexApp` copies three icon
resources and excludes `Resources/Info.plist` / `Resources/AppIcon.icns` from
its bundle (consumed by `Scripts/build-app.sh` instead).

## 1.3 Directory and file map

| Path | Role |
| --- | --- |
| `Package.swift` | Products, targets, resources, Sparkle pin, test-toolchain wiring. |
| `Sources/NumlexApp/Entry.swift` | The real `@main` (`NumlexEntry`): `--validate-packaged-resources` or `NumlexApp.main()`. |
| `Sources/NumlexApp/NumlexApp.swift` | SwiftUI `App` scene, `AppDelegate`, welcome/curtain reveal stages, menus. |
| `Sources/NumlexApp/AppModel.swift` | The one observable state: sheets, selection, settings, folders, rates, weather/geo, random epochs, persistence, appearance/icon. |
| `Sources/NumlexApp/Editor/NotebookEditor.swift` | AppKit bridge: `NSTextView`/TextKit, editing hooks, token insertion, measurements, scroll/bounds. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qulierm/Numlex](https://github.com/Qulierm/Numlex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
