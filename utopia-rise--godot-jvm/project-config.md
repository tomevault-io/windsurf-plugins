---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Godot-JVM** is a Godot engine module that enables Kotlin (and Java/Scala) as scripting languages. It is a hybrid C++/JVM project: the C++ side integrates with Godot's module system, and the Kotlin/Gradle side provides the runtime libraries and tooling for user projects.

Current binding version: `0.17.1` targeting Godot `4.7.2`.

## Engineering Principles

- **Avoid needless helpers** — Keep trivial single-use code inline. Extract a helper only when it has multiple call sites or its body is substantial enough to make the caller clearer.
- **KISS** — Prefer the simplest design that clearly solves the current problem.
- **YAGNI** — Do not add behavior or abstractions for requirements that do not exist yet.
- **DRY** — Keep each piece of knowledge and behavior in one authoritative place.

## Prerequisites

- **JDK 17+** required for building IDE/Gradle plugins (JDK 11+ for runtime-only builds)
- Use **Adoptium/Eclipse Temurin** JDK — Microsoft JDK causes IDE plugin build failures (known issue [microsoft/openjdk#339](https://github.com/microsoft/openjdk/issues/339)). If you must use Microsoft JDK, manually create the `Packages` folder inside `JAVA_HOME`.
- `JAVA_HOME` must be set
- Standard Godot build deps: SCons, Python, C++ compiler

## Build Commands

The project has two independent build systems that must both be built.

### C++ Module (SCons)

This module lives as a submodule at `modules/kotlin_jvm/` inside the Godot source tree. All SCons commands run from the **Godot source root**.

```bash
scons platform=linuxbsd target=editor             # Linux editor
scons platform=windows target=editor              # Windows editor
scons platform=linuxbsd target=template_release   # export template
scons p=linuxbsd -j$(nproc) debug_symbols=yes dev_build=yes  # debug build (multi-core)
```

### Kotlin/Gradle

```bash
cd kt/
./gradlew build                  # build all subprojects
./gradlew build -Prelease        # production/release build
./gradlew publishToMavenLocal    # publish artifacts locally for branch testing
./gradlew publishArtifactsToMavenLocal  # publish all local-consumption artifacts to mavenLocal
```

Full local publish (for testing unreleased branch changes in a user project):
```bash
cd kt/
./gradlew publishArtifactsToMavenLocal
# Version will appear in ~/.m2/repository/com/utopia-rise/godot-gradle-plugin/
```

### Template Generation (Python)

Run after modifying any `.template` or `.godot_template` files under `kt/plugins/godot-intellij-plugin/src/main/resources/template/`:

```bash
python generate_templates.py
```

This converts templates into base64-encoded C++ headers at `src/editor/project/templates.h` (split into 8KB chunks to avoid C++ header size limits), then rebuild the C++ module.

## Testing

```bash
# Kotlin unit tests
cd kt/ && ./gradlew test

# Integration tests (GUT-based, requires a built editor binary)
cd harness/tests/
jlink --add-modules java.base,java.logging --output jvm/jre-amd64-linux  # create JRE first
./gradlew runGutTests
```

The `harness/tests/` directory is a full Godot project. It requires a built editor binary and `godot-bootstrap.jar` copied to the Godot root `bin/` folder before running.

### Testing Changes from a Feature Branch

1. Publish locally (see above)
2. Configure the user project's Gradle repositories to use `mavenLocal()` and use the exact snapshot version you published (e.g. `0.17.1-4.7.2-d68f299-SNAPSHOT`)
3. Run with the dev build: `./bin/godot.linuxbsd.editor.dev.x86_64.jvm` (or platform equivalent)

Full workflow: `docs/src/doc/contribution/test-change-from-branch.md`

### Debugging JVM Code

```bash
# Start Godot with debug port
godot --jvm-debug-port=5005
# Then attach a remote debugger in IntelliJ IDEA to localhost:5005
```

Debugging the registrar generator (bytecode processing):
```bash
cd kt/
./gradlew kspKotlin -Dkotlin.daemon.jvm.options="-Xdebug,-Xrunjdwp:transport=dt_socket\,address=8765\,server=y\,suspend=y"
# Halts compilation until remote debugger attaches at port 8765
# Note: incremental builds are disabled in debug mode — first compile will be slow
```

## Architecture

### C++ Layer (`src/`)

- **`src/gd_kotlin.h/cpp`** — `GDKotlin` singleton; owns the module state machine (`uninitialized → project_discovered → jvm_started → project_loaded → ...`). Many operations gate on correct state — check here first when debugging startup issues.
- **`register_types.cpp`** — Module entry point; registers `JvmScript` types, script languages, resource loaders/savers with Godot.
- **`src/lifecycle/`** — JVM startup (`jvm_manager`), class loader management, project settings parsing.
- **`src/jvm_wrapper/`** — JNI bridges, type conversion, per-thread shared buffer communication.
- **`src/script/`** — Script types: `JvmScript` (abstract base), `KotlinScript`, `JavaScriptLanguage`, `GdjScript`, `ScalaScript`.
- **`src/language/`** — `ScriptLanguage` implementations (`KotlinLanguage`, `JavaLanguage`, etc.) registered as Godot editor language options.
- **`src/binding/`** — Binding manager; maps Godot objects to JVM instances, synchronizes lifecycle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utopia-rise/godot-jvm](https://github.com/utopia-rise/godot-jvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
