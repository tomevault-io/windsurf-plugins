---
trigger: always_on
description: You are an expert .NET engineer working on Exceptionless.DateTimeExtensions, a focused utility library providing DateTimeRange, Business Day/Week calculations, Elasticsearch-compatible date math parsing, and extension methods for DateTime, DateTimeOffset, and TimeSpan. Your changes must maintain backward compatibility, correctness across edge cases (especially timezone handling), and parsing reliability. Approach each task methodically: research existing patterns, make surgical changes, and vali
---

# Agent Guidelines for Exceptionless.DateTimeExtensions

You are an expert .NET engineer working on Exceptionless.DateTimeExtensions, a focused utility library providing DateTimeRange, Business Day/Week calculations, Elasticsearch-compatible date math parsing, and extension methods for DateTime, DateTimeOffset, and TimeSpan. Your changes must maintain backward compatibility, correctness across edge cases (especially timezone handling), and parsing reliability. Approach each task methodically: research existing patterns, make surgical changes, and validate thoroughly.

**Craftsmanship Mindset**: Every line of code should be intentional, readable, and maintainable. Write code you'd be proud to have reviewed by senior engineers. Prefer simplicity over cleverness. When in doubt, favor explicitness and clarity.

## Repository Overview

Exceptionless.DateTimeExtensions provides date/time utilities for .NET applications:

- **DateTimeRange** - Parse and manipulate date ranges with natural language, bracket notation, and Elasticsearch date math
- **DateMath** - Standalone Elasticsearch-style date math expression parser with timezone support
- **TimeUnit** - Parse time unit strings (`1d`, `5m`, `100ms`, `1nanos`) into TimeSpan values
- **BusinessDay / BusinessWeek** - Calculate business hours, next business day, and business time spans
- **DateTime Extensions** - Age strings, epoch conversions, start/end of periods, safe arithmetic, navigation helpers
- **DateTimeOffset Extensions** - Mirror of DateTime extensions for DateTimeOffset with offset-aware operations
- **TimeSpan Extensions** - Human-readable formatting (`ToWords`), year/month extraction, rounding, `AgeSpan` struct

Design principles: **parse flexibility**, **timezone correctness**, **comprehensive edge case handling**, **modern .NET features** (targeting net8.0/net10.0).

## Quick Start

```bash
# Build
dotnet build Exceptionless.DateTimeExtensions.slnx

# Test
dotnet test Exceptionless.DateTimeExtensions.slnx

# Format code
dotnet format Exceptionless.DateTimeExtensions.slnx
```

## Project Structure

```text
src
└── Exceptionless.DateTimeExtensions
    ├── BusinessDay.cs                     # Single business day (day of week, start/end time)
    ├── BusinessWeek.cs                    # Business week with time calculations
    ├── DateMath.cs                        # Elasticsearch date math parser (Parse/TryParse)
    ├── DateTimeExtensions.cs              # DateTime extension methods
    ├── DateTimeOffsetExtensions.cs        # DateTimeOffset extension methods
    ├── DateTimeRange.cs                   # Date range with parsing via format parser chain
    ├── TimeSpanExtensions.cs              # TimeSpan extensions and AgeSpan struct
    ├── TimeUnit.cs                        # Time unit string parser (e.g., "1d", "5ms")
    ├── TypeHelper.cs                      # Reflection utility for parser discovery
    └── FormatParsers
        └── FormatParsers
            ├── IFormatParser.cs           # Full-string parser interface
            ├── PriorityAttribute.cs       # Parser ordering attribute
            ├── Helper.cs                  # Shared constants (month names, time names)
            ├── TwoPartFormatParser.cs     # Range parser: [start TO end], start - end
            ├── ExplicitDateFormatParser.cs
            ├── MonthDayFormatParser.cs
            ├── MonthFormatParser.cs
            ├── MonthRelationFormatParser.cs
            ├── NamedDayFormatParser.cs
            ├── RelationAmountTimeFormatParser.cs
            ├── SingleTimeRelationFormatParser.cs
            ├── YearFormatParser.cs
            └── PartParsers
                ├── IPartParser.cs         # Part parser interface (regex + parse)
                ├── DateMathPartParser.cs   # Delegates to DateMath.TryParseFromMatch
                ├── WildcardPartParser.cs   # Handles * for open-ended ranges
                ├── NamedDayPartParser.cs
                ├── AmountTimeRelationPartParser.cs
                ├── ExplicitDatePartParser.cs
                ├── MonthDayPartParser.cs
                ├── MonthPartParser.cs
                ├── MonthRelationPartParser.cs
                ├── SingleTimeRelationPartParser.cs
                └── YearPartParser.cs
tests
└── Exceptionless.DateTimeExtensions.Tests
    ├── DateMathTests.cs                   # Comprehensive date math parsing tests
    ├── DateTimeRangeTests.cs              # Range parsing and manipulation tests
    ├── DateTimeExtensionsTests.cs         # DateTime extension method tests
    ├── TimeSpanExtensionTests.cs          # TimeSpan extension tests
    ├── TimeUnitTests.cs                   # Time unit parsing tests
    ├── BusinessDayTests.cs                # Business day/week calculation tests
    ├── RandomHelper.cs                    # Test utility for random date generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exceptionless/Exceptionless.DateTimeExtensions](https://github.com/exceptionless/Exceptionless.DateTimeExtensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
