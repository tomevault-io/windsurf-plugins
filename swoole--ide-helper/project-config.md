---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This package provides IDE helper ("stub") files for [Swoole](https://github.com/swoole/swoole-src), the PHP
coroutine/async C extension. It contains no executable logic of its own — every method body is empty. The sole
purpose of each file is to give IDEs (PhpStorm, VS Code, etc.) accurate autocompletion, type hints, and inline
documentation for classes/functions/constants that are actually implemented in C by the Swoole extension (and, for
one subtree, in the companion `swoole/library` PHP package).

There are two distinct kinds of source under `src/`, and they are maintained very differently:

- `src/swoole/` — pure stubs for the Swoole C extension. Every class/method/function here mirrors a symbol exported
  by the extension. Method/function bodies are always empty (`{ }`); all information lives in PHPDoc blocks.
- `src/swoole_library/` — a verbatim copy of the PHP userland source from https://github.com/swoole/library (the
  `swoole/library` package that ships inside the Swoole extension via `swoole.enable_library`). This is real,
  runnable PHP code, not stubs. It gets updated by copying files over from that upstream repo, not by hand-editing
  individual signatures.

Layout inside `src/swoole/`:
- `constants.php` — all `SWOOLE_*` constant definitions.
- `functions.php` — global `swoole_*()` procedural functions.
- `shortnames.php` — `class_alias()` calls for short names (e.g. `Co\Channel` → `Swoole\Coroutine\Channel`), active
  only when the `swoole.use_shortname` ini directive is on.
- `Swoole/**` — one file per class, in a directory structure mirroring the `Swoole\...` namespace (e.g.
  `Swoole/Coroutine/Http/Client.php` defines `Swoole\Coroutine\Http\Client`).

## Commands

There is no local PHP toolchain expected to be installed (`composer.json` declares no dependencies). CI and local
checks both run through the `jakzal/phpqa` Docker image.

Check coding style (dry run, matches CI):
```bash
docker run -q --rm -v "$(pwd):/project" -w /project -i jakzal/phpqa:php8.5-alpine php-cs-fixer fix --dry-run
```

Auto-fix coding style:
```bash
docker run -q --rm -v "$(pwd):/project" -w /project -i jakzal/phpqa:php8.5-alpine php-cs-fixer fix
```

Check PHP syntax (CI runs this for 8.1, 8.2, 8.3, 8.4, and 8.5 — see `.github/workflows/syntax_checks.yml`):
```bash
docker run -q --rm -v "$(pwd):/project" -w /project -i jakzal/phpqa:php8.1-alpine phplint src
```

Locally, run the syntax check against `php8.1-alpine` specifically. PHP's parser is backward-permissive: a construct
only PHP 8.2+ understands (a standalone `false` type, a DNF type) parses fine under 8.5 but fails under 8.1, so 8.1 is
the only version whose parser actually enforces the "inline type declarations must be valid PHP 8.1 syntax"
convention below. Swap the version segment of the image tag to check any other supported version.

There is no test suite — correctness here means "the stub's signature/docblock matches upstream Swoole," not
behavior, since no method body ever executes.

`php-cs-fixer` rules are defined in `.php-cs-fixer.dist.php` and explicitly exclude `swoole_library/` (that
directory keeps upstream's own formatting since it's copied verbatim).

## Stub-writing conventions

This is the main recurring task in this repository. When bringing the stubs up to date with a Swoole release,
compare against the actual C source/headers in https://github.com/swoole/swoole-src for that release, and examine
the actual implementation and changes of that PHP class/method/function in the release (and
https://github.com/swoole/library for anything under `src/swoole_library/`). **Never ever** use or trust the
`.stub.php` files shipped in a Swoole release — they are not a reliable source for this work. Then apply these
conventions consistently — they are what every existing file already follows and what reviewers expect:

- **Above all, write for a PHP developer, not a C developer.** Comments/stubs are meant to be easy to understand,
  unless the technical detail being documented genuinely can't be explained accurately in plain, simple words —
  don't reach for jargon or low-level precision the reader didn't ask for.
- **Stay at the PHP level whenever possible.** When accuracy requires mentioning a system call or another piece of
  the underlying implementation (e.g., `msgget(2)`, `ftok(3)`, `mmap()`, a row/hashtable struct layout), phrase it in
  plain language a PHP developer can follow without prior C/POSIX knowledge, and add a `@see` link to a man page or
  other reference so a reader who wants the full technical depth can dig further on their own — the docblock itself
  should not require that background to be understood.
- **Completeness and typing is the baseline, not a special case.** A symbol existing in the stub isn't enough —
  every public property must carry a native PHP type declaration that accurately reflects what swoole-src actually
  stores there (nullable/union types where applicable; never leave a bare, untyped `public $x;` just because the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swoole/ide-helper](https://github.com/swoole/ide-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
