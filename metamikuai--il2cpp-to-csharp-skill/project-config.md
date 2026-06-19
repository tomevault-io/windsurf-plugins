---
trigger: always_on
description: Analyze Unity IL2CPP game binaries through IDA Pro MCP and reconstruct plausible C# source from user-provided VAs or function names. Use this for IL2CPP Unity reverse engineering workflows that require string literal resolution, vtable mapping, switch/enum cascade recovery, lambda/closure reconstruction, LINQ recovery, coroutine/async state machines, and IL2CPP runtime helper cleanup. Do not use it for generic decompilation, non-IL2CPP .NET binaries, or exploit development.
---


# IL2CPP to C# Source Restoration

Use IDA Pro MCP to analyze Unity IL2CPP binaries from user-provided VAs or function names and return reconstructed C# source.

**Note: all VAs, addresses, function names, class names, helper addresses, and other concrete identifiers in this skill are examples only. For real work, use the current game's IDA output and do not copy example values or names.**

## Core Principles

**IDA pseudo-C is the trusted primary input for source restoration.** The most common failures are invented string contents, inverted conditions, merged switch branches, and "more reasonable" rewrites that are not in IDA. Match the structure of IDA pseudo-C first; only clean it into natural C# after correctness is established.

If `decompile_function` output is truncated, missing its end, or clearly not the full function body, pause source restoration and ask the user to manually decompile the function in IDA Pro and paste the complete pseudo-C. Do not fill truncated regions from context.

Unless the user explicitly provides assembly and asks for assembly-based recovery, do not use assembly as the main reference for normal method restoration. Assembly may be used only for narrow local checks, such as trivial ICF bodies, field-offset noise, or a single object offset that pseudo-C cannot confirm directly. Confirm field names, signatures, enum values, and `[FieldOffset]` with dnSpy / DummyDll stubs, `*_Fields` structures, and `stringliteral.json`.

## Preparation

Before analyzing a function, confirm that the user has already prepared an analyzed IDA database and working IDA Pro MCP tools. Ideally, the user should also provide a dnSpy-exported C# stub project.

### Il2CppDumper Output

Ask the user to locate the main game assembly and metadata files, for example:

- Windows Unity IL2CPP: `GameAssembly.dll`
- Unity metadata: `global-metadata.dat`

Ask the user to process these files with Il2CppDumper and keep:

- `script.json`: method addresses, type information, and metadata mappings
- `stringliteral.json`: real values for `StringLiteral_N`; the user must provide the local path explicitly
- `DummyDll/`: C# type, field, method signature, and VA comment stubs

### IDA Pro Database

Ask the user to decompile the main assembly in IDA Pro, such as `GameAssembly.dll`, and wait for IDA's initial analysis to finish. After analysis, ask the user to run Il2CppDumper's `ida_with_struct_py3` script on the current IDA database.

After the script has run, if the project is large or the analysis will continue for a long time, suggest that the user back up the IDA database before renaming or doing experimental changes.

### IDA Pro MCP

Ask the user to install and enable the IDA Pro MCP plugin so the agent can:

- query and decompile functions
- view assembly for local checks
- query callees and xrefs
- query globals
- inspect structures and fields

Before real analysis, check that the MCP connection works. If MCP is unavailable, pause and ask the user to fix the plugin connection.

### DummyDll Stub Project

Recommend that the user open Il2CppDumper's `DummyDll/Assembly-CSharp.dll` in dnSpy and export it as a local C# project. The DLL contains only stubs, but it is important for source restoration:

- class, namespace, and method signatures
- field names and `[FieldOffset(...)]`
- method VA / RVA comments
- enum definitions and explicit enum values
- auto-properties, events, generic types, and inheritance

If the user does not provide this project, restoration can continue with IDA and Il2CppDumper outputs, but note any uncertainty around fields, enums, or signatures.

## Basic Workflow

1. The user provides a VA or function name.
2. Use `get_function_by_address(VA)` or `get_function_by_name(name)` to confirm the function and detect compiler folding.
3. Use `decompile_function(VA)` to get pseudo-C and check that it is complete. If it is truncated, ask the user to paste the full pseudo-C from IDA.
4. Resolve string literals, switch cascades, vtable calls, lambdas, closures, and other patterns using the sections below.
5. Remove IL2CPP runtime helper noise.
6. Return reconstructed C# source.

Analyze one function at a time. If the user gives multiple VAs, process them one by one and return each result before continuing.

## String Literal Resolution

See [strings.md](strings.md). For normal C# source restoration, do not search for strings as an entry point; only resolve `StringLiteral_N` values that already appear in the current decompile. Prefer `scripts/lookup_strings.py` against the user-provided `stringliteral.json`. **Never invent string contents from context.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaMikuAI/il2cpp-to-csharp-skill](https://github.com/MetaMikuAI/il2cpp-to-csharp-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
