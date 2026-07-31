---
trigger: always_on
description: This file provides guidance to LLM Agents such as Codex, Gemini, Claude Code (claude.ai/code), etc. when working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM Agents such as Codex, Gemini, Claude Code (claude.ai/code), etc. when working with code in this repository.

## CRITICAL REQUIREMENTS

### Test Success
- ALL tests MUST pass for code to be considered complete and working
- Never describe code as "working as expected" if there are ANY failing tests
- Even if specific feature tests pass, failing tests elsewhere indicate broken functionality
- Changes that break existing tests must be fixed before considering implementation complete
- A successful implementation must pass linting, type checking, AND all existing tests

## Project Overview

This is a minimalist, modular Vim configuration designed for sustainability and portability. The configuration follows a philosophy of simplicity and reliability, with conditional loading to prevent breakage when dependencies are missing. It includes `libtestvim`, a hermetic test harness and benchmark toolkit for validating the Vim configuration.

The project has a dual nature:
- **Vim configuration**: `vimrc`, `plugins.vim`, `autoload/`, `settings/`, `ftplugin/`
- **Python package** (`libtestvim`): hermetic Vim harnessing, profiling, and benchmarks under `src/libtestvim/`

## Development Environment

This project uses:
- Python 3.13+ (for `libtestvim`)
- [uv](https://github.com/astral-sh/uv) for dependency management
- [ruff](https://github.com/astral-sh/ruff) for linting and formatting
- [ty](https://github.com/astral-sh/ty) for type checking
- [pytest](https://docs.pytest.org/) for testing
- [just](https://github.com/casey/just) as the task runner
- [hyperfine](https://github.com/sharkdp/hyperfine) for startup benchmarks
- Vim 9.1+ (test target)

## Key Architecture

### Entry Points
- `vimrc`: Main configuration entry that orchestrates all other components (includes inline vim-plug bootstrap)
- `plugins.vim`: Defines all plugins with conditional loading based on executable availability

### Directory Structure
- `autoload/`: Core functions (settings.vim, lib.vim)
- `settings/`: Modular configuration files loaded by autoload/settings.vim
- `ftplugin/`: File-type specific settings
- `plugged/`: Plugin installation directory (gitignored)
- `coc-settings.json`: Language server configurations for CoC.nvim
- `src/libtestvim/`: Python package for hermetic Vim harnessing, profiling, and benchmarks
- `tests/vim/`: Native Vimscript test suites (core/ and integration/)
- `tests/pytest/`: Python test files driven by pytest
- `justfile`: Task runner recipes for setup, test, lint, and benchmark workflows

### Plugin Management
Uses vim-plug with automatic installation. Plugins are conditionally loaded based on executable availability (e.g., rust.vim only loads if `cargo` exists).

## Common Commands

### Plugin Management
```vim
:PlugInstall    " Install missing plugins
:PlugUpdate     " Update all plugins
:PlugClean      " Remove unused plugins
```

### Setup

Create or update the uv-managed test environment:

```bash
just sync
```

Install vim-plug plugins into the hermetic plugged/ directory:

```bash
just plug-install
```

Set up Neovim compatibility symlinks:

```bash
just nvim
```

Link contrib snippets into settings directories:

```bash
just complete
```

### Linting

Run ruff check and format verification:

```bash
just lint
```

Run ty type checking on src/:

```bash
just typecheck
```

Lint Vimscript files with vint:

```bash
just vint
```

### Testing

Run the fast hermetic suites (no tmux or benchmarks):

```bash
just test
```

Run native Vimscript core suites only:

```bash
just test-core
```

Run plugin and executable integration suites:

```bash
just test-integration
```

Run the libtmux-backed terminal smoke test:

```bash
just test-tmux
```

Run the full pytest matrix:

```bash
just test-all
```

### Benchmarking

Generate startup benchmark artifacts:

```bash
just benchmark
```

Compare the current branch against the remote default:

```bash
just compare
```

Compare the current branch against multiple refs:

```bash
just compare-multi
```

### Testing Changes
When modifying configuration:
1. Source the file: `:source %` or `:source ~/.vim/vimrc`
2. Check for errors: `:messages`
3. Test conditional loading by checking executable availability

## Development Workflow

Follow this workflow for code changes:

1. **Lint**: `just lint`
2. **Typecheck**: `just typecheck`
3. **Test**: `just test`
4. **Full matrix**: `just test-all`

## Testing Guidelines

### Dual Test Stack

This project has two test layers:

1. **Native Vimscript suites** (`tests/vim/core/`, `tests/vim/integration/`)
   - Use `Test_*` function naming
   - Use `SetUp()` / `TearDown()` for per-test fixtures
   - Assert with `v:errors` (`call assert_equal(...)`, `call assert_true(...)`)
   - Run in Vim's Ex mode via the test harness

2. **Python tests** (`tests/pytest/`)
   - Driven by pytest through `libtestvim`
   - Use `@pytest.mark.core`, `.integration`, `.tmux`, `.benchmark` markers
   - Available fixtures: `repo_root`, `artifact_root`, `vim_harness`, `vim_suite_runner`, `benchmark_spec`

### Testing Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tony/vim-config-framework](https://github.com/tony/vim-config-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
