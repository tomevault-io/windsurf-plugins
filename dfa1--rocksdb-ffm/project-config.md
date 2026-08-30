---
trigger: always_on
description: This project is heavily AI-driven. As an agent, your goal is to:
---

# AGENTS.md: Project Context & AI-Driven Guidelines

## 🤖 AI-Driven Project Mandate

This project is heavily AI-driven. As an agent, your goal is to:

- **Be Autonomous:** Research C headers (rocksdb/include/rocksdb/c.h) and identify the best mapping to Java FFM.
- **Stay Technical:** Prioritize performance, zero-copy, and manual memory safety.
- **Maintain Consistency:** Follow established naming and ownership patterns.

## 🛠 Tech Stack

- **Language:** Java 25+.
- **Core API:** `java.lang.foreign` (Foreign Function & Memory API).
- **Native Library:** RocksDB (C API via `include/rocksdb/c.h`), built from the `rocksdb/` git submodule (pinned to
  v11.8.1).
- **Native Compiler:** `zig cc` / `zig c++` — used as a drop-in C/C++ compiler via
  `CC="zig cc" CXX="zig c++" PORTABLE=1 make shared_lib`. Zig bundles clang + libc++ for every target, enabling
  cross-compilation without a separate sysroot.
- **Build System:** Maven Wrapper (`./mvnw`). `./mvnw generate-resources` (or `test`, `compile`, ...) auto-detects the
  host OS/arch and cross-compiles RocksDB for just that one `native/*` classifier via zig cc — a plain local build no
  longer compiles all 5 targets. Add `-Pall-natives` to build every classifier regardless of host (what CI and
  releases use). Use `./mvnw` (not `mvn`) to ensure the correct Maven version is used.
  **NEVER run `mvn install` or `./mvnw install`** — it pollutes `~/.m2` with local artifacts. Use `compile`, `test`, or `package` instead.
- **Testing:** JUnit 5, AssertJ.
- **Benchmarking:** JMH (Java Microbenchmark Harness).

## 🏗 Architectural Standards

### 1. Manual Memory Management & Lifecycle

Every class wrapping a native pointer **must** implement `AutoCloseable`.

- **Zero Leaks:** Native resources must be destroyed in `close()`.
- **Ownership Transfer:** When one native object takes ownership of another (e.g., `FilterPolicy` →
  `BlockBasedTableOptions`), the transferred object must be marked so its `close()` becomes a no-op and cannot
  double-free.
- **Transfer Marker:** Call `transferOwnership()` (package-private on `NativeObject`) inside the setter that takes
  ownership. It sets the held pointer to `MemorySegment.NULL`, which makes `close()` a no-op and any later `ptr()`
  throw `IllegalStateException`.

### 2. Data Types & Path Handling

To ensure type safety and consistent units across the API:

- **C API Only:** We use the RocksDB C interface (`rocksdb/c.h`). Do not attempt to link directly to C++ symbols.
- **Read-only headers:** NEVER modify system include files (e.g. `/opt/homebrew/...`, `/usr/include/...`). They are
  read-only references; all mappings live in Java source.
- **Library loading:** `NativeLibrary.java` loads the native library from the classpath resource
  `/native/<os>-<arch>/librocksdb.<ext>` (bundled by each `native/*` module's `exec-maven-plugin` execution). There is
  no brew/system fallback. NEVER add hardcoded system paths back.
- **Paths:** Never use raw `String` for file system paths. Always use `java.nio.file.Path` for any API surface that
  accepts paths (open, backup, checkpoint).
- **Memory Sizes:** Never use raw `long` for byte counts (e.g., cache size, write buffer size). Always use the project's
  `MemorySize` type.
- **Sequence Numbers:** Never use raw `long` for RocksDB sequence numbers. Always use the project's `SequenceNumber`
  type.
- **BackupId:**: Never use raw uint32, use a wrapper Java type that hides this from the user.
- **Timeouts:** Never use raw `long` for a timeout field with a negative-sentinel meaning in the C API (e.g. `-1` =
  wait forever/disabled). Use `Duration`, with `null` — not `Duration.ZERO` — as the sentinel, since `ZERO` usually
  already means something else ("fail immediately"). Reject a non-null negative `Duration`. Verify sentinel semantics
  against the actual `rocksdb/utilities/**/*.h`/`.cc` source, not just existing javadoc (it can be wrong); if the C++
  side documents no negative-sentinel meaning, still convert to `Duration` but require it non-null.

### 3. API Surface Design

For every feature, provide three tiers of access:

1. **`MemorySegment` Version:** Native-first, for performance-critical usage.
2. **`ByteBuffer` Version:** For compatibility with existing NIO-based clients.
3. **`byte[]` Version:** Quick access for convenience (explicitly documented as slower).

## ⚡ FFM Performance & Patterns

### 1. Centralized Error Handling

**NEVER use ThreadLocals for error pointers.** Use the shared helpers on `RocksDB` with the caller's `Arena`:

```java
try (Arena arena = Arena.ofConfined()) {
    MemorySegment err = RocksDB.errHolder(arena);
    MH_DO_SOMETHING.invokeExact(handle, ..., err);
    RocksDB.checkError(err);
} catch (Throwable t) {
    throw RocksDB.wrapInvokeFailure("doSomething failed", t);
}
```

`RocksDB.errHolder`, `RocksDB.checkError`, `RocksDB.toNative`, and `RocksDB.wrapInvokeFailure` are the shared FFM
plumbing used by every wrapper class.

**`RocksDBException` is only ever constructed by `RocksDB.checkError`**, for a genuine `errptr`-reported RocksDB
error. Never construct it — or call something you wrote yourself that would — from an `invokeExact` catch block;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfa1/rocksdb-ffm](https://github.com/dfa1/rocksdb-ffm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
