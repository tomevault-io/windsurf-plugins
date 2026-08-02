---
trigger: always_on
description: **SequentialGuid** is a .NET library that generates database-friendly, time-ordered UUIDs
---

# CLAUDE.md

## Project Overview

**SequentialGuid** is a .NET library that generates database-friendly, time-ordered UUIDs
compliant with [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html). It eliminates clustered
index fragmentation in SQL Server and other databases, embeds timestamps directly in the UUID,
and supports client-side ID generation without a server round-trip.

### Packages

| Package | Description |
|---|---|
| `SequentialGuid` | Core library — zero dependencies |
| `SequentialGuid.EntityFrameworkCore` | EF Core value converters and conventions |
| `SequentialGuid.MongoDB` | MongoDB BSON serializers |
| `SequentialGuid.NodaTime` | NodaTime `Instant`/`ZonedDateTime` overloads |

## Development Workflow — Use Superpowers

This repo has the `superpowers` plugin enabled (`.claude/settings.json`). It's not decorative —
use the skill tree for real work here, and it maps directly onto the spec-first, plan-second,
code-last workflow:

| Phase | Skill |
|---|---|
| Design/spec iteration | `brainstorming` |
| Plan once the spec settles | `writing-plans` |
| Implementation | `executing-plans` (pairs with `test-driven-development`) |
| Bug fixes (spec-first exception) | `systematic-debugging` |
| Before calling anything done | `verification-before-completion` |
| Opening/handling a PR | `requesting-code-review` / `receiving-code-review` |
| Wrapping up a branch | `finishing-a-development-branch` |

If a skill applies to what you're doing, invoke it — don't just read the table and proceed
manually. The transition points (spec → plan, plan → code) are still explicit human decisions,
per the global CLAUDE.md; the skills are how each phase gets executed, not a way around the
hand-off.

## Repository Structure

    SequentialGuid/
    ├── src/
    │   ├── SequentialGuid/                                # Core library
    │   │   ├── GuidV4.cs                                  # Cryptographically random UUID (v4)
    │   │   ├── GuidV5.cs                                  # Deterministic UUID using SHA-1 (v5)
    │   │   ├── GuidV7.cs                                  # Time-ordered UUID, 48-bit Unix ms + 26-bit counter (v7)
    │   │   ├── GuidV8Name.cs                              # Deterministic UUID using SHA-256 (v8)
    │   │   ├── GuidV8Time.cs                              # Time-ordered UUID, 60-bit .NET Ticks (v8)
    │   │   ├── SequentialGuid.cs                          # Strongly-typed struct wrapper (ISequentialGuid<T>)
    │   │   ├── SequentialSqlGuid.cs                       # SQL Server byte-order struct wrapper
    │   │   └── Extensions/
    │   │       ├── ByteArrayExtensions.cs                 # Internal: byte order swaps, RFC bit helpers
    │   │       └── GuidExtensions.cs                      # Public: ToDateTime(), ToSqlGuid(), etc.
    │   ├── SequentialGuid.EntityFrameworkCore/
    │   ├── SequentialGuid.MongoDB/
    │   └── SequentialGuid.NodaTime/
    ├── tests/
    │   ├── unit/
    │   │   ├── SequentialGuid.Tests/                      # xUnit test project (core library)
    │   │   ├── SequentialGuid.EntityFrameworkCore.Tests/
    │   │   ├── SequentialGuid.MongoDB.Tests/
    │   │   └── SequentialGuid.NodaTime.Tests/
    │   └── smoke/
    │       ├── SequentialGuid.AotSmokeTest/                # PublishAot smoke test (core + NodaTime)
    │       └── SequentialGuid.EntityFrameworkCore.AotSmokeTest/
    ├── utils/
    │   └── Generator/                                     # Emits RFC test-vector [InlineData] lines for GuidV7/GuidV8Time
    ├── benches/
    │   └── Benchmarks/                                    # BenchmarkDotNet benchmarks
    └── test.sh                                            # Linux/WSL2 test runner — see Testing below

## Target Frameworks

`global.json` pins the repo to the **.NET 11 preview SDK** (`allowPrerelease: true`,
`rollForward: latestFeature`) — this is a standing pin for this repo, not a temporary
side-channel install.

Most projects multi-target:

- `net11.0`, `net10.0`, `net9.0`, `net8.0` (modern .NET)
- A legacy framework leg — **not the same TFM in `src/` and `tests/`:**
  - `src/`: `net462`
  - `tests/unit/`: `net472`
- `netstandard2.0` (`src/` only)

**Exception:** `SequentialGuid.EntityFrameworkCore` (and its test project) targets only
`net10.0;net9.0;net8.0` — no `net11.0`, no legacy framework leg, no `netstandard2.0`. This is
deliberate (EF Core doesn't support .NET Framework and net11 EF Core support wasn't out yet when
last checked), not an oversight — don't "fix" it to match the other packages without checking
upstream EF Core support first.

Use `#if NET6_0_OR_GREATER` (or the appropriate TFM guard) to separate modern and legacy code
paths. Always provide both paths — do **not** drop legacy support.

## Key Design Patterns

### RFC 9562 Byte Layout

UUIDs are built in **network (big-endian) byte order** per RFC 9562, then converted to .NET's
mixed-endian `Guid` format using `new Guid(bytes, bigEndian: true)` on NET6+ or
`bytes.SwapByteOrder()` on legacy targets.

### SQL Server Byte Ordering

`ToSqlGuid()` / `NewSqlGuid()` reorder bytes so SQL Server's `uniqueidentifier` comparison sorts
correctly. The mapping is defined in `ByteArrayExtensions` and follows the documented SQL Server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buvinghausen/SequentialGuid](https://github.com/buvinghausen/SequentialGuid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
