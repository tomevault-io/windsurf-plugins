---
trigger: always_on
description: kotlinx-rpc is a Kotlin Multiplatform library for building RPC services.
---

# CLAUDE.md

kotlinx-rpc is a Kotlin Multiplatform library for building RPC services.
Version catalog: `versions-root/libs.versions.toml`

## Effort Level
Always operate at maximum effort level. Do not ask about effort preferences.

## Protocols

EXTREMELY IMPORTANT.
This library is a **protocol agnostic** library. It can be called a Kotlin RPC toolkit.
It hosts different protocols:
- kRPC (custom protocol)
- gRPC (HTTP/2 + Protocol Buffers)

They are not TRANSPORTS. Each protocol has its own client and server implementations, as well as code generation patterns.

## Build & Test

**IMPORTANT: Always use Gradle skills** (`running_gradle_builds`, `running_gradle_tests`, `managing_gradle_dependencies`, `introspecting_gradle_projects`, `gradle_expert`, etc.) instead of running `./gradlew` directly. 
Gradle skills provide structured feedback, failure diagnostics, and background orchestration that raw shell commands lack.

### Testing
Use the `running_gradle_tests` skill. Example tasks:
- `<module>:jvmTest --tests "TestClass.testMethod"` -- Single test
- `<module>:jvmTest` -- All JVM tests for a module

### Building
Use the `running_gradle_builds` skill. Example tasks:
- `<module>:build` -- Build a module
- `assemble` -- Assemble all modules

### Validation
Use the `running_gradle_builds` skill for these tasks:
- `checkLegacyAbi` -- Binary compatibility (formerly BCV apiCheck)
- `detekt` -- Static analysis (does NOT fail build, check console)
- `:jpms-check:compileJava` -- Java module system check
- `kotlinUpgradeYarnLock` -- Fix JS dependency locks
- `kotlinWasmUpgradeYarnLock` -- Fix WASM dependency locks

Non-Gradle validation:
```bash
./scripts/validate_published_artifacts.sh -s # Artifact validation (-v verbose, --dump update)
```

### Other specialized commands

```bash
./scripts/publish_local.sh                  # Publish to build/repo/ for local testing
```

## Generated Code — Do Not Edit Manually

Several checked-in files are machine-generated. 
**Never modify them by hand** — always run the appropriate regeneration task and commit the output.
This includes `.api` and `.klib.api` ABI dump files (use `updateLegacyAbi`),
conformance test output, WKT generated code, implicit import lists, and other.
Hand-editing these files produces incorrect results (e.g., klib dump sort order
differs from what's intuitive).

## Module Map

The most important modules have brief READMEs in their directories, read when you need to work with the module.

### Core (multiplatform)
- `:core` -- RPC abstractions (`@Rpc`, `RpcClient`, `RpcServer`, `RpcCall`, descriptors)
- `:utils` -- Shared utilities

### kRPC transport (multiplatform)
- `:krpc:krpc-core`, `:krpc:krpc-client`, `:krpc:krpc-server` -- Protocol, client, server
- `:krpc:krpc-logging`, `:krpc:krpc-test` -- Logging, test utilities
- `:krpc:krpc-serialization:krpc-serialization-{core,json,cbor,protobuf}` -- Serialization formats
- `:krpc:krpc-ktor:krpc-ktor-{core,server,client}` -- Ktor integration

### gRPC transport (experimental KMP)
- `:grpc:grpc-{core,client,server,ktor-server,marshaller,marshaller-kotlinx-serialization}`

### Protobuf (mainly used with gRPC)
- `:protobuf:protobuf-{api,wkt,core}`

### Native shims (standalone Gradle build)
- `native-deps/shims/grpc/` -- gRPC cinterop shim KLIB (`:kotlinx-rpc-grpc-core-shim`)
- `native-deps/shims/protobuf/` -- Protobuf cinterop shim KLIB (`:kotlinx-rpc-protobuf-shim`)
- `native-deps/shims/annotation/` -- Shared opt-in markers (`:kotlinx-rpc-native-shims-annotation`)
- `native-deps/shims/klib-patcher/` -- Internal KLIB metadata patcher
- `native-deps/grpc-c-prebuilt/` -- Prebuilt gRPC static archives and headers

### Included builds (separate Gradle projects)
- `compiler-plugin/` -- Kotlin K2 compiler plugin (FIR diagnostics + IR codegen)
- `gradle-plugin/` -- Gradle plugin (`org.jetbrains.kotlinx.rpc.plugin`)
- `protoc-gen/` -- Protobuf code generator
- `dokka-plugin/` -- Documentation plugin

### Tests (not published)
- `:tests:compiler-plugin-tests` -- Compiler plugin box/diagnostic tests (skipped for Kotlin master builds)
- `:tests:krpc-compatibility-tests`, `:tests:krpc-protocol-compatibility-tests` -- Protocol compat
- `:tests:protobuf-conformance` -- Protobuf spec conformance (not on Windows)
- `:tests:protobuf-unittest`, `:tests:grpc-test-server`, `:tests:test-protos`

## Architecture

### RPC Call Flow
1. User defines `@Rpc` interface with `suspend` methods and/or `Flow` return types
2. **Compiler plugin** generates stub class: each method creates `RpcCall(descriptor, methodName, args, serviceId)` and delegates to `RpcClient.call()` (unary) or `RpcClient.callServerStreaming()` (streaming)
3. **Client** serializes the call and sends over wire
4. **Server** deserializes, looks up `RpcServiceDescriptor`, invokes `RpcInvokator.call(serviceImpl, args)`
5. Response flows back through the same layers
6. Client and Server are bound to RPC protocols (kRPC or gRPC) 

### Key abstractions (`core/src/commonMain/kotlin/kotlinx/rpc/`)
- `@Rpc` annotation on interface -- triggers compiler plugin
- `RpcServiceDescriptor<T>` -- generated per-service metadata + stub factory
- `RpcCallable` -- per-method metadata + `RpcInvokator` (either `UnaryResponse` or `FlowResponse`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kotlin/kotlinx-rpc](https://github.com/Kotlin/kotlinx-rpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
