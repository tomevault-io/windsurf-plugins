---
trigger: always_on
description: Uppercut is an IntelliJ IDEA plugin providing comprehensive IDE support for the **Karate testing framework**. It adds syntax highlighting, code completion, debugging, navigation, inspections, and run configurations for `.feature` (Karate/Gherkin) and `.featurejs` (KarateJs) files.
---

# CLAUDE.md - Uppercut Project Guide

## Project Overview

Uppercut is an IntelliJ IDEA plugin providing comprehensive IDE support for the **Karate testing framework**. It adds syntax highlighting, code completion, debugging, navigation, inspections, and run configurations for `.feature` (Karate/Gherkin) and `.featurejs` (KarateJs) files.

- **Plugin ID:** `com.rankweis`
- **Group:** `com.rankweis.uppercut`
- **License:** Apache 2.0
- **Repository:** https://github.com/rankweis/uppercut

## Build System

**Gradle 8.13** with Kotlin DSL (`build.gradle.kts`). Uses the Gradle wrapper (`./gradlew`).

### Key Commands

```bash
# Build the plugin
./gradlew buildPlugin

# Run all checks (unit tests + checkstyle + kover)
./gradlew check

# Run unit tests only
./gradlew test

# Run platform tests (JUnit Vintage engine)
./gradlew platformTest

# Run integration tests (requires prepareSandbox)
./gradlew integrationTest

# Launch IntelliJ with the plugin loaded for manual testing
./gradlew runIde

# Verify plugin compatibility
./gradlew verifyPlugin

# Generate JavaScript lexer from JFlex grammar
./gradlew generateLexer
```

Tests in CI run under `xvfb` for headless display:
```bash
xvfb-run --auto-servernum --server-args="-screen 0 1920x1080x24" ./gradlew check -x integrationTest
```

### JDK Requirements

- **Main project:** Java 21 (jvmToolchain)
- **KarateTestRunner subproject:** Java 17 (jvmToolchain)

### Gradle Properties

Key version properties are in `gradle.properties`:
- `pluginVersion` - Current plugin version (SemVer)
- `platformVersion` - Target IntelliJ platform version
- `pluginSinceBuild` / `pluginUntilBuild` - Compatibility range
- `karateVersion` - Karate framework version (1.5.1)

Dependency versions are managed in `gradle/libs.versions.toml`.

**When updating `platformVersion`:** Always run `./gradlew verifyPlugin test` to ensure plugin compatibility and tests pass against the new platform version.

## Project Structure

```
uppercut/
├── build.gradle.kts              # Root build configuration
├── settings.gradle.kts           # Multi-project settings
├── gradle.properties             # Version properties
├── gradle/libs.versions.toml     # Dependency version catalog
├── config/checkstyle/            # Checkstyle configuration (Google Java Style)
├── KarateTestRunner/             # Subproject: custom Karate test runner
│   └── build.gradle.kts
├── src/
│   ├── main/
│   │   ├── kotlin/               # Main source (mostly Java despite the path)
│   │   │   ├── com/rankweis/uppercut/
│   │   │   │   ├── karate/       # Core plugin logic
│   │   │   │   ├── parser/       # AST parsing utilities
│   │   │   │   ├── settings/     # Plugin settings UI & persistence
│   │   │   │   └── util/         # General utilities
│   │   │   └── io/karatelabs/js/ # JavaScript language support + JFlex grammar
│   │   ├── java/io/karatelabs/js/# Generated lexer output (js.jflex source also here)
│   │   └── resources/
│   │       ├── META-INF/
│   │       │   ├── plugin.xml        # Main plugin descriptor
│   │       │   └── plugin-withJs.xml # JavaScript support (optional dependency)
│   │       ├── colorSchemes/     # Darcula and Default color schemes
│   │       ├── icons/            # Plugin icons (SVG)
│   │       ├── messages/         # Localization bundles
│   │       └── i18n.json         # Internationalization keywords
│   ├── test/                     # Unit tests
│   │   ├── kotlin/               # Test classes
│   │   └── testData/             # Test fixture files (.feature)
│   ├── integrationTest/          # IDE integration tests (IDE Starter + Driver)
│   └── platformTest/             # Platform compatibility tests
└── .github/workflows/
    ├── build.yml                 # CI: build, test, verify, draft release
    ├── release.yml               # Publish to JetBrains Marketplace
    ├── contrib.yml               # Contributor attribution
    └── run-ui-tests.yml          # Cross-platform UI tests
```

### Main Source Packages (`com.rankweis.uppercut.karate`)

| Package | Purpose |
|---------|---------|
| `psi/` | PSI elements, parser definitions, file types |
| `psi/impl/` | PSI element implementations |
| `psi/parser/` | Parser builders (KarateJs parser) |
| `psi/annotator/` | Syntax error annotations |
| `psi/formatter/` | Code style settings providers |
| `psi/structure/` | File structure view |
| `psi/refactoring/` | Rename/refactoring support |
| `psi/i18n/` | Internationalization support |
| `lexer/` | Lexer interface and implementations |
| `lexer/impl/` | Karate language lexer |
| `lexer/karatelabs/` | JavaScript lexer |
| `actions/` | IDE actions (refactoring, selection) |
| `codeinsight/` | Enter handler, typed handler |
| `completion/` | Code completion contributors |
| `debugging/` | Debug session and breakpoint management |
| `extension/` | JSON, JavaScript, inspection extensions |
| `format/` | Code formatting model builders |
| `highlight/` | Syntax highlighting (Gherkin, JavaScript) |
| `inspections/` | Code inspections (undefined steps, broken tables, etc.) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RankWeis/uppercut](https://github.com/RankWeis/uppercut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
