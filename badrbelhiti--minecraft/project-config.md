---
trigger: always_on
description: A Kotlin-based Minecraft server network with AWS CDK infrastructure management.
---

# Nerds @ ATX Minecraft Network

A Kotlin-based Minecraft server network with AWS CDK infrastructure management.

> **Note to Claude Code**: This file should be continuously updated with best practices, patterns, and architectural decisions established during development. When new patterns emerge or practices are refined, update this document to reflect the current state of the project.

## Project Structure

This is a Gradle monorepo with a flat directory structure:

```
Minecraft/
├── settings.gradle.kts           # Defines all modules
├── build.gradle.kts              # Root build config with common settings
├── gradle.properties             # Gradle configuration
├── gradle/
│   └── libs.versions.toml        # Centralized dependency versions
├── MinecraftCdk/                 # AWS CDK infrastructure as code
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/cdk/
├── BungeeCordProxy/              # BungeeCord proxy plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/proxy/
├── SurvivalServer/               # Survival server configuration
│   ├── build.gradle.kts
│   └── server/
├── SkyBlockServer/               # SkyBlock server configuration
│   ├── build.gradle.kts
│   └── server/
├── SurvivalPlugin/               # Survival game plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/survival/
├── SkyBlockPlugin/               # SkyBlock game plugin
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/skyblock/
├── SharedCore/                   # Shared utilities and models
│   ├── build.gradle.kts
│   └── src/main/kotlin/dev/nerdsatx/shared/
└── Documentation/                # Documentation and diagrams
    ├── build.gradle.kts
    ├── diagrams/                 # Generated PNG diagrams
    └── src/main/plantuml/        # PlantUML source files
```

## Package Structure

All code uses the base package: `dev.nerdsatx`

- `dev.nerdsatx.shared` - Shared utilities, models, database code
- `dev.nerdsatx.survival` - Survival plugin code
- `dev.nerdsatx.skyblock` - SkyBlock plugin code
- `dev.nerdsatx.proxy` - BungeeCord proxy plugin
- `dev.nerdsatx.cdk` - AWS infrastructure code

## Module Dependencies

```
SurvivalPlugin  ──→  SharedCore
SkyBlockPlugin  ──→  SharedCore
BungeeCordProxy ──→  SharedCore
MinecraftCdk    (independent)
SurvivalServer  (independent)
SkyBlockServer  (independent)
Documentation   (independent)
```

## Technology Stack

- **Language**: Kotlin 1.9.22
- **Build Tool**: Gradle with Kotlin DSL
- **Minecraft Platform**: Paper API (1.20.4) for game servers
- **Proxy**: BungeeCord API
- **Infrastructure**: AWS CDK
- **Plugin Packaging**: Shadow plugin for fat JARs

## Key Gradle Patterns

### Version Catalog (`gradle/libs.versions.toml`)

We use Gradle's version catalog feature for centralized dependency management:

```toml
[versions]
kotlin = "1.9.22"
paper = "1.20.4-R0.1-SNAPSHOT"

[libraries]
paper-api = { module = "io.papermc.paper:paper-api", version.ref = "paper" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

Access in build files: `libs.paper.api` or `alias(libs.plugins.kotlin.jvm)`

### Common Build Patterns

**Plugin Module Pattern** (SurvivalPlugin, SkyBlockPlugin):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    alias(libs.plugins.shadow)
}

dependencies {
    implementation(project(":SharedCore"))
    compileOnly(libs.paper.api)
    implementation(libs.kotlin.stdlib)
}

tasks {
    shadowJar {
        archiveFileName.set("PluginName.jar")
        relocate("kotlin", "dev.nerdsatx.pluginname.shaded.kotlin")
    }
    build {
        dependsOn(shadowJar)
    }
}
```

**Proxy Module Pattern** (BungeeCordProxy):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    alias(libs.plugins.shadow)
}

dependencies {
    implementation(project(":SharedCore"))
    compileOnly(libs.bungeecord.api)
    implementation(libs.kotlin.stdlib)
}
```

**CDK Module Pattern** (MinecraftCdk):
```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
    application
}

application {
    mainClass.set("dev.nerdsatx.cdk.MinecraftCdkAppKt")
}

dependencies {
    implementation(libs.aws.cdk.lib)
    implementation(libs.aws.constructs)
}
```

**Documentation Module Pattern** (Documentation):
```kotlin
plugins {
    base  // Required for IntelliJ to recognize as a Gradle module
}

// Configuration for PlantUML dependency
val plantuml by configurations.creating

dependencies {
    plantuml("net.sourceforge.plantuml:plantuml:1.2024.3")
}

val plantumlSourceDir = file("src/main/plantuml")
val plantumlOutputDir = file("diagrams")

tasks {
    // Generate PlantUML diagrams
    val generateDiagrams by registering(JavaExec::class) {
        description = "Generate PNG diagrams from PlantUML files"
        group = "documentation"

        mainClass.set("net.sourceforge.plantuml.Run")
        classpath = plantuml
        jvmArgs("-DPLANTUML_LIMIT_SIZE=8192")
        args(
            "-v",
            "-tpng",
            "-SdefaultRenderer=smetana",  // Use built-in Smetana renderer
            "-o", plantumlOutputDir.absolutePath,
            plantumlSourceDir.absolutePath
        )

        inputs.dir(plantumlSourceDir)
        outputs.dir(plantumlOutputDir)

        doFirst {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BadrBelhiti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
