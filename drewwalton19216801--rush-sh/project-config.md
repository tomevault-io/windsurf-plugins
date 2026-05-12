---
trigger: always_on
description: **Rush** is a comprehensive POSIX sh-compatible shell implementation written in Rust, currently at version 0.8.0. The project aims to provide a fully compliant POSIX shell while leveraging Rust's type safety, performance, and memory management capabilities.
---

# Rush Shell - AI Agent Guide

## Project Overview

**Rush** is a comprehensive POSIX sh-compatible shell implementation written in Rust, currently at version 0.8.0. The project aims to provide a fully compliant POSIX shell while leveraging Rust's type safety, performance, and memory management capabilities.

### Project Goals

- **POSIX Compliance**: Achieve 100% compliance with IEEE Std 1003.1-2008 (POSIX sh)
- **Performance**: Leverage Rust's zero-cost abstractions for optimal execution speed
- **Reliability**: Extensive test coverage and robust error handling
- **Maintainability**: Clean, modular architecture with comprehensive documentation
- **Safety**: Memory safety and thread safety through Rust's ownership system

### Current Status

- **Compliance Level**: ~96% POSIX compliant
- **Test Coverage**: 499+ test functions across all components
- **Built-in Commands**: 33 implemented commands
- **Core Features**: Full variable expansion, arithmetic evaluation, control structures, functions with return, shell options, job control
- **Architecture**: Modular design with separate lexer, parser, executor, and expansion engines

### Recently Implemented Features

- **Job Control**: Complete background job management with comprehensive jobspec support
  - Background execution with `&` operator
  - Job listing with `jobs` builtin
  - Foreground control with `fg` builtin
  - Background control with `bg` builtin
  - Job termination with `kill` builtin
  - Wait for jobs with `wait` builtin
  - **$! Special Variable**: PID of last background process
  - **Smart Jobspec Matching**: Prefix and contains patterns skip completed jobs
- **Times Builtin**: POSIX-compliant `times` command displaying accumulated user and system CPU times for the shell and child processes, with proper time formatting (XmY.ZZs) and 7 comprehensive test cases
- **PS4 Variable Expansion**: Full variable expansion support in PS4 prompt for xtrace output (`set -x`), including special variables like `$LINENO` and support for both `$VAR` and `${VAR}` brace syntax
- **${VAR} Brace Syntax**: Complete support for brace syntax in variable expansion for all variable types, including special variables like `$LINENO`, `$?`, `$$`, etc.
- **Set Builtin**: POSIX-compliant `set` command with comprehensive shell option management (errexit, nounset, xtrace, verbose, noexec, noglob, noclobber, allexport), positional parameter control, named options (-o/+o), and display modes - 86+ test cases covering all functionality
- **Loop Control Builtins**: POSIX-compliant `break` and `continue` commands with support for nested loops via optional [n] argument, working with for/while/until loops, and 29 comprehensive test cases
- **Subshell Support**: Full POSIX-compliant subshells with state isolation, exit code propagation, trap inheritance, depth limit protection (max 100 levels), and 60+ test cases
- **File Descriptor Operations**: Complete FD table management, duplication (N>&M, N<&M), closing (N>&-, N<&-), read/write (N<>), with 30+ test cases
- **Here-documents**: Full implementation of `<<` and `<<<` (here-strings) with proper expansion handling
- **Enhanced Trap System**: Signal normalization, multiple handlers, trap display/reset, signal queue with overflow protection

## Architecture Overview

### Core Components

```text
src/
├── main.rs              # Entry point and REPL loop
├── executor/            # Command execution (modular)
│   ├── mod.rs           # Main execution engine
│   ├── expansion.rs     # Variable and wildcard expansion
│   ├── redirection.rs   # I/O redirection handling
│   ├── command.rs       # Single command and pipeline execution
│   ├── subshell.rs      # Subshell and compound commands
│   ├── async_exec.rs    # Asynchronous job execution
│   └── tests/           # Focused test modules
│       ├── mod.rs
│       ├── execution_tests.rs
│       ├── expansion_tests.rs
│       ├── redirection_tests.rs
│       ├── command_tests.rs
│       ├── subshell_tests.rs
│       └── async_tests.rs
├── parser/              # AST construction (modular)
│   ├── mod.rs           # Main parsing logic
│   ├── ast.rs           # AST type definitions
│   ├── control_flow.rs  # Control structure parsers
│   └── tests/           # Focused test modules
│       ├── mod.rs
│       ├── basic_tests.rs
│       ├── control_flow_tests.rs
│       ├── compound_tests.rs
│       ├── pipeline_tests.rs
│       ├── operator_tests.rs
│       └── redirection_tests.rs
├── lexer/               # Tokenization (modular)
│   ├── mod.rs           # Main lexing logic
│   ├── token.rs         # Token type definitions
│   └── tests/           # Focused test modules
│       ├── mod.rs
│       ├── basic_tests.rs
│       ├── alias_tests.rs
│       ├── quote_tests.rs
│       ├── expansion_tests.rs
│       ├── redirection_tests.rs
│       ├── tilde_tests.rs
│       └── edge_case_tests.rs
├── state/               # Shell state (modular)
│   ├── mod.rs           # Core state management
│   ├── fd_table.rs      # File descriptor table
│   ├── options.rs       # Shell options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drewwalton19216801/rush-sh](https://github.com/drewwalton19216801/rush-sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
