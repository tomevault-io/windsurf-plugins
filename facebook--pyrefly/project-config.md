---
trigger: always_on
description: Pyrefly is a fast language server and type checker for Python.
---

# Guidance for Project Agents

## Project Overview

Pyrefly is a fast language server and type checker for Python.

Architecture:

- Written in Rust using Buck (mostly for meta developers) and cargo (mostly for
  open-source developers)
- Minimal dependencies, framework-free

As described in the README, our architecture follows 3 phases:

- figuring out exports
- making bindings
- solving the bindings

Here's an overview of some important directories:

- pyrefly/lib/alt - Solving step
- pyrefly/lib/binding - Binding step
- pyrefly/lib/commands - CLI
- pyrefly/lib/config - Config file format & config options
- pyrefly/lib/error - How we collect and emit errors
- pyrefly/lib/export - Exports step
- pyrefly/lib/module - Import resolution/module finding logic
- pyrefly/lib/solver - Solving type variables and checking if a type is
  assignable to another type
- pyrefly/lib/state - Internal state for the language server
- pyrefly/lib/test - Integration tests for the typechecker
- pyrefly/lib/test/lsp - Integration tests for the language server
- pyrefly/lib/test/lsp/lsp_interaction - Heavyweight integration tests for the
  language server (only add tests here if it's impossible to add them in the
  lightweight tests)
- crates/pyrefly_types/src - Our internal representation for Python types
- conformance - Typing conformance tests pulled from python/typing. Don't edit
  these manually. Instead, run test.py and include any generated changes with
  your PR.
- test - Markdown end-to-end tests for our IDE features
- website - Source code for pyrefly.org
- lsp - vscode extension written in typescript

## Codebase style and guidelines

Coding style: All code must be clean, documented and minimal. That means:

- Keep It Simple Stupid (KISS) by reducing the "Concept Count". That means,
  strive for fewer functions or methods, fewer helpers. If a helper is only
  called by a single callsite, then prefer to inline it into the caller.
- At the same time, Don't Repeat Yourself (DRY)
- There is a tension between KISS and DRY. If you find yourself in a situation
  where you're forced to make a helper method just to avoid repeating yourself,
  the best solution is to look for a way to avoid even having to do the
  complicated work at all.
- If some code looks heavyweight, perhaps with lots of conditionals, then think
  harder for a more elegant way of achieving it.
- Code should have comments and functions should have docstrings. The best
  comments are ones that introduce invariants, or prove that invariants are
  being upheld, or indicate which invariants the code relies upon. Don't duplicate comments, or write unnecessary comments for code that is obvious.
- **Unreachable states must panic, not silently degrade.** Do not use defensive
  programming to handle states that should be impossible. If a match arm, Option,
  or Result should never occur given the surrounding invariants, use
  `unreachable!("explanation")` or `.expect("explanation")` — never
  `_ => default`, `.unwrap_or_default()`, or silent fallbacks. A type checker
  that silently produces wrong results is far worse than one that crashes with a
  clear message. Silent fallbacks hide bugs and confuse maintainers by making
  unreachable states look reachable.
- Check for existing helpers in the `pyrefly_types` crate before manually
  creating or destructuring a `Type`.
- Minimize the number of places `Expr` nodes are passed around and the number of
  times they are parsed. Generally, this means extracting semantic information
  as early as possible.
- **Imports:** Always add `use` imports at the top of the file rather than using
  inline qualified paths (e.g., write `use crate::foo::Bar;` and then `Bar`,
  not `crate::foo::Bar` inline). The only exception is when there is a name
  collision between two imports, which is rare.

## Commit Messages

Do not write a laundry list of implementation changes. Focus on:

- **Why**: what problem or design gap motivated the change
- **What** (high level): the approach or solution, not individual file edits
- **Why it works**: how the code changes realize the solution

A reader should be able to understand the intent and rationale from the commit message, without following all the code changes in details.

## Development environments

There are three possible development environments:

1. **External/GitHub checkout**: Only `cargo` is available. The `buck` and `arc`
   commands do not exist.
2. **Internal on-demand**: Only `buck` is available. The `cargo` command may not
   be configured.
3. **Internal devserver with cargo**: Both `buck` and `cargo` are available.

**How to detect the environment:** Check for the presence of a `BUCK` file in
the project root. BUCK files are not exported to GitHub, so:
- If `BUCK` exists → internal checkout, `buck` and `arc` are available
- If `BUCK` does not exist → GitHub checkout, only `cargo` works

### Source control

**Do not assume git.** This repo may be either a Git checkout or a Sapling
(Mercurial-based) checkout. Before running any source-control commands, detect
which VCS is in use:

- If `.git` exists at the repo root → Git. Use `git` commands.
- If `.sl` exists at the repo root → Sapling. Use `sl` commands (`sl status`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/pyrefly](https://github.com/facebook/pyrefly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
