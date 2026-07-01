---
trigger: always_on
description: **Current development**: Target against main branch and compare changes against it.
---

# CLAUDE.md

**Current development**: Target against main branch and compare changes against it.

## Build & Test Commands

### Building
```bash
# Build entire solution
dotnet build --configuration Release

# Build specific project
dotnet build --configuration Release Jint/Jint.csproj
```

Important: Never use --no-build - always ensure you are working with the latest compiled code.

### Testing
```bash
# Run all tests
dotnet test --configuration Release

# Run specific test project
dotnet test --configuration Release Jint.Tests/Jint.Tests.csproj

# Run specific test class
dotnet test --configuration Release --filter "FullyQualifiedName~Jint.Tests.Runtime.EngineTests"

# Run specific test method
dotnet test --configuration Release --filter "FullyQualifiedName~Jint.Tests.Runtime.EngineTests.CanEvaluateScripts"

# Run Test262 conformance tests
dotnet test --configuration Release Jint.Tests.Test262/Jint.Tests.Test262.csproj
```

Important: Run tests in Release mode for faster feedback loop.

## Requirements

- The project uses central package management via Directory.Packages.props

## Architecture

Jint follows a layered interpreter architecture:

```
Acornima Parser (external) → AST → Interpreter → Runtime → Interop
```

### Core Components

**Engine (`Jint/Engine.cs`)**
- Main API entry point split across multiple partial files (Engine.cs, Engine.Advanced.cs, Engine.Modules.cs, etc.)
- Manages execution context stack, intrinsics (built-in objects), and realms
- Configuration via `Options` class (constraints, CLR interop, modules, debugging)

**JsValue Type System (`Jint/Native/`)**
- `JsValue` is the abstract base class for all JavaScript values
- Primitive types: JsUndefined, JsNull, JsBoolean, JsNumber, JsString, JsBigInt, JsSymbol
- Object types: ObjectInstance and derived classes (JsArray, JsDate, JsMap, JsSet, JsPromise, etc.)
- All built-in JavaScript objects are in `Jint/Native/` organized by type (Array/, Date/, Error/, Function/, etc.)

**Runtime (`Jint/Runtime/`)**
- **TypeConverter.cs** (1,048 lines): All JavaScript type coercion (ToPrimitive, ToNumber, ToString, ToObject, etc.)
- **Intrinsics.cs**: Singleton containing all built-in object constructors and prototypes (lazily initialized)
- **Realm.cs**: ECMAScript Realm encapsulating global environment and intrinsics
- **Environments/**: Scope chain implementation (GlobalEnvironment, FunctionEnvironment, DeclarativeEnvironment, etc.)

**Interpreter (`Jint/Runtime/Interpreter/`)**
- **Statements/**: One handler class per statement type (JintIfStatement, JintForStatement, JintTryStatement, etc.)
- **Expressions/**: One handler class per expression type (JintBinaryExpression, JintCallExpression, JintMemberExpression, etc.)
- AST nodes are evaluated via specialized Jint* handler classes
- Caching: JintFunctionDefinition caches compiled function metadata for reuse

**Interop (`Jint/Runtime/Interop/`)**
- **ObjectWrapper.cs**: Wraps .NET objects for JavaScript access
- **TypeReference.cs**: Exposes CLR types to JavaScript (e.g., `System.String`)
- **ClrFunction.cs**: Wraps .NET methods/delegates as JavaScript functions
- **DefaultTypeConverter.cs**: Bidirectional conversion between JS values and CLR types
- **Reflection/**: Type discovery and method binding with caching

**Modules (`Jint/Runtime/Modules/`)**
- ES6 module system with import/export support
- ModuleLoader handles module resolution and loading
- Supports cyclic dependencies via CyclicModule
- Modules can be defined from JavaScript source or programmatically via ModuleBuilder

### Important Patterns

**Lazy Initialization**
- Built-in objects (Intrinsics) are lazily initialized to reduce startup time
- Properties are typically null until first access

**Object Pooling (`Jint/Pooling/`)**
- Reference, Arguments, and JsValue arrays are pooled to reduce GC pressure
- Use `ReferencePool`, `ArgumentsInstancePool`, `JsValueArrayPool`

**Property Key Optimization**
- `KnownKeys.cs` contains pre-computed common property names
- `HybridDictionary` switches between list and hash based on property count
- `StringDictionarySlim` for string-only dictionary keys

**Partial Classes**
- Large classes are split: Engine.*.cs, Intrinsics.*.cs, ObjectInstance.*.cs, etc.
- Keep related functionality together when editing

**ECMAScript Spec References**
- Code includes TC39 spec section references in comments (e.g., `// https://tc39.es/ecma262/#sec-...`)
- Maintain these references when implementing new features
- The to follow TC 39 spec when possible
- The test files are located in ..\test262\test when source code needed

**Type Flags**
- `InternalTypes` enum enables fast type checking without casting
- Many hot paths use type flags for performance

## ECMAScript Compliance
- Follow ECMAScript specification behavior as closely as practical
- Do not introduce non-standard language extensions
- Support both strict and sloppy mode with spec-defined differences

## Code Conventions

- **Namespaces**: Global usings for Acornima and Acornima.Ast defined in Directory.Build.props
- **Nullable**: Enabled across the codebase (NRT)
- **Unsafe Code**: Allowed (used for performance-critical paths)
- **Warnings as Errors**: Enabled - all warnings must be fixed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebastienros/jint](https://github.com/sebastienros/jint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
