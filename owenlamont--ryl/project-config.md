---
trigger: always_on
description: Guidance on how to navigate and modify this codebase.
---

# Coding Agent Instructions

Guidance on how to navigate and modify this codebase.

## What This Tool Does

ryl is a CLI tool for linting yaml files

## Project Structure

- **/src/** – All application code lives here.
- **/tests/** – Unit and integration tests.
- **pyproject.toml** - Package configuration
- **prek.toml** - Prek managed linters and some configuration

## Coding Standards

- Code maintainability is the top priority - ideally a new agent can be onboarded onto
  using this repo and able to get all the necessary context from the documentation and
  code with no surprising behaviour or pitfalls (this is the pit of success principle -
  the most likely way to do something is also the correct way).
- In relation to maintainability / readability keep the code as succinct as practical.
  Every line of code has a maintenance and read time cost (so try to keep code readable
  with good naming of files, functions, structures, and variable instead of using
  comments). Remember every new conditional added has a significant testing burden as it
  will likely require a new test to be added and maintained. We want to keep code bloat
  to a minimum and the best refactors generally are those that remove lines of code
  while maintaining functionality.
- Comments should only be used to explain unavoidable code smells (arising from third
  party crate use), or the reason for temporary dependency version pinning (e.g.
  linking an unresolved GitHub issues) or lastly explaining opaque code or non-obvious
  trade-offs or workarounds.
- Leverage the provided linters and formatters to fix code, configuration, and
  documentation often - it's much cheaper to have the linters and formatters auto fix
  issues than correcting them yourself. Only correct what the linters and formatters
  can't automatically fix.
- No unsafe Rust code. Do not introduce `unsafe` in application code or tests. If a
  change appears to require `unsafe`, propose an alternative design that keeps code
  safe. The crate is built with `#![forbid(unsafe_code)]` and tests should follow the
  same principle.
- Remember the linter/formatter prek won't scan any new modules until they are added to
  git so don't forget to git add any new modules you create before running prek.
- Use the most modern Rust idioms and syntax allowed by the Rust version (currently this
  is Rust 1.93.1).
- Keep `clippy.toml` `msrv` in sync with `rust-toolchain.toml` whenever the Rust
  toolchain channel is changed.
- Don't rely on your memory of libraries and APIs. All external dependencies evolve fast
  so ensure current documentation and/or repo is consulted when working with third party
  dependencies.
- When mirroring yamllint behaviour, spot-check tricky inputs with the ryl CLI so
  our diagnostics and message text match (e.g., mixed newline styles or config keys of
  type int/bool/null/tagged scalar).
- Keep YAML configuration strictly aligned with functionality that yamllint currently
  supports. Put any ryl-only settings, experimental rule options, or ahead-of-upstream
  behaviour in TOML configuration so future yamllint additions cannot clash with
  existing YAML semantics.

## Code Change Requirements

- Whenever any files are edited ensure all prek linters pass (run:
  `prek run --all-files`).
- `prek` already runs the key tooling (e.g., trim/fix whitespace, `cargo fmt`,
  `cargo clippy --fix`, `cargo clippy`, `rumdl` for Markdown/docs, etc.), so skip
  invoking those individually. Re-run `prek run --all-files` until the auto-fixes
  stabilise and a full pass succeeds without modifying files before running coverage.
- Whenever source files are edited ensure the full test suite passes (run
  `./scripts/coverage-missing.sh` (Unix) or
  `pwsh ./scripts/coverage-missing.ps1` (Windows) to regenerate coverage; it reports
  uncovered ranges and confirms when coverage is complete)
- After lint, tests, and coverage are green, review code size changes with
  `uv run scripts/source_size.py --compare-to <branch-or-ref>` (typically the branch
  point or `HEAD`). If the size increase looks large relative to the added
  functionality, look for opportunities to make the implementation DRYer, reuse shared
  helpers, or simplify it before committing.
- For any behaviour or feature changes ensure all documentation is updated
  appropriately.

## Development Environment / Terminal

- This repo runs on Mac, Linux, and Windows. Don't make assumptions about the shell
  you're running on without checking first (it could be a Posix shell like Bash or
  Windows Powershell).
- `prek`, `rg`, `rumdl`, `typos`, `yamllint`, and `zizmor` should be installed as global
  tools (if they don't appear to be installed raise that with the user).
- `gh` will be available in most, but not all environments to inspect GitHub.
- When reviewing PR feedback, prefer `gh pr view <number> --json comments,reviews` for
  summary threads and `gh api repos/<owner>/<repo>/pulls/<number>/comments` when you
  need inline review details without guesswork. Avoid flags that the GitHub CLI does not
  support (e.g., `--review-comments`).
- Linters and tests may write outside the workspace (e.g., `~/.cache/prek`). If
  sandboxed, request permission escalation when running `prek`, `cargo test`,
  or coverage commands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owenlamont/ryl](https://github.com/owenlamont/ryl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
