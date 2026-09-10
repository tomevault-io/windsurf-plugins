---
trigger: always_on
description: This file is the authoritative guide for working on
---

# Godot IntelliJ Plugin Guide

This file is the authoritative guide for working on
`kt/plugins/godot-intellij-plugin`.

The plugin provides Godot JVM code insight for Kotlin, Java, and Scala. Keep
the implementation small: inspections dispatch PSI elements, analyzers own
the rules, and tests exercise the same source fixtures through IntelliJ's
real highlighting pipeline.

## Inspection Architecture

Inspection entry points live in:

- `src/main/kotlin/godot/intellij/plugin/inspection/JavaInspection.kt`
- `src/main/kotlin/godot/intellij/plugin/inspection/KotlinInspection.kt`
- `src/main/kotlin/godot/intellij/plugin/inspection/ScalaInspection.kt`
- `src/main/kotlin/godot/intellij/plugin/inspection/CoreTypeCopyModificationInspection.kt`

Shared JVM rules live under `analysis/jvm/`. Kotlin PSI and K2-specific rules
live under `analysis/kotlin/`. Registration-mode decisions belong in
`registration/RegistrationPolicy.kt`.

Keep inspection classes as dispatchers. Do not copy rule logic into the
language entry points when a shared analyzer can express it cleanly.

## New Script Action

`action/NewGodotScriptAction` adds `New | Godot Script` to the project view.
It is only visible for directories inside a Godot project.

`NewGodotScriptDialog` collects the class name, language, Godot base class,
and lifecycle overrides. The base class chooser is an inheritance chooser
rooted at `KtObject`, and only the lifecycle functions declared by the chosen
base class can be selected.

`GodotScriptGenerator` builds the source. Annotations follow
`RegistrationMode`: none in Automatic, `@Script` in Inferred, and `@Script`
plus `@Register` on every override in Explicit. Its output is covered by
`GodotScriptGeneratorTest` for Kotlin, Java, and Scala.

## Run Configuration

`run/GodotRunConfigurationType` registers the `Godot` configuration type. Its
factory is applicable only when the project contains a `project.godot`, which
`Project.godotRoot` resolves through `GodotProjectScopeService`.

`GodotRunConfiguration.buildCommandLine` is the part worth protecting. It
passes the chosen JDK with `--jvm-path`, which the binding resolves before the
embedded JRE and the environment. It also exports `JAVA_HOME` for the tools
Godot starts itself, such as Gradle, but never touches `PATH` and never falls
back to either: Godot-JVM builds without that argument are not supported.
`GodotRunConfigurationTest` covers the editor, game, and JDK cases.

The JDK picker is the platform `SdkComboBox`, so the project SDK, registered
JDKs, and JDKs detected on the machine are all offered without extra code.

## IDE Compatibility

`pluginVerification.ides` in `build.gradle.kts` lists the oldest supported IDE
plus the versions the Marketplace verifies against. Keep it in sync with the
Marketplace, and run it before publishing:

```powershell
.\gradlew.bat :godot-intellij-plugin:verifyPlugin
```

Verifying only the IDEs the plugin is built against hides breakage: Kotlin
plugin internals such as `KtUltraLightClass` disappear in newer builds. Prefer
public, long lived APIs (`KtLightClass`), and own small helpers rather than
calling deprecated Kotlin PSI methods.

## Marketplace Metadata

Both descriptions the Marketplace shows are generated at build time by
`patchPluginXml`; never edit `plugin.xml` for them.

- description: the `<!-- Plugin description -->` block in `README.md`.
  Keep it current when a user-visible feature is added.
- what's new: `CHANGELOG.md`. `publishPlugin` depends on `patchChangelog`,
  which turns `[Unreleased]` into a versioned section, so `changeNotes`
  reads the released version first and falls back to `[Unreleased]`.

Add every user-visible change to `[Unreleased]` under a Keep a Changelog
group. It is published, not just internal documentation.

## Registration Highlighting

Registration highlighting is separate from inspections. It gives declaration
lines a subtle background:

- orange: the declaration is not structurally registerable
- blue: the declaration is a registration candidate
- green: the current mode selects the declaration for registration

Only Godot script classes and their members are highlighted. Unrelated classes
must remain untouched. The shared eligibility rules live under `highlighting/`;
the Scala annotator only adapts Scala property PSI to those rules.

`RegistrationHighlightingTest` covers the same Kotlin/Java/Scala by
Explicit/Inferred/Automatic 3x3 matrix as regular registration inspections.

## Registration Modes

Every regular registration inspection must be tested in all three modes.

### Explicit

Only direct registration annotations count:

- class: `@Script`
- property: `@Visible`
- signal: `@Emit`
- function: `@Register` or `@Notification`

Meta-annotations are not expanded. Godot lifecycle overrides such as
`_ready` must be registered explicitly.

### Inferred

Registration meta-annotations are expanded. Examples:

- `@Tool` carries `@Script`
- `@Export` and property hints carry property registration metadata
- `@Rpc` carries function registration metadata

Godot lifecycle overrides are inferred. Logical signals in registered
classes are inferred, while an effective `@Emit` still identifies a signal
inside a class that is missing registration.

### Automatic


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utopia-rise/godot-jvm](https://github.com/utopia-rise/godot-jvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
