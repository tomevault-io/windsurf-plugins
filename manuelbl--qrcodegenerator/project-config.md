---
trigger: always_on
description: QrCodeGenerator is a library to generate QR code. It is designed to be easy to use and performant.
---

# CLAUDE.md

## Purpose

QrCodeGenerator is a library to generate QR code. It is designed to be easy to use and performant.

The library only supports limited graphics types and options in order to work without
any graphics libraries, which might not run all platforms.
The library can provide the QR code as a list of rectangles or as a two-dimensional array of pixels
(called modules by the QR code standard). It is then up to the application to display the QR code.
Many demo projects show how to use this approach for different graphics libraries and UI frameworks.

The main target of the library is .NET Standard 2.0 so it runs in virtually any current .NET environment.


## Commands

```bash
# Build
dotnet build
dotnet build --configuration Release

# Test (all)
dotnet test

# Test (single class)
dotnet test --filter "FullyQualifiedName~QrCodeTest"

# Test (single method)
dotnet test --filter "FullyQualifiedName=Net.Codecrete.QrCodeGenerator.Test.QrCodeTest.Constants"

# Pack NuGet
dotnet pack --no-build
```

## Build targets

- **`QrCodeGenerator/`** (the library) targets `netstandard2.0;net6.0`. The `net6.0` target exists only to enable trimming (`IsTrimmable`). Keep public API and language usage compatible with netstandard2.0 — don't reach for newer BCL/`Span` APIs that aren't available there.
- **`QrCodeGeneratorTest/`** (the unit tests) targets `net8.0;net10.0`, plus `net481` on Windows. `dotnet test` runs every target framework.
- **`QrCodeGeneratorProfiling/`** targets `net10.0` (BenchmarkDotNet).
- **`QrCodeAnalyzer/`** is a separate WPF tool in its own solution (`QrCodeAnalyzer/QrCodeAnalyzer.sln`), not part of `QrCodeGenerator.sln`.


## Architecture

`QrCode` is the only substantial public surface: factory methods create immutable `QrCode` instances. They can be rendered to
sVG, PNG, BMP or a list of rectangles. It holds a single `BitMatrix` of modules. Almost all real work lives in `internal` types.


### Encoding pipeline

Text/bytes → segments → codewords → matrix, in this order:

1. **`DataSegment.FromText` / `FromBinaryData`** — chooses the text encoding. For automatic ECI: tries Latin-1 (ISO-8859-1) and adds no ECI; falls back to UTF-8 with an ECI designator if Latin-1 is lossy. Segments retain the *original unencoded bytes* as an `ArraySegment` over the caller's array until the bit stream is built — **the source array must not be mutated** in the meantime.
2. **`SegmentCompaction`** — picks the cheapest per-byte mode (numeric > alphanumeric > Kanji > binary), groups consecutive bytes into blocks, then greedily *merges* adjacent blocks when the mode-switch overhead outweighs the savings. Merge cost depends on `version` (count-indicator width changes at versions 1/10/27). This is what produces the "smallest possible QR code" claim.
3. **`QrCodeBuilder.Build`** is a thin orchestrator that wires the remaining stages, each its own `internal static` module:
   - **`VersionPlanner.Plan`** — smallest version that fits, then boost ECC level for free if it doesn't grow the version. Returns a named `(int Version, int Ecc)`.
   - **`Codewords.BuildData`** — segments → `BitStream` → byte codewords + terminator + 0xEC/0x11 padding.
   - **`Codewords.AddErrorCorrection`** — splits into blocks, computes Reed-Solomon ECC (`ReedSolomon`), interleaves data and ECC codewords per spec.
   - **`MatrixEncoder.Encode`** — matrix layout then mask selection:
     - `FixedPatterns.BuildFixedPatterns` deals with the fixed-pattern geometry of a version: one walk emits both the *drawn* matrix (finder/timing/alignment/version) and the *reserved-module* mask. The reserved mask, inverted, is the *payload-area map* (`GetPayloadAreaMap`). Then `FillPayload` zig-zags the codewords into the free modules.
     - `ApplyBestPattern` — XORs each of the 8 mask patterns, scores it (`Penalty`), keeps the lowest, and draws the format information. `EncodingInfo.ForcedDataMask` can override the choice.

   The ISO/IEC 18004 lookup tables these stages share live in **`QrCodeParameters`**.

### `BitMatrix` and the transpose trick

`BitMatrix` (`internal readonly struct`) is the central data structure: a square bit grid stored as 4 `ulong`s per row (256-bit rows regardless of size), in row-major order. It exposes fast whole-matrix `And`/`Xor`/`Invert`/`PopCount` and an in-place **64×64 block transpose**.

The transpose is load-bearing, not a convenience: every penalty/format rule that operates on *columns* is implemented by transposing the matrix and reusing the *row*-wise algorithm. `ApplyBestPattern` keeps a `modules` and a `transposed` copy in lock-step, and `Penalty` takes both. `Penalty` itself is bit-parallel (operates on whole `ulong` words, not per-module) and has an early-stop path that bails once the running score exceeds the best-so-far.

### Performance-tuned constants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manuelbl/QrCodeGenerator](https://github.com/manuelbl/QrCodeGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
