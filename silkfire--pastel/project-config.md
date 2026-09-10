---
trigger: always_on
description: A tiny library that colorizes console output by wrapping strings in ANSI escape sequences. It only produces strings; it never writes to the console itself.
---

# Pastel

A tiny library that colorizes console output by wrapping strings in ANSI escape sequences. It only produces strings; it never writes to the console itself.

This file records the things that aren't obvious from reading the code. Everything else, read the source.


## Target frameworks

`net462;net8.0;net9.0`. Every change has to compile on all three, and the code forks heavily on `#if NET8_0_OR_GREATER` / `#if NET9_0_OR_GREATER`.

**The net462 target compiles as C# 7.3.** The SDK defaults non-Core target frameworks to that language version, and there's no `Directory.Build.props` or `global.json` overriding it. So inside an `#else` branch there are no list patterns, no collection expressions (`[]`), no target-typed `new`. Use `Array.Empty<T>()` rather than `[]` in any code shared across the branches. Verify with:

```powershell
dotnet msbuild src\Pastel.csproj -p:TargetFramework=net462 -getProperty:LangVersion
```

This asymmetry has already produced one shipped crash: the pre-.NET 8 branch hand-rolled what the list pattern does for free, and got it wrong.


## MSBuild: don't condition an ItemGroup on $(DefineConstants)

```xml
<!-- Silently never matches -->
<ItemGroup Condition="$(DefineConstants.Contains('NET8_0_OR_GREATER'))">
```

An `ItemGroup` at the project root is evaluated *before* the framework-specific `DefineConstants` are populated, so `$(DefineConstants)` is only `TRACE`/`TRACE;DEBUG` at that point and the check quietly fails for every target framework. Condition on `$(TargetFramework)` instead.

This shipped broken from 2022 to 2026 without anyone noticing, because a failed condition produces no error — the attribute simply never appeared on any published assembly. `AssemblyTests` now guards it.

The trap is hard to reproduce in isolation: in a *single*-target-framework project the same idiom evaluates to `True`, so a minimal repro will mislead you. Check the real thing instead:

```powershell
Select-String "DisableRuntimeMarshalling" "src\obj\Release\net9.0\Pastel.AssemblyInfo.cs"
```


## Colors: the ConsoleColor web mapper

`s_consoleColorWebMapper` (used by `useWebColors: true`) is the **combined CSS3 list**, which is not the same thing as either the X11 palette or the Windows console palette.

Of the four names where the web and X11 definitions conflict, only two exist in `ConsoleColor`, and both resolve to their **web** value:

| | Pastel | X11 |
| --- | --- | --- |
| `Gray` | #808080 | #BEBEBE |
| `Green` | #008000 | #00FF00 (the web's `Lime`) |

Everything else in the enum is a name the two systems agree on (the `Dark*` shades).

Consequences worth knowing before "fixing" anything:

- **`DarkGray` (#A9A9A9) is lighter than `Gray` (#808080).** `DarkGray` descends from X11, `Gray` from the web, and the combined list keeps both. This is intentional and matches `System.Drawing.Color` exactly — verified for all 15 names it also defines.
- These are **not** the Windows console values (console `Gray` is #C0C0C0 legacy / #CCCCCC Campbell; console `DarkGray` is #808080 / #767676).
- The parameter was called `useLegacy` through v7. `useX11` was considered and rejected: it would contradict both conflicting values.

**The mapper is complete and closed.** It's indexed by `ConsoleColor`, a BCL enum with exactly 16 contiguous members, and all 16 have entries. `Maroon` and `Purple` are the other two web/X11 conflicts but are *not* `ConsoleColor` members, so they can't be added — `ConsoleColor.Maroon` doesn't compile. Anything outside the 16 names goes through the `Color` or hexadecimal overloads.


## The reset scanning invariant

`ScanColorFormatStringInsertPositions` finds every place the color format string has to be re-inserted. The rule, for the tail following each `\x1b[0m`:

> re-insert if the tail is non-empty **and** does not itself start with `\x1b[0m`

A trailing *partial* escape sequence (`\x1b`, `\x1b[`, `\x1b[0`) is not a reset, so it counts as an insert position. That's the case the old net462 branch got wrong: it checked for the `\x1b[` prefix and then sliced 2 more characters that weren't necessarily there.

The predicate was verified exhaustively against the .NET 8 list pattern over 19608 tails (alphabet `\x1b [ 0 m 3 8 x`, lengths 0-5) with zero divergence. If you touch it, re-verify rather than reason about it — the semantics are subtle and the tests are the specification.


## Code style

`.editorconfig` covers formatting, encoding and the analyzer severities, but it can't express everything below. Match the surrounding code.

- Block-scoped namespace, `using` directives **inside** it, `System` first.
- Allman braces. Single-statement `if` bodies always get braces.
- `#if`/`#else`/`#endif` at **column 0**, never indented.
- Heavy column alignment: `=` signs line up within a block, array initializers align under the opening construct, and operators lead continuation lines (note the padding after `return`).
- Static field prefixes are inconsistent by any standard rule, so match the neighbouring field *of the same shape*: arrays get `s_`, while dictionaries, bools and `char` consts get `_`. Win32 interop constants are `SCREAMING_SNAKE`.


## Line endings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silkfire/Pastel](https://github.com/silkfire/Pastel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
