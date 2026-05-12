---
trigger: always_on
description: **Purpose:** This document helps AI assistants understand and work with the Rip language compiler and its ecosystem of packages.
---

# AI Agent Guide for Rip

**Purpose:** This document helps AI assistants understand and work with the Rip language compiler and its ecosystem of packages.

**What is Rip:** An elegant reactive language that compiles to modern JavaScript (ES2022), featuring zero dependencies, self-hosting capability, and built-in reactivity primitives.

Detailed subsystem notes live in nested `AGENTS.md` files in the relevant directories (`src/`, `docs/`, `packages/ui/`, `packages/vscode/`, `packages/stamp/`, `test/types/`).

---

## Quick Start

### Essential Commands

```bash
# Debug any code
echo 'your code' | ./bin/rip -t  # Tokens (lexer)
echo 'your code' | ./bin/rip -s  # S-expressions (parser)
echo 'your code' | ./bin/rip -c  # JavaScript (codegen)

# Run tests
bun run test
bun test/runner.js test/rip/FILE.rip

# Rebuild parser (after grammar changes)
bun run parser

# Build browser bundle
bun run build

# Serve an app (watches *.rip, HTTPS, mDNS)
rip server

# Interactive REPL (toggle .tokens, .sexp, .js modes)
./bin/rip
```

### File Editing Rules

| File                      | Can Edit? | Notes                                                  |
| ------------------------- | --------- | ------------------------------------------------------ |
| `src/compiler.js`         | Yes       | Code emitter (`CodeEmitter`); main compiler work       |
| `src/lexer.js`            | Yes       | Lexer and rewriter                                     |
| `src/types.js`            | Yes       | Type system sidecar                                    |
| `src/components.js`       | Yes       | Component system sidecar                               |
| `src/schema/`    | Yes       | Schema feature subdirectory (`schema` keyword) — entry `schema.js`, runtime fragments, loaders, `dts.js`. Imported from sibling modules as `./schema/schema.js`. |
| `src/grammar/grammar.rip` | Carefully | Run `bun run parser` after changes                     |
| `src/parser.js`           | Never     | Generated file                                         |
| `src/sourcemaps.js`       | Yes       | Source map generator                                   |
| `src/browser.js`          | Yes       | Browser entry point                                    |
| `rip-loader.js`           | Yes       | Bun plugin for `.rip` compilation and import rewriting |
| `src/grammar/solar.rip`   | Never     | Given parser generator                                 |
| `test/rip/*.rip`          | Yes       | Test files                                             |

### Critical Rules

- **Never guess what code does — verify it.** Before claiming something is unnecessary, redundant, or works a certain way, read the source. This applies to Rip internals, packages, and external dependencies alike. If you can't verify, say so.
- **Never edit `src/parser.js`** — it is generated
- **Never edit `src/grammar/solar.rip`** — it is given
- **Never commit without running tests** — `bun run test` must pass
- **Never add dependencies** — zero dependencies is a core principle
- **Never read or execute scripts directly** — use `bun run <name>`
- **Never write `x ? y` in Rip** — binary existential was removed; use `x ?? y` or full ternary `x ? y : z`
- **Never write `await fn(args)` in `.rip` source when `fn!` will do** — the dammit operator is the idiomatic form. `fetch! url` compiles to `await fetch(url)`; `User.find! 1` to `await User.find(1)`; `user.save!` to `await user.save()`. Reserve raw `await` for JS interop in `.js` files, tests that document the await→!  equivalence, and the rare cases where `fn!` is ambiguous with a dammit-returning expression.
- Run `bun run parser` after grammar changes
- Run `bun run build` after codegen, `components.js`, `browser.js`, or `app.rip` changes
- Run `bun run build:schema-runtime` after editing any `src/schema/runtime-*.js` fragment (CI's `test:schema-fresh` fails on staleness)
- Run `bun run bump` for the standard release flow

## Compilation Pipeline

```text
Rip Source -> Lexer -> emitTypes -> Parser -> S-Expressions -> CodeEmitter -> JavaScript
                       (types.js)           (arrays + .loc)                  + source map
                          ↓
                       file.d.ts (when types: "emit")
```

**Key insight:** S-expressions are simple arrays like `["=", "x", 42]`, not large AST objects.

Detailed compiler, lexer, and component internals are in `src/AGENTS.md`.

---

## Common Tasks

### Fix a Bug in Codegen

```bash
echo 'failing code' | ./bin/rip -s
rg "GENERATORS" src/compiler.js
bun run test
```

### Add a Grammar Rule

```bash
# Edit src/grammar/grammar.rip
bun run parser
# Add codegen in src/compiler.js if needed
bun run test
```

## Testing

Test helpers:

```coffee
test "name", "x = 42; x", 42
code "name", "x + y", "(x + y)"
fail "name", "invalid syntax"
```

Test files live in `test/rip/`.

Component testing notes are in `src/AGENTS.md`. Type-system and audit guidance is in `packages/vscode/AGENTS.md` and `test/types/AGENTS.md`.

---

## Packages

The `packages/` directory contains optional packages written in Rip, with zero dependencies, running on Bun.

### @rip-lang/server


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shreeve/rip-lang](https://github.com/shreeve/rip-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
