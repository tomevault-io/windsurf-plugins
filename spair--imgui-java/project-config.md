---
trigger: always_on
description: Guidance for AI coding agents working on `imgui-java`.
---

# AGENTS.md

Guidance for AI coding agents working on `imgui-java`.

This file is the operational guide: what the repo is, how to build it, the procedure for
upgrading submodules, and the codegen-internals conventions. The everyday conventions and
"don't"s live in companion files — read them too:

- **`.claude/rules/guardrails.md`** — what *not* to do. Covers the golden rules (never
  edit `src/generated/java/`, never commit `bin/` natives), the JNI contract, Java 8
  compatibility on consumed classes, doclint pitfalls, dependency hygiene, AST drift,
  and the rule against mixing submodule bumps with feature work.
- **`.claude/rules/codestyle.md`** — Java style for `imgui-binding/`, `imgui-lwjgl3/`,
  `imgui-app/` (formatting, naming, modifiers, imports, javadoc, wrapper class shape,
  binding-source annotations). Authoritative copy is `config/checkstyle/checkstyle.xml`.
- **`.claude/docs/patterns.md`** — recurring patterns when adding to the binding (the
  codegen loop, annotated stubs, struct wrappers, out-parameter `Im*` wrappers, module
  boundaries, application lifecycle).
- **`docs/CONTRIBUTING.md`** — PR/commit workflow. Includes the conventional-commit
  format and, importantly, the **`Co-authored-by` trailer rule for AI-assisted commits**
  and the "you are responsible for the change" rule. Read it before opening a PR.

If anything below conflicts with those files, the dedicated file wins — update it
there, not here.

## What this repo is

JNI-based Java binding for [Dear ImGui](https://github.com/ocornut/imgui) + extensions (ImPlot, ImNodes, ImGuizmo, imgui-node-editor, imgui-knobs, imgui-file-dialog, imgui-text-edit, imgui-club). Multi-module Gradle build, published as `io.github.spair:imgui-java-{binding,lwjgl3,app}`. Java is **codegen-driven**: annotated stubs in `imgui-binding/src/main/java/`, expanded by the Spoon-based generator in `buildSrc/` into `imgui-binding/src/generated/java/` — both trees committed.

The most important rule to internalize before doing anything else: **never edit
`imgui-binding/src/generated/java/` by hand** — it is regenerated from the annotated
sources in `imgui-binding/src/main/java/`. See `.claude/rules/guardrails.md` for the
full statement and the workflow.

## Project layout

| Path                                                   | What's there                                                                                            |
|--------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| `imgui-binding/`                                       | Core JNI binding + codegen                                                                              |
| `imgui-binding/src/main/java/`                         | Hand-written annotated sources (`@BindingSource`, `@BindingField`, `@BindingMethod`, `@BindingAstEnum`) |
| `imgui-binding/src/generated/java/`                    | Codegen output; committed. See `.claude/rules/guardrails.md`                                            |
| `imgui-binding/src/main/native/`                       | Hand-written JNI `.cpp` / `.h` (struct marshaling etc.)                                                 |
| `imgui-lwjgl3/`                                        | LWJGL backend (GLFW + OpenGL)                                                                           |
| `imgui-app/`                                           | Convenience wrapper bundling natives + an `Application` entry-point                                     |
| `include/`                                             | Git submodules: imgui, implot, imnodes, imguizmo, etc.                                                  |
| `bin/`                                                 | Native libs (`.so/.dylib/.dll`), committed by CI                                                        |
| `buildSrc/`                                            | Gradle plugin: codegen tasks, JNI build, AST parser                                                     |
| `buildSrc/src/main/resources/generator/api/ast/*.json` | Clang-dumped AST data, consumed by `@BindingAstEnum`                                                    |
| `example/`                                             | Example apps                                                                                            |
| `patches/`                                             | Local patches against vendored submodules                                                               |

## Build & test

Gradle's toolchain pins JDK 17 regardless of the system JDK.

```bash
./gradlew :imgui-binding:compileJava        # fast Java-side check
./gradlew :imgui-binding:javadoc            # catch doc errors (CI also runs this; mandatory pass)
./gradlew :imgui-binding:generateApi        # regen src/generated/java from annotated sources
./gradlew generateAst                       # regen clang AST JSONs (needs clang++ installed)
./gradlew :imgui-binding:generateLibs       # build native JNI libs (per-OS; CI handles cross-platform)
./gradlew build                             # full build + tests
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpaiR/imgui-java](https://github.com/SpaiR/imgui-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
