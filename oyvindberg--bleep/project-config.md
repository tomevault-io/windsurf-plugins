---
trigger: always_on
description: This document provides comprehensive background for working with bleep, a modern JVM build tool for Java, Kotlin, and Scala that emphasizes speed, simplicity, and excellent developer experience.
---

# Bleep Build Tool - Developer Guide

This document provides comprehensive background for working with bleep, a modern JVM build tool for Java, Kotlin, and Scala that emphasizes speed, simplicity, and excellent developer experience.

## Overview

Bleep is a native-image compiled JVM build tool that replaces Maven, Gradle, and sbt with:
- **Instant startup** (~0ms) through GraalVM native image compilation
- **Declarative YAML configuration** instead of XML, Groovy/Kotlin DSL, or Scala DSL
- **Excellent IDE integration** through BSP (1-second imports)
- **Simple script system** — write build logic in Java or Scala
- **Fast compilation** via its own BSP server with Zinc, javac, and kotlinc

## Core Concepts

### 1. Build Configuration Structure

Bleep uses YAML for build configuration (`bleep.yaml`):

```yaml
$schema: https://raw.githubusercontent.com/oyvindberg/bleep/master/schema.json
$version: 1.0.0-M7
jvm:
  name: graalvm-community:24.0.1
projects:
  myapp:
    dependencies: 
      - com.lihaoyi::fansi:0.3.1  # Scala dependency (double colon)
      - org.scalameta:svm-subs:101.0.0  # Java dependency (single colon)
    dependsOn: myapp-core
    extends: template-common
    platform:
      mainClass: com.example.Main
templates:
  template-common:
    scala:
      version: 3.3.3
scripts:
  generate-docs:
    main: bleep.scripts.GenDocumentation
    project: scripts
```

### 2. Templates System

Templates reduce duplication through inheritance:
- Projects extend templates with `extends: template-name`
- Templates can extend other templates
- Common patterns: `template-common`, `template-cross-all`, `template-scala-3`

### 3. Cross-Building

Cross-building for different Scala versions/platforms:

```yaml
projects:
  mylib:
    cross:
      jvm213:
        scala:
          version: 2.13.11
      jvm3:
        scala:
          version: 3.3.3
      js3:
        platform:
          name: js
          jsVersion: 1.13.2
```

### 4. Java Annotation Processors

Three composable fields under `java:`. Default off — nothing scans, nothing runs unless you opt in.

```yaml
projects:
  myapp:
    dependencies:
      - org.projectlombok:lombok:1.18.46
    java:
      scanForAnnotationProcessors: true        # opt-in: scan deps for META-INF/services
      annotationProcessors:                     # explicit processor-only deps (not on runtime classpath)
        - org.mapstruct:mapstruct-processor:1.5.5.Final
      annotationProcessorOptions:               # -A<k>=<v> flags
        mapstruct.suppressGeneratorTimestamp: "true"
```

Resolution runs as a per-project DAG task at compile time, not bootstrap (`bleep-bsp/.../TaskDag.scala` `ResolveAnnotationProcessorsTask` and `MultiWorkspaceBspServer.makeAnnotationProcessorHandler`). Failures surface through `BuildSummary.apResolutionFailed` so `commands.compile` throws on misconfig. See `bleep-site-in/usage/annotation-processing.mdx` for the user-facing doc.

### 5. Scripts (Replacing sbt Tasks/Plugins)

Scripts are regular Scala programs that replace sbt's task system:

```scala
object MyScript extends BleepScript("MyScript") {
  override def run(started: Started, commands: Commands, args: List[String]): Unit = {
    // Access build information
    val build = started.build.exploded
    // Run commands
    commands.compile(started.build.explodedProjects.keys.toList)
  }
}
```

Key advantages:
- Can be debugged like normal programs
- No classloader issues
- Full access to build model
- Can use any dependencies

## Working with Liberated Projects

The `liberated/` directory contains sbt projects ported to bleep. Each is a git submodule with:
- `origin` remote pointing to bleep-build fork
- `origin-original` remote pointing to upstream project

### Porting Patterns from sbt to bleep

Based on analysis of sbt-ci-release and sbt-sonatype ports:

1. **Package Restructuring**
   - Move to `bleep.plugin.*` namespace
   - Example: `com.geirsson.CiReleasePlugin` → `bleep.plugin.cirelease.CiReleasePlugin`

2. **Dependency Changes**
   - Replace sbt-specific dependencies with standard libraries
   - Use bleep's logging (ryddig) instead of sbt's logger
   - Example: sbt's `Keys` → direct parameter passing

3. **Plugin Architecture**
   - sbt plugins become regular classes with constructor parameters
   - No more `AutoPlugin` trait or `override def requires`
   - Settings become constructor parameters or method arguments

4. **Build Definition**
   - `build.sbt` → included directly in parent's `bleep.yaml`
   - Source directories referenced via `sources:` field
   - Example:
     ```yaml
     bleep-plugin-ci-release:
       sources: ../liberated/sbt-ci-release/plugin/src/main/scala
       dependsOn:
         - bleep-plugin-dynver
         - bleep-plugin-pgp
         - bleep-plugin-sonatype
     ```

5. **API Adaptations**
   - Replace sbt tasks with method calls
   - Use bleep's model types (e.g., `model.Repository`)
   - Logging: `sbt.Logger` → `ryddig.Logger`

6. **Cross-Compilation**
   - Use `for3Use213: true` for libraries not yet on Scala 3
   - Handle binary compatibility carefully

## Bleep's Own Build Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oyvindberg/bleep](https://github.com/oyvindberg/bleep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
