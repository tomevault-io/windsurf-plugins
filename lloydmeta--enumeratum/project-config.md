---
trigger: always_on
description: This document provides comprehensive guidance for AI agents (Claude, GitHub Copilot, etc.) working on the Enumeratum codebase.
---

# Enumeratum - AI Agent Guide

This document provides comprehensive guidance for AI agents (Claude, GitHub Copilot, etc.) working on the Enumeratum codebase.

## Project Overview

**Enumeratum** is a type-safe enumeration library for Scala that provides:
- Exhaustive pattern matching with compiler warnings
- 4x faster performance than stdlib Enumeration
- Zero dependencies in core
- Full ScalaJS, ScalaNative, and JVM support
- Rich integrations with popular Scala libraries

**Current Version**: 1.9.8-SNAPSHOT (stable: 1.9.7)
**License**: MIT
**Scala Versions**: 2.12.21, 2.13.18, 3.3.8
**Main Branch**: `master`

## Architecture

### Module Structure

```
enumeratum/
├── macros/                    # Compile-time value discovery (core dependency)
├── enumeratum-core/           # Base Enum and ValueEnum traits
├── enumeratum-test/           # Shared test models
└── enumeratum-{integration}/  # Library integrations (play, circe, cats, etc.)
```

### Three-Layer Architecture

1. **Macros Layer** (`/macros`)
   - Compile-time discovery of enum values via `findValues`
   - Scala 2: Uses `scala.reflect.macros`
   - Scala 3: Uses `scala.quoted` API
   - Version-specific: `/src/main/scala-2/` and `/src/main/scala-3/`

2. **Core Layer** (`/enumeratum-core`)
   - `EnumEntry`: Base trait with `entryName`
   - `Enum[A]`: String-based lookup (withName, indexOf)
   - `ValueEnum[ValueType, EntryType]`: Value-based lookup (withValue)
   - Specialized: IntEnum, LongEnum, ShortEnum, ByteEnum, CharEnum, StringEnum

3. **Integration Layer** (various modules)
   - JSON: play-json, circe, argonaut, json4s
   - Web: play (bindables, forms)
   - Database: slick, quill, doobie, reactivemongo
   - Typeclasses: cats, scalacheck

### Cross-Platform Support

Most modules use `crossProject(JSPlatform, JVMPlatform, NativePlatform)` with `CrossType.Pure`:
- JVM/JS/Native share identical source code
- No platform-specific code in most modules
- Exceptions: Play (JVM only), some integrations (JVM+JS only)

## Development Conventions

### Code Style

**Formatter**: Scalafmt 3.8.6 (config: `.scalafmt.conf`)
```bash
sbt scalafmtAll scalafmtSbt
```

**Settings**:
- Max column: 100
- Style: defaultWithAlign
- Dialect: Scala 2.13 syntax targeting Scala 3 (Scala213Source3)

### Compiler Flags

**Always**: `-Xfatal-warnings` (all warnings are errors)

**Scala 2 Key Flags**:
- `-language:higherKinds`, `-language:implicitConversions`
- `-Ywarn-dead-code`, `-Ywarn-value-discard`
- `-Xlint`

**Scala 3 Key Flags**:
- `-Yretain-trees` (REQUIRED for ValueEnums to work)

### Naming Conventions

**Projects**:
- Main modules: `enumeratum-{feature}` (e.g., `enumeratum-circe`)
- Aggregates: `{feature}-aggregate` (e.g., `circe-aggregate`)
- Cross-projects auto-suffixed: `coreJVM`, `coreJS`, `coreNative`

**Packages**:
- Main: `enumeratum`
- Values: `enumeratum.values`

**Traits**:
- Base: `Enum[A]`, `ValueEnum[V, E]`
- Entries: `EnumEntry`, `IntEnumEntry`, etc.
- Integrations: Library-prefixed (e.g., `PlayEnum`, `CirceEnum`)

### Version-Specific Code

**Source Layout**:
```
/src/main/scala/        # Common code (all versions)
/src/main/scala-2/      # Scala 2.x only
/src/main/scala-3/      # Scala 3.x only
/compat/src/main/scala-2.12/  # Scala 2.12 specific
/compat/src/main/scala-2.13/  # Scala 2.13+ specific
```

**Pattern**: Isolate version-specific code in separate directories rather than using version checks.

## Common Patterns

### Sealed Trait Enum Pattern

```scala
sealed trait MyEnum extends EnumEntry
object MyEnum extends Enum[MyEnum] {
  val values = findValues  // Macro discovers case objects at compile-time

  case object Value1 extends MyEnum
  case object Value2 extends MyEnum
}
```

### ValueEnum Pattern

```scala
sealed abstract class MyEnum(val value: Int) extends IntEnumEntry
object MyEnum extends IntEnum[MyEnum] {
  val values = findValues  // Validates uniqueness at compile-time

  case object Value1 extends MyEnum(1)
  case object Value2 extends MyEnum(2)
}
```

### Name Transformations

Stackable traits for automatic name transformation:
```scala
sealed trait Color extends EnumEntry with Snakecase
// or: Uppercase, Lowercase, UpperSnakecase, Hyphencase, etc.
```

**Important**: All transformations cached in lazy vals for performance.

### Error Handling

- **Throwing**: `withName`, `withValue`
- **Option**: `withNameOption`, `withValueOpt`
- **Either**: `withNameEither`, `withValueEither` (returns `NoSuchMember[A]`)

## Build System (SBT)

**SBT Version**: 1.12.11+

The project requires SBT 1.12.11 or later. Install via:
```bash
sdk install sbt 1.12.11
# or update project/build.properties
```

### Key Files

- `/build.sbt` - Main build (898 lines)
- `/project/plugins.sbt` - SBT plugins
- `/project/Versions.scala` - Version management
- `/project/CoreJVMTest.scala` - Generated test logic

### Cross-Compilation

```bash
# Test specific version
sbt "++2.13.18 test"

# Test all versions
sbt "+test"

# Compile for all platforms
sbt "coreJVM/test" "coreJS/test" "coreNative/test"
```

### Local Version Testing

To test with local macro changes (important when modifying macros):
```bash
sbt -Denumeratum.useLocalVersion "++2.13.18 test"
```

This is set in CI via `SBT_OPTS`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lloydmeta/enumeratum](https://github.com/lloydmeta/enumeratum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
