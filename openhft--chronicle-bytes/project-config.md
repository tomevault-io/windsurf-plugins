---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Chronicle Bytes is a low-level memory access library that provides a high-performance alternative to Java's `ByteBuffer`. It offers off-heap memory management with deterministic resource cleanup, support for 63-bit sizes, and rich APIs for reading/writing primitives, strings (UTF-8/ISO-8859-1), and complex data structures.

**Key concepts:**
- **Bytes vs BytesStore**: `Bytes` instances are elastic and track read/write positions; `BytesStore` instances have fixed capacity and no position tracking
- **Off-heap memory**: Most implementations work with native memory outside the Java heap
- **Reference counting**: All off-heap resources must be explicitly released via `releaseLast()` or similar
- **Flyweight pattern**: Bytes objects act as views over underlying memory

## Build Commands

### Basic Build and Test

```bash
# Clean build with tests
mvn clean verify

# Build without tests (faster iteration)
mvn clean install -DskipTests

# Quiet mode (less output)
mvn -q clean verify
```

### Running Single Tests

```bash
# Run specific test class
mvn -Dtest=BytesTest test

# Run specific test method
mvn -Dtest=BytesTest#testAllocateElasticDirect test
```

### Code Quality Checks

```bash
# Run Checkstyle (checks coding standards)
mvn checkstyle:check

# Run SpotBugs (static analysis)
mvn spotbugs:check

# Quality profile with all checks
mvn -P quality clean verify

# Code coverage with JaCoCo
mvn -P sonar clean verify
```

### Benchmarks

```bash
# Run microbenchmarks
mvn -P run-benchmarks clean test
```

## Project Structure

```
src/main/java/net/openhft/chronicle/bytes/
  ├── Bytes.java              # Main interface - elastic, position-aware
  ├── BytesStore.java         # Fixed-size memory block interface
  ├── BytesIn.java            # Read operations interface
  ├── BytesOut.java           # Write operations interface
  ├── BytesMarshallable.java  # Serialization support
  ├── MappedBytes.java        # Memory-mapped file wrapper
  ├── NativeBytes.java        # Off-heap implementation
  ├── VanillaBytes.java       # Standard implementation
  ├── HexDumpBytes.java       # Debug wrapper with hex output
  ├── algo/                   # Algorithms (hashing, compression)
  ├── internal/               # Internal implementation classes
  ├── pool/                   # Object pooling
  ├── ref/                    # Reference types
  └── util/                   # Utility classes

src/main/docs/                # AsciiDoc documentation (canonical location)
  ├── project-requirements.adoc
  ├── architecture-overview.adoc
  ├── decision-log.adoc
  └── security-review.adoc
```

## Architecture Principles

### Memory Management
- Chronicle Bytes uses **reference counting** for deterministic cleanup of off-heap resources
- Always call `bytes.releaseLast()` when done (or use try-with-resources)
- Tests MUST use `assertReferencesReleased()` from `Chronicle-Test-Framework` to verify cleanup

### Position Tracking
Every `Bytes` instance maintains four key positions:
- `readPosition`: where to read from next
- `writePosition`: where to write to next
- `readLimit`: maximum position that can be read
- `writeLimit`: maximum position that can be written

Unlike `ByteBuffer`, you don't need to flip between reading and writing.

### Threading
- `Bytes` instances are NOT thread-safe by default
- `BytesStore` can be shared across threads if data access is synchronized
- Atomic operations (CAS, volatile reads/writes) are available for `int`, `long`, `float`, `double`

### Encoding
- **Binary encoding**: Fixed-width primitives, stop-bit compression
- **Text encoding**: Parsing and appending primitives as text
- **String encoding**: Both ISO-8859-1 (8-bit) and UTF-8 supported
- **Stop-bit encoding**: Variable-length compression (see https://github.com/OpenHFT/RFC/blob/master/Stop-Bit-Encoding/Stop-Bit-Encoding-1.0.adoc)

## Common Development Tasks

### Creating Bytes Instances

```java
// On-heap, elastic
Bytes<byte[]> bytes = Bytes.allocateElasticOnHeap();

// Off-heap, elastic (must release)
Bytes<?> bytes = Bytes.allocateElasticDirect();
try {
    // use bytes
} finally {
    bytes.releaseLast();
}

// Memory-mapped file
MappedBytes bytes = MappedBytes.mappedBytes(file, chunkSize);
```

### Reading and Writing

```java
// Binary primitives
bytes.writeInt(42);
bytes.writeLong(123L);
int value = bytes.readInt();

// With explicit positions (random access)
bytes.writeInt(offset, 42);
int value = bytes.readInt(offset);

// Strings
bytes.writeUtf8("hello");
bytes.write8bit("world");
String s = bytes.readUtf8();

// Stop-bit compressed
bytes.writeStopBit(1234567L);
long value = bytes.readStopBit();
```

### Testing Resource Cleanup

```java
@Test
public void testBytesCleanup() {
    Bytes<?> bytes = Bytes.allocateElasticDirect();
    bytes.writeInt(42);
    bytes.releaseLast();

    // Verify all off-heap resources released
    assertReferencesReleased();
}
```

## Code Style Requirements

### Language and Character Set
- **British English** spelling (`synchronise`, `behaviour`, `colour`)
- **ISO-8859-1** characters only - no smart quotes, em-dashes, or Unicode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHFT/Chronicle-Bytes](https://github.com/OpenHFT/Chronicle-Bytes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
