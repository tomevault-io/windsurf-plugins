---
trigger: always_on
description: .NES transpiles .NET IL code into 6502 assembly, producing NES ROMs. C# compiles to MSIL, then `Transpiler` reads it via `System.Reflection.Metadata` and emits machine code for the NES.
---

# .NES (dotnes) AI Coding Instructions

## Overview
.NES transpiles .NET IL code into 6502 assembly, producing NES ROMs. C# compiles to MSIL, then `Transpiler` reads it via `System.Reflection.Metadata` and emits machine code for the NES.

## Architecture

### Transpilation Pipeline
```
Program.cs → dotnet build → .dll (MSIL) → Transpiler → .nes ROM
                                ↑                ↓
                           chr_generic.s    IL2NESWriter → NESWriter
```

**Key files:**
- [src/dotnes.tasks/Utilities/Transpiler.cs](src/dotnes.tasks/Utilities/Transpiler.cs) - Single-pass MSIL reader and ROM assembler
- [src/dotnes.tasks/Utilities/IL2NESWriter.cs](src/dotnes.tasks/Utilities/IL2NESWriter.cs) - IL→6502 opcode mapping
- [src/dotnes.tasks/Utilities/NESWriter.cs](src/dotnes.tasks/Utilities/NESWriter.cs) - ROM binary format
- [src/dotnes.tasks/ObjectModel/BuiltInSubroutines.cs](src/dotnes.tasks/ObjectModel/BuiltInSubroutines.cs) - 6502 assembly for all built-in subroutines
- [src/dotnes.tasks/ObjectModel/Program6502.cs](src/dotnes.tasks/ObjectModel/Program6502.cs) - ROM code block ordering and address resolution
- [src/neslib/NESLib.cs](src/neslib/NESLib.cs) - Reference-only API (all methods `throw null!`)

### Reference Assembly Pattern
`neslib` has **no implementations**—methods provide compile-time API only. The transpiler looks up method names to emit corresponding 6502 subroutine calls. Adding new NES APIs requires:
1. Add method stub in `NESLib.cs` with `throw null!`
2. Implement 6502 equivalent in `BuiltInSubroutines.cs`

## Build & Test

```bash
dotnet build                    # Build entire solution
dotnet test                     # Run tests (ALWAYS rebuilds, NEVER use --no-build)
cd samples/hello && dotnet run  # Build + run in emulator
```

**⚠️ IMPORTANT:** Always run `dotnet test` without `--no-build`. The test project depends on build outputs that must be fresh.

**Diagnostic logging:** Add `<NESDiagnosticLogging>true</NESDiagnosticLogging>` to project.

### Diagnostic Scripts

> **💡 Skills available:**
> - `nes-rom-debug` — Static binary analysis: disassembly, byte comparison, IL dump. Use when investigating transpiler output or comparing ROMs.
> - `nes-emu-debug` — Dynamic runtime analysis: run ROMs headlessly in Mesen2 to inspect memory, palette, nametable, screen buffer, CPU/PPU state. Use when a ROM doesn't display correctly or you need to verify runtime behavior.

- `python scripts/disasm.py <file.nes> [start_hex] [end_hex]` — Disassembles PRG ROM into 6502 assembly (default: full `$8000-$FFFF` range)
- `python scripts/compare_rom.py <cc65.nes> <dotnes.nes>` — Side-by-side ROM comparison with 6502 disassembly (useful for debugging transpiler output vs cc65 reference)
- `dotnet run scripts/ildump.cs -- <path-to-dll>` — Dumps IL opcodes from a .NET DLL (useful for understanding what IL the transpiler will process)
- `dotnet run scripts/screenshot-nes.cs -- <rom.nes> [delay_ms] [output.png]` — Launches the ROM in ANESE emulator, captures a screenshot, and saves it as PNG (Windows only, default 3000ms delay)
- `dotnet run scripts/record-nes.cs -- <rom.nes> [--gif] [--frames N] [--interval N] [--delay N] [--output FILE]` — Records a PNG screenshot or animated GIF from a running ROM (Windows only)
- `dotnet run scripts/record-all-samples.cs` — Builds all samples and records PNG/GIF for each into `samples/{name}/{name}.png` or `.gif`
- `python scripts/gen_chr_tileset1.py` — One-time generator that created the `tileset1` CHR ROM `.s` files. Use as a template when adding new samples with custom CHR tilesets.

### MSBuild Binary Logs

Use `dotnet build -bl:output.binlog` to capture binary logs for debugging MSBuild evaluation issues. Analyze with the `binlogtool` .NET global tool:

```bash
dotnet tool install --global binlogtool     # Install once
binlogtool search output.binlog "term"      # Search for properties, items, messages
binlogtool listproperties output.binlog     # List all evaluated properties
binlogtool savestrings output.binlog out.txt # Extract all strings
```

## MSBuild Integration
- [bin/Debug/dotnes.props](bin/Debug/dotnes.props) - Disables BCL (`NoStdLib=true`), forces `Optimize=true`
- [bin/Debug/dotnes.targets](bin/Debug/dotnes.targets) - Runs `TranspileToNES` task after Build

The Transpile target automatically creates `.nes` from `.dll` + `*.s` files.

**⚠️ When adding new public MSBuild properties**, always update [docs/msbuild-properties.md](docs/msbuild-properties.md) with the property name, type, default value, description, and an XML example.

## Testing Patterns
Tests in [src/dotnes.tests/](src/dotnes.tests/) use **Verify snapshots** and **Roslyn-based** tests:
- Test data DLLs live in `Data/` folder (pre-compiled debug/release)
- `TranspilerTests.Write` verifies entire ROM output byte-for-byte
- `TranspilerTests.ReadStaticVoidMain` verifies IL parsing
- `RoslynTests` compile C# source at test time via Roslyn and assert on emitted 6502 bytes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonathanpeppers/dotnes](https://github.com/jonathanpeppers/dotnes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
