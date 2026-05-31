---
trigger: always_on
description: Instructions for AI coding agents working in the ZLint repository. ZLint is an
---

# AGENTS.md

Instructions for AI coding agents working in the ZLint repository. ZLint is an
opinionated linter for the Zig programming language, written in Zig. It has its
own semantic analyzer (inspired by [oxc](https://github.com/oxc-project/oxc))
independent from the Zig compiler.

## Output Principles
- All code must be correct and memory safe. No undefined behavior or sloppy work.
- We care heavily about performance. Every cache miss and allocation matters.
- Test everything extensively. Leave nothing unverified.
- Avoid wasting tokens. Don't summarize what you will do before doing it; just do it.
- Ask yourself: "Would a staff engineer do this?" If the answer is no, don't do it.

## Rules
- Do not commit, push, rebase, etc. unless explicitly asked.
- Do not make false or unfounded assumptions. Collect evidence from the user, source code, etc.
- Never write quick hacks. Address the root cause of the problem.

## Guidelines
- Keep changes small and focused.
- Favor editing existing files over creating new ones.
- Check for existing solutions before proposing a new one; avoid reinventing the wheel.
- Follow conventions in [CONTRIBUTING.md](./CONTRIBUTING.md).

## Project Layout

```
.
├── build.zig / build.zig.zon     Build graph + package manifest (authoritative Zig version)
├── Justfile                      Task runner. `just --list` to see everything
├── src/
│   ├── main.zig                  CLI entrypoint
│   ├── root.zig                  Library entrypoint (used by e2e tests + codegen)
│   ├── Semantic.zig, Semantic/   Parser, scopes, symbols, references, modules
│   ├── linter/
│   │   ├── linter.zig            Orchestrates rule execution
│   │   ├── rule.zig              Rule vtable + Meta/Category definitions
│   │   ├── rules/                One file per rule (snake_case) + snapshots/
│   │   ├── config/               Rule config structs (partly codegen)
│   │   ├── fix.zig               Autofix data types + Fixer
│   │   └── tester.zig            RuleTester (pass/fail/fix + snapshots)
│   ├── cli/, reporter/, printer/, visit/, util/
├── tasks/
│   ├── docgen.zig                Generates docs/rules/*.md from doc-comments
│   ├── confgen.zig               Generates rules_config.zig + zlint.schema.json
│   └── new-rule.ts               Bun script that scaffolds a new rule
├── test/                         E2E binary, fixtures, snapshots, repos.json
├── docs/rules/                   Auto-generated per-rule docs
└── apps/{site, vscode-extension} Docs site (Docusaurus) + VS Code extension
```

## Tools
Zig 0.15, `just` for tasks, `bun` for package management and running JS apps, `typos` for spell checking.

## Build, Test, Run

Prefer `just` targets over raw `zig build`. `just ready` is the canonical "is my
branch green?" command — it runs fmt, check, codegen, install, unit tests, and
E2E, then `git status`. Failures have clear error messages; success may be
silent but will exit with code 0. After a clean run, `git diff` must be empty
(codegen output is checked in). Run it before opening a PR.

Scripts running e2e tests (`just e2e`, `just ready`) will fail if test repos are not cloned.
Run `just submodules` to clone them.

See the [Quick Reference](#quick-reference) table below for all common
commands.

When debugging a single-threaded issue, pass `-Dsingle-threaded`:

```sh
just run -Dsingle-threaded -- path/to/file.zig
just e2e -Dsingle-threaded
```

E2E tests compile as a separate `test-e2e` binary that imports `zlint` as a
module (via `src/root.zig`). Run `just submodules` once to clone the repos in
`test/repos.json`, then `just e2e`.

## Code Guidelines

## Conventions

### Constructors and Destructors

1. Constructors that allocate are named `init`.
2. Constructors that do not allocate are named `new`.
3. Destructors are named `deinit`.

### File Naming and Layout

Two kinds of files:

- **Object files** — the whole file is a single `struct` with fields at the top
  level. Use `PascalCase.zig` (e.g. `Semantic.zig`, `LintService.zig`).
  Layout: fields → constants → methods (`init`/`deinit` first, then others) →
  nested types → imports → tests.
- **Namespace files** — re-export types/functions, no top-level struct. Use
  `snake_case.zig` (e.g. `lint.zig`, `span.zig`, `rules.zig`). Avoid declaring
  free functions at the top scope; group by domain struct where possible.
  Layout: imports → public types → public functions → private types/functions →
  tests.

### Other Conventions

- Tests live at the bottom of the file they cover, or in a neighboring
  `test/` directory (e.g. `src/Semantic/test/`, `src/linter/test/`).
- Rule files use `snake_case.zig` (`no_print.zig`), rule _names_ are
  kebab-case (`no-print`).
- Prefer `util.assert(cond, fmt, args)` / `util.debugAssert(...)` over bare
  `std.debug.assert` when a failure message is helpful.
- `util.IS_DEBUG`, `util.RUNTIME_SAFETY`, `util.IS_TEST`, `util.IS_WINDOWS` are
  the standard compile-time flags.
- Memory ownership is explicit: accept an `Allocator`, provide `deinit`,
  document if a type takes ownership or borrows.
- Use `std.log` in library code, never `std.debug.print` (the `no-print` rule
  flags it). Test code is fine.
- E2E tests (`test/test_e2e.zig`) consume zlint via the `src/root.zig` public

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DonIsaac/zlint](https://github.com/DonIsaac/zlint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
