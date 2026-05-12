---
trigger: always_on
description: **@rosalana/sandbox** is a lightweight WebGL wrapper for simple, beautiful shader effects. Focuses on clean API, type safety, and fast setup. Ideal for gradients, ambient backgrounds, and animated GLSL experiments.
---

# CLAUDE.md - Rosalana Sandbox

## Project Overview

**@rosalana/sandbox** is a lightweight WebGL wrapper for simple, beautiful shader effects. Focuses on clean API, type safety, and fast setup. Ideal for gradients, ambient backgrounds, and animated GLSL experiments.

- **Version**: 0.0.5
- **WebGL support**: WebGL1 and WebGL2 with automatic fallback
- **License**: MIT
- **npm**: `@rosalana/sandbox`

## Architecture

### Entry Point
- [src/index.ts](src/index.ts) - Exports `Sandbox` class and all types/errors

### Core Classes

| Class | File | Purpose |
|-------|------|---------|
| `Sandbox` | [src/index.ts](src/index.ts) | Main public API, orchestrates entire system |
| `WebGL` | [src/tools/web_gl.ts](src/tools/web_gl.ts) | Internal WebGL orchestrator - context, render loop |
| `Program` | [src/tools/program.ts](src/tools/program.ts) | Shader compilation, program linking |
| `Geometry` | [src/tools/geometry.ts](src/tools/geometry.ts) | Vertex buffers, VAO (WebGL1/2 compatibility) |
| `Uniforms` | [src/tools/uniforms.ts](src/tools/uniforms.ts) | Uniform collection, built-in uniforms management |
| `Uniform` | [src/tools/uniform.ts](src/tools/uniform.ts) | Single uniform - method inference, GPU upload |
| `Clock` | [src/tools/clock.ts](src/tools/clock.ts) | High-precision timing via requestAnimationFrame |
| `Hooks` | [src/tools/hooks.ts](src/tools/hooks.ts) | Hook system (before/after render callbacks) |
| `Listener` | [src/tools/listener.ts](src/tools/listener.ts) | Event listener helper with cleanup |

### Module System Classes

| Class | File | Purpose |
|-------|------|---------|
| `Shader` | [src/tools/shader.ts](src/tools/shader.ts) | Extends Compilable, adds built-in uniform injection |
| `Compilable` | [src/tools/compilable.ts](src/tools/compilable.ts) | Shader preprocessing pipeline - `#import` resolution, namespacing, tree-shaking |
| `Parser` | [src/tools/parser.ts](src/tools/parser.ts) | GLSL parser - extracts imports, uniforms, functions, dependencies |
| `Module` | [src/tools/module.ts](src/tools/module.ts) | Single module definition - extraction, dependency resolution, options |
| `ModuleRegistry` | [src/tools/module_registry.ts](src/tools/module_registry.ts) | Registry of modules - lookup, compilation, option resolution |

### Globals
File: [src/globals.ts](src/globals.ts)

- `modules` - Global `ModuleRegistry` with built-in sandbox modules (immutable, grows with `defineModule`)
- `runtime_modules` - `ModuleRegistry` of modules currently used by the active shader (flushed on shader switch)
- `uniforms` - Map of built-in uniform names → GLSL types (exempt from namespacing)

### Error Classes
Directory: [src/errors/](src/errors/)

| Class | File | Code | When |
|-------|------|------|------|
| `SandboxError` | [base.ts](src/errors/base.ts) | — | Base class for all errors |
| `SandboxWebGLNotSupportedError` | [context.ts](src/errors/context.ts) | `CONTEXT_ERROR` | WebGL not available |
| `SandboxContextCreationError` | [context.ts](src/errors/context.ts) | `CONTEXT_ERROR` | GPU unavailable |
| `SandboxGLSLShaderCompilationError` | [shader.ts](src/errors/shader.ts) | `SHADER_ERROR` | GLSL compilation failed |
| `SandboxShaderVersionMismatchError` | [shader.ts](src/errors/shader.ts) | `VALIDATION_ERROR` | Vertex/fragment version mismatch |
| `SandboxShaderRequirementMismatchError` | [shader.ts](src/errors/shader.ts) | `SHADER_ERROR` | Imported uniform type conflicts with existing |
| `SandboxShaderWithoutFunctionError` | [shader.ts](src/errors/shader.ts) | `SHADER_ERROR` | Shader has no functions |
| `SandboxShaderImportSyntaxError` | [shader.ts](src/errors/shader.ts) | `SHADER_ERROR` | Invalid `#import` syntax |
| `SandboxShaderDuplicateImportNameError` | [shader.ts](src/errors/shader.ts) | `SHADER_ERROR` | Same import name/alias used twice |
| `SandboxModuleNotFoundError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | Module not registered |
| `SandboxModuleMethodNotFoundError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | Function not found in module |
| `SandboxAttemptedToImportMainFunctionError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | Importing `main` is forbidden |
| `SandboxAttemptedToImportDefaultFunctionError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | `default` is reserved |
| `SandboxForbiddenModuleNameError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | User module named `sandbox*` |
| `SandboxOverwriteModuleError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | Duplicate module definition |
| `SandboxMentionUniformNotFoundError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | `@func.uniform` — uniform not in module |
| `SandboxMentionFunctionNotFoundError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | `@func.uniform` — function not imported |
| `SandboxMentionCouldNotBeReplacedError` | [module.ts](src/errors/module.ts) | `MODULE_ERROR` | `@mention` replacement failed |
| `SandboxProgramError` | [program.ts](src/errors/program.ts) | `PROGRAM_ERROR` | Program linking failed |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rosalana/sandbox](https://github.com/rosalana/sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
