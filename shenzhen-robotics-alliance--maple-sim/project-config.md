---
trigger: always_on
description: **Maple-Sim** is an advanced FRC (FIRST Robotics Competition) Java robot simulation library that integrates the open-source Java rigid-body dynamics engine **dyn4j** to simulate realistic 2D forces and collisions between rigid shapes.
---

# Maple-Sim Developer Knowledge Base

## Project Overview

**Maple-Sim** is an advanced FRC (FIRST Robotics Competition) Java robot simulation library that integrates the open-source Java rigid-body dynamics engine **dyn4j** to simulate realistic 2D forces and collisions between rigid shapes.

### Project Information
- **Repository**: https://github.com/Shenzhen-Robotics-Alliance/maple-sim
- **Documentation**: https://shenzhen-robotics-alliance.github.io/maple-sim/
- **JavaDocs**: https://shenzhen-robotics-alliance.github.io/maple-sim/javadocs/
- **Physics Engine**: dyn4j 5.0.2 (2D rigid-body dynamics)
- **Java Version**: 17 (source & target compatibility)
- **WPILib Version**: 2025.3.2
- **Gradle Version**: 8.11

---

## Build System

### Project Structure
```
maple-sim/
├── project/                    # Main library source
│   ├── src/main/java/          # Java source code
│   ├── src/main/native/        # C++ source (if applicable)
│   ├── build.gradle            # Main build configuration
│   ├── publish.gradle          # Publishing configuration
│   ├── config.gradle           # C++ configuration
│   └── settings.gradle         # Gradle settings
├── docs/                       # Documentation (Markdown)
│   ├── vendordep/              # VendorDep JSON and Maven repo
│   └── javadocs/               # Generated JavaDocs
└── templates/                  # Example projects

```

### Gradle Plugins Used
- `java` - Java compilation
- `cpp` - C++ support (minimal usage)
- `google-test` - Testing framework
- `com.diffplug.spotless` 7.0.0.BETA4 - Code formatting
- `edu.wpi.first.wpilib.repositories.WPILibRepositoriesPlugin` 2025.0 - WPILib repos
- `edu.wpi.first.NativeUtils` 2025.9.0 - Native utilities
- `edu.wpi.first.GradleJni` 1.1.0 - JNI support
- `edu.wpi.first.GradleVsCode` 2.1.0 - VSCode integration
- `edu.wpi.first.GradleRIO` 2025.3.2 - FRC robot support
- `maven-publish` - Publishing to Maven

### Dependencies

**Core Physics**:
- `org.dyn4j:dyn4j:5.0.2` - Physics engine

**WPILib Dependencies** (version 2025.3.2):
- `cscore-java` - Camera support
- `cameraserver-java` - Camera server
- `ntcore-java` - NetworkTables
- `wpilibj-java` - Core WPILib
- `wpiutil-java` - Utilities
- `wpimath-java` - Math library
- `wpiunits-java` - Units library
- `wpilibNewCommands-java` - Command-based framework
- `hal-java` - Hardware abstraction layer

**Additional Libraries**:
- `org.ejml:ejml-simple:0.43.1` - Matrix operations
- `com.fasterxml.jackson.*:2.15.2` - JSON processing
- `edu.wpi.first.thirdparty.frc2024.opencv:opencv-java:4.8.0-2` - OpenCV

### Build Tasks

**Compilation**:
```bash
./gradlew build              # Compiles and runs tests
./gradlew compileJava        # Compiles Java only (triggers spotlessApply)
```

**Code Formatting**:
```bash
./gradlew spotlessApply      # Formats all code (automatic on compile)
./gradlew spotlessCheck      # Checks formatting without applying
```

**Documentation**:
```bash
./gradlew javadoc            # Generates JavaDocs to ../javadocs/
```

**Publishing**:
```bash
./gradlew publish            # Publishes to local Maven repo (docs/vendordep/repos/releases)
./gradlew publishToMavenLocal # Publishes to ~/.m2 for local testing
```

**Artifact Generation**:
```bash
./gradlew outputJar          # Creates main JAR
./gradlew outputSourcesJar   # Creates sources JAR
./gradlew outputJavadocJar   # Creates JavaDoc JAR
./gradlew copyAllOutputs     # Copies all outputs to build/allOutputs
```

### Publishing Configuration

**Maven Coordinates**:
- **Group ID**: `org.ironmaple`
- **Artifact ID**: `maplesim-java`
- **Version**: Defined in `project/publish.gradle` (currently `0.3.14-test`)

**Publishing Location**:
- Local test repo: `docs/vendordep/repos/releases`
- VendorDep JSON: `docs/vendordep/maple-sim.json` (auto-generated with version)

**Artifacts Published**:
1. Main JAR (`maplesim-java-{version}.jar`)
2. Sources JAR (`maplesim-java-{version}-sources.jar`)
3. JavaDoc JAR (`maplesim-java-{version}-javadoc.jar`)

### Testing Changes

To test your changes:

1. **Update version** in `project/publish.gradle`:
   ```groovy
   def pubVersion = 'my-feature-test'
   ```

2. **Publish to Maven Local**:
   ```bash
   ./gradlew publishToMavenLocal
   ```
   Or use IntelliJ: Click "Publish to Maven Local" in Gradle tool window

3. **Copy vendordep** from `docs/vendordep/maple-sim.json` to a template project's `vendordeps/` folder

4. **Test** in one of the template projects

---

## Code Style Guidelines

### ⚠️ CRITICAL: Automatic Formatting

**Spotless** is configured to automatically format code on every compilation. Your code will be formatted when you run `./gradlew build` or `./gradlew compileJava`.

- **Java**: Palantir Java Format 2.39.0 (with JavaDoc formatting)
- **Groovy/Gradle**: Greclipse formatter
- **Markdown/Misc**: 4-space indentation

### Formatting Rules Applied

**Java**:
- Palantir Java Format style
- JavaDoc formatting enabled
- Unused imports removed
- Trailing whitespace trimmed
- Files end with newline

**Gradle Files**:
- 4-space indentation
- Trailing whitespace trimmed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shenzhen-Robotics-Alliance/maple-sim](https://github.com/Shenzhen-Robotics-Alliance/maple-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
