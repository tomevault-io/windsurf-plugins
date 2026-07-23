---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

Tiny is a Kotlin Multiplatform game engine with Lua scripting support that compiles to desktop (JVM) and web (JavaScript) platforms. The project is structured as a multi-module Gradle build:

- **tiny-engine**: Core multiplatform game engine (commonMain, jvmMain, jsMain)
- **tiny-cli**: JVM-based CLI tool for development workflows
- **tiny-doc**: Documentation generation using Asciidoctor
- **tiny-doc-annotations**: Annotations for documentation generation
- **tiny-doc-generator**: KSP-based documentation processor
- **tiny-web-editor**: Web-based editor interface
- **tiny-samples**: Sample games and examples

## Key Technologies

- **Kotlin Multiplatform**: Shared code between JVM and JS platforms
- **Lua**: Game scripting via luak library
- **OpenGL**: Graphics rendering via kgl and LWJGL
- **Ktor**: HTTP server for CLI serve command
- **KSP**: Documentation generation from code annotations

## Development Commands

### Building
```bash
./gradlew build                    # Build all modules
./gradlew test                     # Run all tests
./gradlew publishToMavenLocal      # Deploy to local maven (also: make deploy)
./gradlew clean                    # Clean build artifacts
```

### Testing
```bash
./gradlew test                     # Run all tests
./gradlew :tiny-engine:test        # Run tests for specific module
./gradlew :tiny-engine:commonTest  # Run common multiplatform tests
./gradlew :tiny-engine:jvmTest     # Run JVM-specific tests
./gradlew :tiny-engine:jsTest      # Run JS-specific tests
```

### Linting
```bash
make lint          # or ./gradlew ktlintCheck
make lintfix       # or ./gradlew ktlintFormat
```

### CLI Development
```bash
make install       # Build and install CLI to ~/.bin/tiny-cli
./gradlew :tiny-cli:assembleDist   # Build CLI distribution
```

### Documentation Generation
```bash
make docs          # Generate full documentation (requires CLI install)
./gradlew asciidoctor              # Generate docs only
```

### CLI Commands (after installation)
```bash
tiny-cli create <name>    # Create new game project
tiny-cli run              # Run game in current directory
tiny-cli debug            # Run with debugger
tiny-cli serve            # Dev server with hot reload
tiny-cli export           # Export for web deployment
tiny-cli sfx              # Sound effect editor
tiny-cli add              # Add resources to project
tiny-cli palette          # Generate color palettes
```

## Architecture Details

### Platform Abstraction
The engine uses a Platform interface to abstract platform-specific functionality:
- `GlfwPlatform` for desktop (LWJGL/GLFW). It uses OpenGL 3.
- `WebGlPlatform` for web (WebGL). It uses WebGL 2.0.

### Resource Management
Games are structured around:
- `_tiny.json`: Game configuration
- `game.lua`: Main game script
- Assets: sprites, sounds, levels (LDtk support)

### Lua API Organization
The engine exposes functionality through organized Lua libraries:
- `gfx`: Graphics operations
- `spr`: Sprite management  
- `sfx`: Sound effects
- `shape`: Drawing primitives
- `ctrl`: Input handling
- `map`: Level/tilemap operations

### Open GL Organization
The engine use 2 stages of rendering: 
- tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/gl/SpriteBatchStage.kt : it renders everything in a framebuffer at a lower resolution.
- tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/gl/FrameBufferStage.kt : it renders the framebuffer from the previous stage at the screen resolution.
- The OpenGL abstraction is managers by 
  - tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/shader/ShaderProgram.kt (manage shader program)
  - tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/shader/ShaderParameter.kt (manager shader program parameters)
  - the shader program is created, alongside the shader program parameters. These parameters are created in Kotlin and added in the shader source code program. 
  - The shader program parameters can be configured using the method `setup` to access the vertex and fragment shader.
  - before draw, the `blind` is called. `unbind` is called after. 
  - fragColor is added automatically as out vec4 in FragmentShader (See tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/shader/BaseShader.kt)
  - #version is added automatically in the shader program source code (See tiny-engine/src/commonMain/kotlin/com/github/minigdx/tiny/render/shader/BaseShader.kt)

### Build Artifacts & Tasks
The build produces several specialized artifacts:
- `tinyWebEngine`: JS engine for web deployment
- `tinyApiAsciidoctor`: Generated API documentation  
- `tinyApiLuaStub`: Lua API stubs
- `tinyResources`: Packaged engine resources

Key Gradle tasks:
- `tiny-web-editor:tinyWebEditor`: Builds web editor interface
- `assembleDist`: Creates CLI distribution zip
- `asciidoctor`: Generates documentation using generated content

## Performance Considerations

### Critical Performance Areas
- **Input handling**: LWJGL input system can be slower than WebGL due to cursor position polling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minigdx/tiny](https://github.com/minigdx/tiny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
