---
trigger: always_on
description: PHP DSL that defines cross-platform UIs declaratively and generates native source code for 11 platforms (SwiftUI, HTML, Android XML, Jetpack Compose, GTK4, WinUI, Wasm, ArkTS/HarmonyOS, Glance, Wear Tiles, Flutter). Codegen only — no runtime.
---

# PERRY PHP — UI Codegen Framework (Port of perry-ts)

## OVERVIEW

PHP DSL that defines cross-platform UIs declaratively and generates native source code for 11 platforms (SwiftUI, HTML, Android XML, Jetpack Compose, GTK4, WinUI, Wasm, ArkTS/HarmonyOS, Glance, Wear Tiles, Flutter). Codegen only — no runtime.

This is the **UI codegen layer** port of perry-ts (Rust TS→native compiler). Perry-ts is a full compiler pipeline (31 crates, 334k LOC); perry-php focuses exclusively on the UI definition + codegen portion (~19k PHP LOC).

**Status:** 979 tests, 3691 assertions — all passing ✅ | 11 backends, 36 style properties, 16 widgets, 97+ PHP function mappings

## STRUCTURE

```
src/
├── App.php       # Entry point (setRoot, generateCode, generateForTarget)
├── Build/        # Build pipeline orchestration
├── Codegen/      # 11 platform code generators (SwiftUI, Html, AndroidXml, WinUI, Gtk4, Compose, Wasm, ArkTs, Glance, WearTiles, Flutter)
├── Generator/    # Language-specific code generators (Swift, Kotlin, Dart, JS, C#)
├── IR/           # PHP AST→IR (for Closure transpilation), 54 node types
└── UI/           # DSL components
    ├── Platform/   # Platform-specific drivers (macOS, iOS, Android, GTK4, Windows, Web)
     ├── Styling/    # Style system (Style::make()->fontSize()), 29 properties, platform matrix
    └── Widget/     # Widget class hierarchy (16 widgets: VStack, HStack, Button, Text, etc.)
```

## PORT STATUS (vs perry-ts)

### Architecture Comparison

| Area | perry-ts (Rust) | LOC | perry-php (PHP) | LOC | Completeness |
|------|----------------|-----|-----------------|-----|-------------|
| Compiler pipeline | parser, types, HIR, LLVM codegen, transforms, dispatch, diagnostics | 204k | **Not ported** | 0 | **0%** |
| Runtime/Stdlib | GC, NaN-boxing, stdlib (Node APIs), JS runtime | 161k | **Not ported** | 0 | **0%** |
| UI Codegen (Rust) | perry-codegen-{swiftui,js,wasm,arkts,glance,wear-tiles} | 57k | Codegen/ + Generator/ + IR/ | 7.9k | **~14%** |
| UI Widget abstraction | perry-ui (6 rs, 1.5k LOC) | 1.5k | UI/Widget/* + Widget.php | 2.0k | **100%+** (more widgets) |
| CLI / Build | perry crate (25+ commands) | 33k | bin/perry + Build/ | 1.1k | **~3%** |
| Tests | — | — | tests/ | 10.0k (914 tests, 3136 assertions) | **Growing** |
| Native platform bindings | perry-ui-{macos,ios,android,gtk4,windows,visionos,watchos,tvos} | 31k | Codegen/* (generates source instead) | — | **N/A** (different approach) |
| **TOTAL** | **31 crates** | **334k** | **src/** | **19.1k** | **UI layer only** |

### Overall Port Completion

| Layer | Scope | Ported | Status |
|-------|-------|--------|--------|
| **UI Widget DSL** | 16 widgets, 29 style properties, 11 codegen backends | ✅ 100% | Production-ready |
| **Closure Transpilation** | PHP closure → AST → IR → 5 target languages | ✅ Core done | 97+ PHP function mappings |
| **IR System** | 54 node types, 90 interface methods | ✅ 100% | All generators implemented |
| **Build Pipeline** | Compiler, linker, toolchain integration | ⚠️ Stubs | Missing real toolchain |
| **Compiler** | Parser, type system, HIR, LLVM codegen | ❌ 0% | Out of scope |
| **Runtime** | GC, NaN-boxing, stdlib | ❌ 0% | Out of scope |

### Codegen Backend Comparison

| Backend | perry-ts | perry-php | Notes |
|---------|----------|-----------|-------|
| SwiftUI | ✅ perry-codegen-swiftui (3.0k LOC) | ✅ SwiftUIBackend.php (627 loc) | Full app generation with state, actions, styling |
| JavaScript | ✅ perry-codegen-js (8.2k LOC) | ✅ HtmlBackend.php (604 loc) | HTML+CSS+JS vs perry-ts raw JS |
| Jetpack Compose | Via perry-ui-android (native) | ✅ ComposeBackend.php (490 loc) | Unique to PHP |
| Android XML | ❌ | ✅ AndroidXmlBackend.php (888 loc) | Unique to PHP, most LOC backend |
| GTK4 | Via perry-ui-gtk4 (native) | ✅ Gtk4Backend.php (615 loc) | Codegen vs native bindings |
| WinUI | Via perry-ui-windows (native) | ✅ WinUIBackend.php (874 loc) | Codegen vs native bindings |
| WASM | ✅ perry-codegen-wasm (20.4k LOC) | ✅ WasmBackend.php (622 loc) | Generates HTML+JS with perry_ui_* bridge API |
| ArkTS/HarmonyOS | ✅ perry-codegen-arkts (20.2k LOC) | ✅ ArkTsBackend.php (495 loc) | Full ArkUI codegen with @State bindings |
| Glance | ✅ perry-codegen-glance | ✅ GlanceBackend.php (447 loc) | Kotlin Glance composables for home screen widgets |
| Wear Tiles | ✅ perry-codegen-wear-tiles | ✅ WearTilesBackend.php (406 loc) | Kotlin Wear OS TileService builder API |
| Flutter | ❌ | ✅ FlutterBackend.php (608 loc) | Flutter Material Design widgets (unique to PHP) |

### Generator Language Coverage

| Language | Lines | IR Methods | PHP Mappings | Tests | Notes |
|----------|-------|-----------|-------------|-------|-------|
| Swift | 853 loc | 90/90 (100%) | 97+ | 19 | Full IR, expanded PHP mappings (P5+P6+P7) |
| Kotlin | 861 loc | 90/90 (100%) | 97+ | 22 | Full IR, expanded PHP mappings (P5+P6+P7) |
| Dart | 869 loc | 90/90 (100%) | 97+ | 22 | Full IR, expanded PHP mappings (P5+P6+P7) |
| JavaScript | 871 loc | 90/90 (100%) | 97+ | 43 | Full IR, expanded PHP mappings (P5+P6+P7) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangweijie/perry-php](https://github.com/yangweijie/perry-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
