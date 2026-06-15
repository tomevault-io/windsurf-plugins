---
trigger: always_on
description: Use for Nim symbol navigation, diagnostics, and type resolution. MANDATORY: Use specialized MCP tools (nimFindSymbols, nimFindReferences, nimListSymbols, nimCheckFile, nimCheckProject, nimFindTypeDefinition) first; fall back to grep only on error or user confirmation.
---


# Nim MCP Tools

## Core Mandate

AI agents MUST prefer specialized MCP tools over general-purpose instruments (grep, shell commands, `read_file` with manual parsing) for all Nim symbol-related and Nim diagnostics tasks. This is critical for:

1. **Token Efficiency**: MCP tools return structured, relevant data, avoiding large file reads or noisy grep outputs.
2. **Precision**: These tools understand Nim semantics (scopes, imports, overloads) which string-based search cannot.

## Activation Rule

If the user asks to find, rename, remove, audit, update, standardize naming, or fix issues/project of a Nim symbol or its usages, this skill MUST be used first and the workflow must start with the Nim MCP symbol tools.

If the user asks to check a single Nim file for errors, warnings, hints, diagnostics, issues, problems, or compiler feedback, this skill MUST be used first and the workflow must start with `nimCheckFile`.

If the user asks to check a Nim project for errors, warnings, hints, diagnostics, issues, problems, compiler feedback, or to "fix" it, this skill MUST be used first and the workflow must start with `nimCheckProject`.

If the user asks to determine where a symbol comes from, what type it is, what its fields are (including private ones), or resolve a symbol's type definition, this skill MUST be used first and the workflow must start with `nimFindTypeDefinition`.

This applies to requests phrased as:

- "find all usages/references of `Foo`"
- "remove all definitions of and references to `Foo`"
- "rename `Foo` everywhere"
- "standardize naming for `Foo`"
- "fix casing for all variables"
- "where is `Foo` defined?"
- "list the symbols in this Nim file/module"
- "check this file/module for errors"
- "show diagnostics for `foo.nim`"
- "check this project/workspace/repository/package for errors"
- "fix issues in this project"
- "fix the repo"
- "find Nim diagnostics in the current codebase"
- "show warnings and hints for this repo"
- "scan the current module tree for Nim issues"
- "what is the type of this symbol?"
- "what module/package does this type come from?"
- "what fields does this type have?"
- "is this a type alias or a concrete type?"
- "where is this type defined?"

Treat user wording such as **project**, **workspace**, **repository**, **repo**, **package**, **codebase**, **checkout**, and **module tree** as referring to the current Nim project context when they are asking for project-wide diagnostics.

Treat user wording such as **file**, **module** (when a concrete Nim file is identified), **source file**, and explicit `*.nim` paths as referring to single-file diagnostics when they are asking for diagnostics for one file.

Do **not** pair `nimFindSymbols`, `nimCheckFile`, or `nimCheckProject` with `grep`, `ripgrep`, or shell search "just to double-check". If the task is about a Nim symbol or Nim diagnostics, MCP tools own the search unless they have already failed.

## User Terminology vs MCP `kind`

Users may ask for symbol categories using looser or non-strict terminology. AI agents MUST map that wording to the exact Nim MCP `kind` values before filtering results from `nimListSymbols(...)` or `nimFindSymbols(...)`.

The MCP server returns Nim-oriented kind names derived from nimsuggest symbol kinds with the leading `sk` removed, such as `Const`, `EnumField`, `Field`, `Iterator`, `Converter`, `Let`, `Macro`, `Method`, `Proc`, `Template`, `Type`, `Var`, and `Func`.

Use these terminology mappings when interpreting user requests:

- **function** / **functions**: usually match `Func` and `Proc`
- **pure function** / **pure functions**: match `Func`
- **callable** / **routine**: may include `Func`, `Proc`, `Method`, `Iterator`, `Converter`, `Macro`, and `Template`
- **class** / **classes**: match `Type`
- **variable** / **variables**: match `Var` and `Let`
- **property** / **properties**: match `Field`
- **enum member** / **enum members**: match `EnumField`
- **constant** / **constants**: match `Const`

Interpret user wording semantically, not literally. For example:

- "list all classes in this module" -> filter for `kind == Type`
- "list all variables" -> filter for `kind in {Var, Let}`
- "list all functions" -> filter for `kind in {Func, Proc}` unless the surrounding context clearly asks for all callables
- "list all pure functions" -> filter for `kind == Func`

## When to Use

- **Finding References**: To rename a symbol, update a signature, or find usages.
- **Symbol Discovery**: To find where a symbol is defined by name.
- **Type Resolution**: To determine where a local symbol comes from (which module), what its type is, and what fields it has (including private ones). Uses `nimFindTypeDefinition(path, line, column)` on the symbol's cursor position.
- **Naming Standardization**: To fix casing or follow style guides (e.g., standardizing to `camelCase`) across the project.
- **Fixing Project Issues**: To iteratively find and resolve all diagnostics in the project.
- **File Analysis**: To get an overview of all symbols in a file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nim-lang/langserver](https://github.com/nim-lang/langserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
