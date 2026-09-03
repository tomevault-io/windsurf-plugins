---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Bun plugin that makes `.php` files importable: `import { greet } from "./hello.php"` returns an async JS
function that calls into a PHP 8.5 interpreter running in WebAssembly (php-wasm). No PHP binary involved.
Published as the `bun-php` package; `src/` ships as-is (no build step — `exports` points straight at `.ts`).

## Commands

```bash
bun install
bun test                          # all tests
bun test test/parse.test.ts       # one file
bun test -t "variadic"            # one test by name
bun run test:versions             # the cross-version compatibility suite (installed builds only)
bun run php-builds:install        # install every @php-wasm/node-8-* build (~370 MB), then
bun run php-builds:install 8.1    #   just one; both leave package.json and bun.lock untouched
bun run example                   # example/index.ts against example/hello.php
bun run demos                     # demos/index.ts against real Composer packages
bun run typecheck                 # bunx tsc -p tsconfig.json (noEmit)
bun run lint                      # oxlint (correctness rules)
bun run fmt                       # oxfmt, write in place
bun run fmt:check                 # oxfmt --check (what CI runs)

composer install --working-dir=demos   # required before demos/ runs or its tests
bun run demos:vendor:pack         # rebuild demos/vendor.zip (needs Composer; commit the result)
bun run demos:vendor:unpack       # unzip demos/vendor.zip into demos/vendor (what CI runs)
```

There is a lint step (oxlint + oxfmt) but no build step.

CI (`.github/workflows/test.yml`) runs on PRs and `main`: typecheck, lint, `fmt:check`, then unpacks
`demos/vendor.zip` and runs the tests on ubuntu + macos. The demo Composer deps are committed as
`demos/vendor.zip` (built by `demos:vendor:pack`) so CI needs neither Composer nor a system PHP —
regenerate and recommit that zip whenever `demos/composer.lock` changes. Releases go out via
`.github/workflows/release.yml`: release-please cuts the tag, then npm publishes over OIDC (no token).
`CHANGELOG.md` is in `.oxfmtrc.json`'s `ignorePatterns` because release-please regenerates it in its own
Markdown style — reformatting it only wins until the next release rewrites it, and `fmt:check` fails on
every release PR in the meantime.

## Architecture

The pipeline runs once per imported `.php` file, at load time:

```
onLoad (plugin.ts)
  → parsePhp (parse.ts)      PHP source  → PhpModuleMeta
  → generateModule (codegen.ts)  meta    → JS module source, returned to Bun
  → generateDts (dts.ts)         meta    → sidecar <file>.php.d.ts written next to the source
  → resolveProject (project.ts)  path    → { root to mount, autoload to require }
```

The generated module imports `createPhpModule` from `runtime.ts` and exports one async wrapper per PHP
function. At call time: `runtime.ts` → `marshal.ts` (build the PHP script, decode the result) →
`interpreter.ts` → `php-runtime.ts` (the interpreter).

| File                                           | Responsibility                                                                                       |
| ---------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `src/plugin.ts`                                | `onLoad` hook, option defaults, sidecar writing                                                      |
| `src/register.ts`                              | Side-effecting `plugin(phpPlugin())` for `preload`                                                   |
| `src/parse.ts`                                 | php-parser AST → `PhpModuleMeta` (functions, constants, skip notes)                                  |
| `src/php-types.ts`                             | PHP type declarations → TypeScript type expressions                                                  |
| `src/codegen.ts`                               | Emits the JS module; owns the aliasing helpers (`bindingNameFor`, `exportLines`) dts.ts shares       |
| `src/dts.ts`                                   | Emits the `.d.ts` sidecar                                                                            |
| `src/runtime.ts`                               | `PhpInstance` (a `PhpInterpreter` plus call streaming, stdout modes, the `--hot` cache) and `$`-API  |
| `src/inline.ts`                                | The `` BunPHP`...` `` tagged template for file-less snippets                                         |
| `src/marshal.ts`                               | The JS ⇄ PHP call protocol                                                                           |
| `src/project.ts`                               | Walks up from a `.php` file to find its Composer root and autoloader                                 |
| `src/php-runtime.ts`                           | The only module that _calls_ `@php-wasm/*`: version→build map, `bootPhp`, journal ops, mount handler |
| `src/interpreter.ts`                           | `PhpInterpreter` — lazy boot, journal, `cli()`, and the `isolation: "process"` dispatch              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khromov/bun-php](https://github.com/khromov/bun-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
