---
trigger: always_on
description: This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.
---

# Agent Guide for Telegram Desktop

This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.

## Build System Structure

The build system expects this directory layout:

```text
L:\Telegram\                    # BuildPath
L:\Telegram\tdesktop\           # Repository (you work here)
L:\Telegram\Libraries\          # 32-bit dependencies (Linux/macOS)
L:\Telegram\win64\Libraries\    # 64-bit dependencies (Windows)
L:\Telegram\ThirdParty\         # Build tools (NuGet, Python, etc.)
```

Dependencies are located relative to the repository: `../Libraries`, `../win64/Libraries`, or `../ThirdParty`.

## Build Configuration

### Build Commands

**From repository root, run:**

```bash
cmake --build out --config Debug --target Telegram
```

That's it. The `out/` directory is already configured. The executable will be at `out/Debug/Telegram.exe`.

**Important:** When running cmake from a shell that doesn't support `cd`, use quoted absolute paths:
```bash
cmake --build "l:\Telegram\tx64\out" --config Debug --target Telegram
```

**Never build Release** - it's extremely heavy and not needed for testing changes.

## Platform-Specific Requirements

### Windows
- Requires Visual Studio 2022
- Must run from appropriate Native Tools Command Prompt:
  - "x64 Native Tools Command Prompt" for `win64`
  - "x86 Native Tools Command Prompt" for `win`
  - "ARM64 Native Tools Command Prompt" for `winarm`
- Dependencies: `../win64/Libraries` (64-bit) or `../Libraries` (32-bit)

### macOS
- Requires Xcode
- Dependencies: `../Libraries/local/Qt-*`
- Set `QT` environment variable: `export QT=6.8`

### Linux
- Build dependencies in `../Libraries`
- Set `QT` environment variable if needed

## Key Files

- **`Telegram/build/version`** - Version information
- **`out/`** - Build output directory

## Troubleshooting

### "Libraries not found"
Ensure the repository is in `L:\Telegram\tdesktop`. The build system requires `../win64/Libraries` to exist.

### Build fails with "wrong command prompt"
On Windows, use the correct Visual Studio Native Tools Command Prompt matching your target (x64/x86/ARM64).

### Build fails with PDB or EXE access errors

**âš ï¸ CRITICAL: DO NOT RETRY THE BUILD. STOP AND WAIT FOR USER.**

If the build fails with ANY of these errors:
- `fatal error C1041: cannot open program database`
- `cannot open output file 'Telegram.exe'`
- `LNK1104: cannot open file`
- Any "access denied" or "file in use" error

**STOP IMMEDIATELY.** These errors mean files are locked by a running process (Telegram.exe or debugger).

**What to do:**
1. Do NOT attempt another build - it will fail the same way
2. Do NOT try to delete files - they are locked
3. Do NOT try any workarounds or fixes
4. IMMEDIATELY inform the user:

> "Build failed - files are locked. Please close Telegram.exe (and any debugger) so I can rebuild."

**Then WAIT for user confirmation before attempting any build.**

Retrying builds wastes time and context. The ONLY fix is for the user to close the running process.

## Best Practices

1. **Always use Debug builds** - Release builds are extremely heavy
2. **Don't build Release configuration** - it's too heavy for testing

## Text File Format

- On Windows, keep project text files with CRLF line endings.
- Do not save source, header, build/config, style, or localization files as UTF-8 with BOM. Use UTF-8 without BOM.
- When rewriting project text files for normalization, preserve file content otherwise and do not introduce a BOM.

## Local Storage Serialization

Both app-level (`Core::Settings`) and session-level (`Main::SessionSettings`) use sequential binary serialization via `QDataStream`. Key rules:

- New fields must ALWAYS be appended at the **end** of the stream, never inserted in the middle
- Reading new fields must be guarded with `!stream.atEnd()` and provide a meaningful default/fallback
- Inserting in the middle breaks reading of data saved by older versions (the new read code consumes bytes that belong to subsequent fields)
- For simple flags and values, prefer using the generic KV prefs facility (`writePref<Type>` / `readPref<Type>`) instead of adding to the binary stream -- this avoids serialization ordering issues entirely

---

# Development Guidelines

## Coding Style

**Do NOT write comments in code:**

This is important! Do not write single-line comments that describe what the next line does - they are bloat. Comments are allowed ONLY to describe complex algorithms in detail, when the explanation requires at least 4-5 lines. Self-documenting code with clear variable and function names is preferred.

```cpp
// BAD - don't do this:
// Get the user's name
auto name = user->name();
// Check if premium
if (user->isPremium()) {

// GOOD - no comments needed, code is self-explanatory:
auto name = user->name();
if (user->isPremium()) {

// ACCEPTABLE - complex algorithm explanation (4+ lines):
// The algorithm works by first collecting all visible messages
// in the viewport, then calculating their intersection with
// the clip rectangle. Messages are grouped by date headers,
// and we need to account for sticky headers that may overlap
// with the first message in each group.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Revincx/Yukigram](https://github.com/Revincx/Yukigram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
