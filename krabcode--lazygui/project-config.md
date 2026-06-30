---
trigger: always_on
description: LazyGui is a GUI library for Processing (Java-based creative coding framework). It provides a feature-rich, visually minimalist GUI with lazy initialization of control elements - you don't register controls in `setup()`, you just ask for their values in `draw()` at unique string paths.
---

# LazyGui - Copilot Coding Agent Instructions

## Repository Overview

LazyGui is a GUI library for Processing (Java-based creative coding framework). It provides a feature-rich, visually minimalist GUI with lazy initialization of control elements - you don't register controls in `setup()`, you just ask for their values in `draw()` at unique string paths.

**Repository Stats:**
- Size: ~2.1MB
- Language: Java (79 source files)
- Build System: Gradle 8.0
- Java Compatibility: Java 8+ (sourceCompatibility 1.8)
- Main Dependencies: Processing 3.3.7, Gson 2.8.9
- No automated tests exist in this repository

## Build and Validation Steps

### Prerequisites
- Java 8 or later (tested with Java 17)
- Gradle 8.0 (included via wrapper)
- 7z (for deployment only)

### Build Commands (Ordered by Frequency of Use)

**IMPORTANT:** Always make gradlew executable before first use:
```bash
chmod +x gradlew
```

**1. Standard Build (fast, ~1-2 seconds)**
```bash
./gradlew build
```
- Compiles Java sources
- Creates jar in `build/libs/LazyGui.jar` (~186KB)
- No tests run (repository has no test suite)
- Safe to run anytime

**2. Clean Build (when in doubt)**
```bash
./gradlew clean build
```
- Removes `build/` directory
- Full recompile from scratch
- Use when build artifacts seem stale

**3. Shadow JAR (for releases)**
```bash
./gradlew shadowJar
```
- Creates fat jar with bundled Gson dependency: `build/libs/LazyGui-with-gson.jar` (~670KB)
- Excludes Processing (users provide their own Processing installation)
- Required before running deploy.sh

**4. Deployment (maintainer only)**
```bash
chmod +x deploy.sh
./gradlew shadowJar && ./deploy.sh
```
- Packages library for Processing IDE distribution
- Creates .jar, .txt, .zip in `build/deploy/`
- Requires shadowJar to be run first
- Deletes `data/gui/` (test saves) before packaging

### Common Build Issues and Solutions

**Issue: Permission denied on gradlew**
- Solution: Run `chmod +x gradlew` before executing gradle commands

**Issue: Permission denied on deploy.sh**
- Solution: Run `chmod +x deploy.sh` before executing

**Issue: Build artifacts from old code**
- Solution: Run `./gradlew clean` before building

**No automated CI/CD:** This repository has no GitHub Actions workflows or automated testing. Manual validation is required.

## Project Architecture and Layout

### Source Structure
```
src/main/java/com/krab/lazy/
├── LazyGui.java                    # Main API class (1435 lines)
├── LazyGuiSettings.java            # Configuration builder
├── Input.java                      # Keyboard input utility
├── ShaderReloader.java             # Live shader reload utility
├── PickerColor.java                # Color picker return type
├── ColorPoint.java                 # Gradient color point
├── KeyState.java                   # Input state tracking
├── examples/                       # Processing IDE examples (.pde files)
│   ├── BasicExample/
│   ├── GeneralOverview/
│   ├── GradientColorAt/
│   ├── IndexedPaths/
│   ├── InputWatcher/
│   ├── MouseDrawing/
│   ├── OptionalSetup/
│   ├── PathHiding/
│   ├── PeasyCamExample/
│   ├── ShaderReloading/
│   ├── TexturedTriangleFan/
│   └── UtilityMethods/
├── examples_intellij/              # IntelliJ/IDE examples (.java files)
│   ├── Controls.java
│   ├── ExampleSketch.java
│   ├── Gradient.java
│   ├── GradientSimple.java
│   ├── ReadmeVisualsGenerator.java
│   ├── SimpleShape.java
│   └── ... (14 total)
├── nodes/                          # GUI control element implementations
│   ├── AbstractNode.java
│   ├── ButtonNode.java
│   ├── ColorPickerFolderNode.java
│   ├── FolderNode.java
│   ├── SliderNode.java
│   └── ... (~30 node types)
├── stores/                         # Data persistence
├── themes/                         # GUI theming
├── utils/                          # Utility classes
├── windows/                        # Window management
└── input/                          # Input handling internals
```

### Key Configuration Files
- `build.gradle` - Build configuration, dependencies, shadowJar setup
- `settings.gradle` - Basic Gradle project settings
- `library.properties` - Processing library metadata (version, description, etc.)
- `.gitignore` - Excludes: `.idea/`, `build/`, `data/gui/`, `.gradle/`

### Resource Files
- `data/shaders/` - GLSL shaders for GUI rendering and gradients
  - `gradient.glsl` - Main gradient rendering shader
  - `sliderBackground.glsl`, `sliderBackgroundColor.glsl` - Slider visuals
  - `checkerboard.glsl`, `guideGridPoints.glsl`, `testShader.glsl`
- `data/JetBrainsMono-Regular.ttf` - Default GUI font
- `docs/` - Generated Javadoc (regenerated from IntelliJ, published to GitHub Pages)

### Public API (only ~6 classes)
The public-facing API that users interact with:
1. `LazyGui.java` - Main GUI class with all control methods
2. `LazyGuiSettings.java` - Constructor settings builder
3. `Input.java` - Keyboard input utility
4. `ShaderReloader.java` - Shader hot-reload utility
5. `PickerColor.java` - Color picker return value

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KrabCode/LazyGui](https://github.com/KrabCode/LazyGui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
