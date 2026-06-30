---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

GpDelphiUnits is a collection of open-source Delphi utility units providing Win32/Win64 wrappers, data structures, synchronization primitives, file I/O helpers, and various cross-cutting utilities. This is a library repository - there are no build artifacts, only reusable Pascal units.

## Project Structure

- `src/` - All Pascal unit files (.pas)
- `src/tests/` - Test projects (.dpr, .dproj) for specific units
- `update.bat` - Batch file to synchronize files from another location (x:\gp\common)

## Key Units

### Core Infrastructure
- **DSiWin32.pas** - Extensive Win32/Win64 API wrappers and helper functions (378KB, version 2.09+)
- **GpStuff.pas** - General utilities including TGpBuffer, generic helpers, constants (version 2.29)

### Attribute-Based Frameworks
- **GpAutoCreate.pas** - Automatic field creation/destruction via `[GpManaged]` attribute
- **GpCommandLineParser.pas** - Parse command-line arguments using attributes on class properties

### Data Structures
- **GpLists.pas** - TList descendants and compatible classes (297KB)
- **GpLockFreeQueue.pas** - Lock-free, O(1) threadsafe queue implementation
- **GpStringHash.pas** - Preallocated hash table implementation

### File & Stream I/O
- **GpHugeF.pas** - 64-bit file functions with enhanced functionality (155KB)
- **GpTextFile.pas** - 8/16-bit text file interface (uses GpHugeF)
- **GpStreams.pas** - TStream descendants and helpers (115KB)
- **GpStreamWrapper.pas** - Stream wrapper utilities
- **GpTextStream.pas** - Unicode stream wrapper with automatic encoding detection

### Synchronization
- **GpSync.pas** - Enhanced synchronization primitives (88KB)
- **GpSync.CondVar.pas** - Condition variable implementation
- **SpinLock.pas** - Scalable atomic lock

### Time & Measurement
- **GpTimestamp.pas** - Type-safe timestamp with multiple time sources (TGpTimestamp record)
  - Prevents mixing incompatible time measurements through operator overloading
  - Supports GetTickCount64, QueryPerformanceCounter, and custom timebases
  - All values stored internally in nanoseconds (provides 292 years of range at nanosecond precision)
  - Replaces unsafe raw Int64 time values that could be accidentally mixed

### Specialized Utilities
- **GpSharedMemory.pas** - Shared memory implementation (183KB)
- **GpSharedEvents.pas** - Distributed multicast event manager
- **GpStructuredStorage.pas** - Compound file/embedded file system (112KB)
- **GpVersion.pas** - Version info accessors, modifiers, storage
- **GpSecurity.pas** - Windows NT security wrappers
- **GpSysHook.pas** - System-wide hooks (keyboard, mouse, shell, CBT)

## Testing

Test projects are located in `src/tests/` directory:
- `GpTimestampTests.dpr` - DUnitX tests for GpTimestamp (18 tests covering all features)
- `TestGpAutoCreate.dpr/dproj` - Tests for GpAutoCreate
- `TestGpCommandLineParser.dpr/dproj` - Tests for command line parser
- `TestGpStringHash.dpr/dproj` - Tests for string hash
- `TestCondVar.dpr/dproj` - Tests for condition variables
- `DemoCondVar.dpr/dproj` - Demonstration for condition variables

Individual units may also have standalone test programs (e.g., `GpTimestamp_Test.pas/.exe`).

## Design Patterns & Key Concepts

### Type-Safe Time Measurement (GpTimestamp)

**Problem Solved:**
Traditional time measurement in Delphi has three critical issues:
1. Ambiguous units - Hard to tell if Int64 stores milliseconds, microseconds, or nanoseconds
2. Unsafe mixing - Easy to accidentally subtract QueryPerformanceCounter from GetTickCount values
3. No compile-time safety - Relies only on variable naming conventions (_ms, _us suffixes)

**Design Philosophy:**
- **Fail-fast approach** - Mixing incompatible timebases raises EInvalidOpException immediately
- **Operator overloading** - Automatic compatibility checking on all arithmetic and comparisons
- **Internal precision** - All values stored as nanoseconds (Int64) for maximum precision and future needs

**Type Storage:**
Each TGpTimestamp stores: `(FTimeSource: TTimeSource, FValue: Int64)`
- `TTimeSource` enum: tsNone, tsTickCount, tsQueryPerformanceCounter, tsTimeGetTime, tsStopwatch, tsDateTime, tsDuration
- `FValue`: Time value in nanoseconds
- `tsDuration`: Special source for pure durations/differences, compatible with all other sources

**Compatibility Rules:**
1. `tsDuration` is compatible with everything (allows duration + timestamp arithmetic)
2. Same source type = automatically compatible (e.g., all `FromTickCount` calls)
3. Different sources = raises exception on comparison/arithmetic

**Arithmetic Semantics:**
- `timestamp - timestamp` → duration (with `tsDuration` source)
- `timestamp + duration` → timestamp (preserves timestamp's source)
- `duration + duration` → duration
- `timestamp - duration` → timestamp
- `timestamp + timestamp` → **ERROR** (raises EInvalidOpException)
- `duration - timestamp` → **ERROR** (raises EInvalidOpException)

**Usage Patterns:**

Simple timing with fluent API (Subtract returns TGpTimestamp with tsDuration):
```pascal
var
  start: TGpTimestamp;
  elapsed_ms: Int64;
begin
  start := TGpTimestamp.Now;
  DoWork;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabr42/GpDelphiUnits](https://github.com/gabr42/GpDelphiUnits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
