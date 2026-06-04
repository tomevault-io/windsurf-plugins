---
trigger: always_on
description: This document contains repo-wide architectural guidelines, development workflows, and coding conventions for maintaining the pure Vim Script JVM implementation.
---

# `javim` Project Instructions & Conventions

This document contains repo-wide architectural guidelines, development workflows, and coding conventions for maintaining the pure Vim Script JVM implementation.

## Core Architectural Design

The JVM is designed as a modular interpreter running entirely inside Vim 8 (excluding Neovim support).

1. **Bytecode Decoder (`autoload/javim/classfile.vim`):** Reads Java bytecode `.class` files as a binary `Blob` using Vim 8 `readfile(..., 'B')` and parses constant pools, interfaces, fields, methods, and the `Code` attribute.
2. **Interpreter & Predecoder (`autoload/javim/interpreter.vim`):** Manages stack frames, JVM heap allocation, static fields, and method invocations. To ensure maximum execution speed, methods are **predecoded** once prior to execution. The original bytecode is converted into a non-destructive `instructions` stream object array, with jump targets pre-resolved via a `pc_to_ip` map. It also dynamically injects Superinstructions using an internal extended opcode space (`>= 0x10000`).
3. **Instruction Set (`autoload/javim/instructions.vim`):** Implements standard JVMS opcodes (`0x00` to `0xff`). The execution loop operates over the predecoded instruction index (`ip`) rather than sequential byte-reading.
4. **Vim-Native Standard Library (`autoload/javim/rt/`):** Bypasses the need for real compiled `rt.jar` class files. Standard Java runtime classes (like `java.lang.Object`, `java.lang.String`, `java.lang.System`, and `java.io.PrintStream`) are implemented directly in Vim Script as built-in runtime support, exposing pre-configured `ClassDict` structures with native Vim callback mappings.
5. **Execution Entry Point (`autoload/javim.vim`):** Exposes the public `javim#run(...)` API, which parses runtime parameters (such as classpath override `-cp`), configures an isolated VM state dictionary, maps execution arguments to native JVM String array references, and executes the target class `main` method. The `:JavimRun` command defined in `plugin/javim.vim` acts as a lightweight wrapper delegating directly to this function.

---

## Coding Conventions & Best Practices

All new and modified Vim Script files in this repository must strictly adhere to the following standards:

### 1. File Header and Compatibility Guard
Every single `.vim` file under `plugin/`, `autoload/`, and `test/` must place its file path comment at the absolute top (line 1), followed immediately by the standard `'cpoptions'` (`cpo`) save and restore block. This ensures a clean, unified, and professional file structure while preventing line continuation (`\`) syntax errors:
```vim
" autoload/javim/example.vim

let s:save_cpo = &cpo
set cpo&vim

" ... script logic ...

let &cpo = s:save_cpo
unlet s:save_cpo
```

### 2. Script-Local Scope Prefix
Never use underscores for script-local function names (such as `s_my_helper()`). Always use the proper colon-prefixed local script syntax:
```vim
" Correct (script-local function prefix)
function! s:my_helper() abort
  " ...
endfunction

" Incorrect (throws E128 syntax error)
function! s_my_helper() abort
  " ...
endfunction
```

### 3. Vim 8 Exclusivity
Do not employ Neovim-specific namespaces, APIs, or Lua wrappers. Always program to Vim 8+ specifications (Blob types, standard test assertions like `assert_equal()`).

---

## Development Workflows

### 1. Launching for Debugging & Tab-Completion
Use the bundled debug script to launch Vim with `-u NONE` (no vimrc), forcing `nocompatible` mode with our plugin added to the `'runtimepath'`:
```bash
./vim-debug.sh
```
* Once inside Vim, type `:Ja<Tab>` to auto-complete to `:JavimRun`.
* Execute classfiles by passing their package path: `:JavimRun test.classes.HelloWorld`.

### 2. Sourcing and Compiling Java Test Targets
If you write new test targets inside `test/classes/`:
```bash
javac -d . test/classes/MyNewTest.java
```
Always use `-d .` to output classfiles in packages mapped exactly to their directory structure, allowing the JVM ClassLoader to resolve them.

### 3. Running Automated Tests
The headless testing framework runs via:
```bash
vim -u NONE -S test/run_tests.vim
```
This script appends `.` (repository root) to the `'runtimepath'` to resolve autoload dependencies and runs all test suites non-interactively, returning an exit code of `0` on success, or logging test failures to standard output/logs and exiting with `cquit!` (non-zero) on error.

---

## Documentation & Specification Maintenance Policy

To maintain a clear and accurate understanding of the interpreter's compliance status, we maintain three hand-managed specification and compatibility documents under the `docs/` folder:

1. **`docs/JVMS.md`**: Tracks compatibility against the *Java Virtual Machine Specification, Java SE 8 Edition (JVMS SE 8)*, covering binary class formats, constant pool tags, attributes, descriptors, runtime areas, stacks, and method invocation boundaries.
2. **`docs/OPCODES.md`**: A manually updated listing of the entire 1-byte opcode space (`0x00` to `0xff`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kamichidu/vim-javim](https://github.com/kamichidu/vim-javim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
