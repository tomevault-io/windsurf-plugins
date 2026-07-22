---
trigger: always_on
description: This guide helps contributors work effectively on the React4j codebase.
---

# Repository Guidelines

This guide helps contributors work effectively on the React4j codebase.

## User Interaction

When asked to perform a task, ask the user questions one at a time until you have enough context. Feel free to make
reasonable assumptions based on patterns present in the code and ask the user to confirm the assumptions if there are
reasonable alternatives.

## Notes for the Agent

When you learn something non-obvious, add it here if it would make future changes faster or of higher quality.

- Render-prelude observation no longer comes from `@Input`. For `TRACKING` and `MAYBE_TRACKING` views, React4j scans subclass-accessible declared/inherited instance fields and zero-arg instance methods annotated with `@ComponentDependency` or `@AutoObserve`; known nullable candidates use `ComponentObservable.maybeObserve(...)`, uncertain types use a runtime `instanceof ComponentObservable` guard, and `@CascadeDispose` is ignored.
- `bundle exec buildr test` includes `downstream-test` builds of external example repos pinned to their GitHub `master` branches via `tasks/util.rb`, so breaking annotation/API changes can fail there until those downstream repos are migrated even when local processor and api-diff tests pass.
- Explicit `@View` constructors now emit a suppressible `React4j:ConstructorParameterOrder` warning unless parameters are grouped as non-`@Input`, then `@Input(fromTreeContext = true)`, then other `@Input`; the diagnostic is attached to the constructor and uses the compact labels `inject`, `tree`, and `input`.
- Generated view builder visibility is now controlled solely by `@View(exportBuilder = true|false)`: `false` keeps the top-level builder class, `newBuilder()`, static entry methods, and step interfaces package access, while `true` makes that builder API public; public `@View` classes themselves now emit suppressible `React4j:PublicView` warnings.
- Generated native view `displayName` values trim a trailing `View` from the effective `@View.name`, but only for that React debug name; builder keys, generated class names, and other descriptor-name uses still keep the full view name.
- Processor fixture expectations for generated builders are split between `React4jProcessorTest.deriveExpectedOutputs()` for most cases and a few dedicated nested/inheritance tests with hardcoded builder filenames, so builder naming changes usually require updates in both places.

## Project Structure & Module Organization

- Java modules:
  - `core/` (React4j runtime + annotations)
  - `dom/` (React DOM bindings, DOM props/events)
  - `processor/` (annotation processor that generates view/builder/factory code)
  - `api-test/` (Revapi diff tests against previous releases)
  - `downstream-test/` (builds/validates downstream example apps and collects stats)
  - `doc-examples/` (GWT examples embedded into the docs site)
- Docs and site: `docs/` (Markdown), `website/` (Docusaurus v1), `assets/` (icons/favicons, api-diff assets).
- Build configuration: `buildfile` (Buildr), `build.yaml` (artifact coordinates), `tasks/*.rake` (CI, GWT, release, docs).
- Dependency coordinates are centralized in `build.yaml`; update there when adding or upgrading libraries.
- Source layout: `*/src/main/java/...`; tests: `*/src/test/java/...`.
- Generated or local-only directories: `target/`, `reports/`, `node_modules/` should not be edited or committed.
- Encrypted secrets live in `etc/secrets`; do not edit or attempt to decode.

### Module-specific notes

- `core/`
  - Core APIs live under `react4j.*` with internal helpers in `react4j.internal`.
  - GWT module is `core/src/main/java/react4j/React.gwt.xml` with compile-time properties set in `react4j.ReactConfig`.
  - `ReactDev.gwt.xml` sets development defaults and serves assets from `react4j/public/dev`.
  - Uses Grim `@OmitPattern`/`@KeepPattern` in `core/src/main/java/react4j/package-info.java` to trim output.
  - JS assets (React 16.6.0) are checked in under `core/generated/js_assets` and patched to expose `React.Element`.
- `dom/`
  - DOM bindings and ReactDOM live under `react4j.dom.*`; event wrappers under `react4j.dom.events`.
  - GWT modules are `Dom.gwt.xml` and `DomDev.gwt.xml` in `dom/src/main/java/react4j/dom`.
  - `react4j.dom.DOM` is generated from `tasks/react_dom.rake` into `dom/generated/reactgen/main/java/react4j/dom/DOM.java`.
  - JS assets (react-dom 16.6.0) live under `dom/generated/js_assets`.
- `processor/`
  - Annotation processor targets `@react4j.annotations.View` and generates `React4j_*` types plus builders/factories.
  - Integrates with Arez and Sting annotations (see `react4j.processor.Constants`).
  - Processor options include `react4j.defer.unresolved`, `react4j.defer.errors`, `react4j.debug`, `react4j.profile`,
    `react4j.verbose_out_of_round.errors`.
  - The processor jar shades `javapoet` and `proton` under `react4j.processor.vendor.*` (see `buildfile`).
  - Tests use fixtures under `processor/src/test/fixtures/{input,expected,bad_input}`.
- `api-test/`
  - Uses `revapi-diff` to compare previous/current core+dom jars; fixtures under
    `api-test/src/test/resources/fixtures`.
- `downstream-test/`
  - Uses Gir to clone/build downstream apps and collect build statistics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [react4j/react4j](https://github.com/react4j/react4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
