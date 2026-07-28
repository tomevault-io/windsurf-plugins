---
trigger: always_on
description: Handles `@LengthPrefixed`, `@LengthFrom`, `@WhenTrue` (including value class property conditions), `@WireBytes`, sealed dispatch, `@DispatchOn` (value class and data class discriminators), nested messages, and multiple variable-length fields.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ByteBuffer is a Kotlin Multiplatform library providing platform-agnostic byte buffer management with an API similar to Java's ByteBuffer. It delegates to native implementations on each platform to avoid memory copies.

**Package:** `com.ditchoom.buffer`

## Build Commands

```bash
# Build for all platforms
./gradlew build

# Run tests
./gradlew allTests                # Run tests for all platforms (aggregated report)
./gradlew check                   # Run all checks (tests + linting)
./gradlew test                    # Common/JVM tests
./gradlew connectedCheck          # Android instrumented tests (requires emulator)

# Linting
./gradlew ktlintCheck             # Check code style
./gradlew ktlintFormat            # Auto-format code

# Run specific test class
./gradlew :jvmTest --tests "com.ditchoom.buffer.BufferTests"
./gradlew :jsNodeTest --tests "com.ditchoom.buffer.BufferTests"

# Codec processor tests (KSP compile-time validation)
./gradlew :buffer-codec-processor:test

# Codec integration tests (protocol round-trips: TLS, PNG, RIFF, WebSocket, MQTT)
./gradlew :buffer-codec-test:jvmTest
```

## Architecture

### Kotlin Multiplatform Structure

The project uses the expect/actual pattern with platform-specific implementations:

```
src/
├── commonMain/          # Shared interfaces (PlatformBuffer, ReadBuffer, WriteBuffer)
├── commonTest/          # Shared tests run on all platforms
├── jvmCommonMain/       # Shared JVM/Android: BaseJvmBuffer, CharsetEncoderHelper
├── jvmMain/             # JVM: HeapJvmBuffer, DirectJvmBuffer, JvmBuffer
├── androidMain/         # Android: extends JVM + SharedMemory/Parcelable IPC
├── appleMain/           # iOS/macOS/watchOS/tvOS: MutableDataBuffer (NSMutableData)
├── jsMain/              # Browser/Node.js: JsBuffer (Int8Array, SharedArrayBuffer support)
├── wasmJsMain/          # WASM: LinearBuffer (native memory) + ByteArrayBuffer (heap)
├── nonJvmMain/          # Shared native/WASM: ByteArrayBuffer
└── nativeMain/          # Linux/Apple native: uses nonJvmMain
```

### Buffer Types by Platform

| Platform | Heap (wrap/Heap zone) | Direct (allocate) | Shared Memory |
|----------|----------------------|-------------------|---------------|
| JVM | `HeapJvmBuffer` | `DirectJvmBuffer` | Falls back to Direct |
| Android | `HeapJvmBuffer` | `DirectJvmBuffer` | `ParcelableSharedMemoryBuffer` |
| Apple | `ByteArrayBuffer` | `MutableDataBuffer` | Falls back to Direct |
| JS | `JsBuffer` | `JsBuffer` | `JsBuffer` (SharedArrayBuffer) |
| WASM | `ByteArrayBuffer` | `LinearBuffer` | Falls back to Direct |
| Linux | `ByteArrayBuffer` | `ByteArrayBuffer` | Falls back to Direct |

### Memory Access Interfaces

- `NativeMemoryAccess` - Direct native memory pointer (DirectJvmBuffer, MutableDataBuffer, LinearBuffer, JsBuffer, NativeBuffer)
- `ManagedMemoryAccess` - Kotlin ByteArray backing (HeapJvmBuffer, ByteArrayBuffer, JsBuffer)
- `SharedMemoryAccess` - Cross-process shared memory (ParcelableSharedMemoryBuffer, JsBuffer with SharedArrayBuffer)

### Native Data Conversions

Convert buffers to platform-native types for interop with platform APIs:

```kotlin
// Get native memory handle (returns NativeData wrapper)
val nativeData: NativeData = buffer.toNativeData()

// Get mutable native memory handle (returns MutableNativeData wrapper)
val mutableData: MutableNativeData = buffer.toMutableNativeData()

// Get managed memory (guarantees ManagedMemoryAccess)
val bytes = buffer.toByteArray()
```

**Accessing platform-specific types:**

```kotlin
// JVM/Android
val byteBuffer: ByteBuffer = buffer.toNativeData().byteBuffer

// Apple
val nsData: NSData = buffer.toNativeData().nsData

// JS
val arrayBuffer: ArrayBuffer = buffer.toNativeData().arrayBuffer
val int8Array: Int8Array = buffer.toMutableNativeData().int8Array

// WASM
val linearBuffer: LinearBuffer = buffer.toNativeData().linearBuffer

// Linux
val nativeBuffer: NativeBuffer = buffer.toNativeData().nativeBuffer
```

**Mental model:**
- `toNativeData()` / `toMutableNativeData()` → guarantees native memory (direct ByteBuffer, NSData, NativeBuffer, etc.)
- `toByteArray()` → guarantees managed memory (Kotlin ByteArray)

**Zero-copy vs Copy:**
- Zero-copy when source already matches target type (e.g., direct buffer → direct ByteBuffer)
- Copies when conversion is needed (e.g., heap buffer → direct ByteBuffer)

**Platform wrapper contents:**

| Platform | `NativeData` contains | `MutableNativeData` contains | `toByteArray()` |
|----------|----------------------|------------------------------|-----------------|
| JVM | `ByteBuffer` (direct, read-only) | `ByteBuffer` (direct) | `ByteArray` |
| Android | `ByteBuffer` (direct, read-only) | `ByteBuffer` (direct) | `ByteArray` |
| Apple | `NSData` | `NSMutableData` | `ByteArray` |
| JS | `ArrayBuffer` | `Int8Array` | `ByteArray` |
| WASM | `LinearBuffer` | `LinearBuffer` | `ByteArray` |
| Linux | `NativeBuffer` | `NativeBuffer` | `ByteArray` |

**Apple-specific helpers:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DitchOoM/buffer](https://github.com/DitchOoM/buffer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
