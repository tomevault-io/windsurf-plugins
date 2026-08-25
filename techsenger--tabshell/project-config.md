---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Techsenger ShellFX is a Java/JavaFX platform for building applications structured as a tree of MVP
(Model-View-Presenter) components (window, tab, area, page, dialog, popup), each with its own lifecycle and
history. It is built on top of [PatternFX](https://github.com/techsenger/patternfx) (`patternfx-mvp`,
`patternfx-core`), which provides the underlying MVP component/tree machinery. ShellFX deliberately stays on
MVP — do not propose MVVM, StateFX, or property-binding patterns for presenters.

Requires Java 25 and JavaFX 26. Multi-module Maven build, parent POM inherits from `com.techsenger.maven.root`.

## Commands

```
mvn clean install          # build all modules (runs Checkstyle automatically via parent POM)
cd shellfx-demo && mvn javafx:run   # run the demo app (debugger settings are in shellfx-demo/pom.xml)
mvn test -pl shellfx-material       # run tests for a single module
mvn test -pl shellfx-material -Dtest=ColumnListViewTest   # run a single test class
```

Checkstyle runs as part of `mvn install`/`mvn verify` (results land in `target/checkstyle-result.xml` per
module) — a build failure may be a style violation, not just a compile error.

Only some modules currently have tests: `shellfx-material`, `shellfx-layout`, `shellfx-storage`. UI tests that
need a real `Stage` (e.g. `shellfx-material/.../column/*Test.java`) run headless via
`glass.platform=Headless` + `prism.order=sw`, started once through a shared `FxTestSupport`/`FxPlatform.start()`
helper and executed on the FX Application thread with `FxPlatform.runLaterAndWait`. When such a test asserts on
measured geometry, a real AtlantaFX user-agent stylesheet must be applied first, otherwise CSS `-color-*`
lookups silently fall back to defaults and mask regressions.

## Module structure

Modules and their dependency direction (all depend on `shellfx-material`, which depends on nothing else
in-repo):

```
material  (base UI elements, no shellfx deps)
  ├── core      (Shell, Window, Tab, Page, Dialog, Popup, registries, settings)
  ├── shared    (depends on core; FindBase/FindPanel used by other modules)
  ├── layout    (depends on core, shared; TabHost, DockHost, PageHost, TreePageHost)
  ├── storage   (depends on core; file-system abstractions)
  ├── dialogs   (depends on core, shared, storage; AlertDialog, FileChooserDialog, NameValueDialog)
  ├── devtools  (depends on core, shared, dialogs, layout; component-tree/scene-graph inspectors)
  ├── icons     (depends on material; Material Design Icons font + stylesheets)
  └── demo      (depends on everything; showcase app, not published — see publishing.plugin.exclusions)
```

Every module is a JPMS module (`module-info.java` under `src/main/java`); when adding a new public package,
remember to add both the `exports` (and `opens` for CSS/FXML-loaded packages) in `module-info.java`.

## Architecture

- **Component tree + scene graph are two parallel hierarchies.** Every component addition/removal must be
  reflected in both. Removing a node from the JavaFX scene graph without removing it from the component tree
  leaks memory. DevTools (`shellfx-devtools`) can inspect both trees live.
- **Lifecycle is explicit and developer-controlled.** Component init/deinit happens in `Composer` methods, not
  automatically — see Naming Convention below.
- **Core interfaces vs. base implementations.** Each component is an interface + a default `Abstract*`
  implementation (e.g. `ShellFxView` interface backing `Shell`). Code should reference the interface, not the
  concrete class, matching the platform's own convention (`ShellFxView`, not `DefaultShellFxView`).
- **Menu system.** The main menu is assembled dynamically at runtime by `ControlRegistry` from
  registered/unregistered menu, group, and item factories (supports plugin-style dynamic contribution).
  `MenuManager` tracks the focused component via `Scene#focusOwnerProperty()`, walks up the component tree to
  find the nearest ancestor implementing `MenuAwarePort`, and dispatches state/actions to it. A component that
  should focus on click of an empty area must call `requestFocus()` explicitly.
- **Windows.** `Window` comes in `NESTED` (managed by `WindowManager`, hosted inside a `HostWindow` or
  `HostTab`) and `TOP_LEVEL` (own OS `Stage`) variants, both accessed through the same API — dialogs/wizards
  built on `Window` work unmodified in either mode.
- **DockHost** (in `shellfx-layout`) has a whole-tree API (`ModelNode`/`GroupNode`/`AreaNode` built via
  `ModelNodeBuilder`, applied/captured via `Composer#applyModel`/`captureModel`) for full layout
  construction/restoration, and a partial-tree API (anchors resolved live via `Composer#getModelNode(AreaFxView)`)
  for incremental runtime changes (add-next-to, replace, remove, user-driven docking).

## Language

Everything in the project is written in English — README, documentation, Javadoc, code comments, commit
messages, etc. Always — regardless of what language the conversation with the assistant happens in.

## Member ordering


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techsenger/tabshell](https://github.com/techsenger/tabshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
