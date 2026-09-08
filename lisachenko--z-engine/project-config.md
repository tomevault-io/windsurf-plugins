---
trigger: always_on
description: z-engine reaches into the Zend Engine's own memory through PHP FFI. That makes
---

# Working on z-engine

z-engine reaches into the Zend Engine's own memory through PHP FFI. That makes
it uniquely powerful and uniquely fragile: a wrong struct offset or a call
against the wrong PHP version does not throw — it corrupts memory and segfaults
the interpreter. These rules exist to keep that from happening. They apply to
human contributors and automated agents alike.

## The one rule that is non-negotiable: version matching

**Never run z-engine code or tests against a PHP minor version other than the
one the current branch targets.** The engine's C structures change between
every minor version (`zend_class_entry` alone changed size in 8.1, 8.3 and
8.4). z-engine reads those structures by offset. Run it on a mismatched
version and you are reading and writing the wrong memory — the result is a
crash, or worse, silent corruption.

- `master` targets the newest supported PHP minor (currently **8.5**).
- Branch `8.4` targets **PHP 8.4**.
- Branch `8.0` is the frozen legacy line for PHP 8.0.

`Core::init()` enforces this at runtime and refuses to boot on the wrong minor.
Do not try to defeat that guard.

## Branch model

Fixes land on the **minimum affected version branch** and are merged *upward*,
never cherry-picked downward. The succession is declared in
`.github/branch-flow.json` and automated by `.github/workflows/merge-up.yml`,
which opens a merge-up PR when a version branch is pushed.

```
8.0 (frozen)      8.4  ──►  master (8.5)
```

So a bug that exists in both 8.4 and 8.5 is fixed on `8.4`, and the cascade
carries it into `master`. A bug that only exists on 8.5 is fixed on `master`
directly. When resolving a merge-up conflict inside `include/`, do **not**
merge the generated headers textually — regenerate them on the target branch
(`composer gen-headers`) instead.

## Generated engine definitions — never hand-edit

Everything under `include/<minor>/<os>-<arch>-<ts>/` is generated:

| File | What it is |
|------|-----------|
| `engine.h` | FFI header (structs, functions, globals) sliced from the PHP source |
| `constants.php` | `#define`/enum/opcode values, the ground truth for the PHP class constants |
| `layouts.json` | `sizeof`/`offsetof` of every dereferenced struct, from the C compiler |
| `probe.c` | the generated C probe (kept so a probe-only run can reuse it) |

Two more generated artifacts live at the branch level (not per-platform) and come
out of the same pipeline — the canonical `linux-x64-nts` target publishes them and
every other target byte-compares against them (see the struct-stub note below):

| File | What it is |
|------|-----------|
| `stubs/zend-engine-structs.php` | one analysis-only PHP class per engine struct (`ZEngine\Generated\*`), never loaded — see "Engine structs are typed by generated stub classes" |
| `.phpstorm.meta.php` | PhpStorm type map for the `Core::new()/cast()` legacy string literals |

This branch maintains **two thread-safety targets**: `linux-x64-nts` and
`linux-x64-zts` (the manifest in `tools/generator/symbols.php` is
thread-safety-aware — on ZTS the per-thread EG/CG are reached through the TSRM
offsets instead of the plain extern symbols, see issue #60). Regenerate them
with:

```bash
composer gen-headers          # all targets for this branch (needs Docker on Linux)
```

The generator (`tools/generator/`) runs inside the official `php:<minor>` Docker
image so the artifacts always match a real build. Regenerate whenever you:

- bump the branch to a new PHP minor,
- add or remove an engine symbol in `tools/generator/symbols.php`,
- or CI's `header-drift` job goes red.

If you touch a struct the PHP code dereferences, add it to `layout_structs` in
`symbols.php` so its layout is verified. The generator's own validation stage
FFI-loads the header and asserts every offset against the C compiler, so a
wrong header cannot be produced.

### Regenerating without Docker (native mode)

`generate.php --native` runs the pipeline directly on the host — no Docker.
It is auto-selected on non-Linux hosts (a Docker container is Linux by
construction, so it can never produce e.g. darwin artifacts) and is also the
escape hatch for sandboxed/proxied Linux environments where Docker or the
Debian package mirrors are unreachable. `emit.php` derives everything from the
*running* PHP build (`php-config --includes`, clang over the real headers, a C
probe compiled with `cc`); the php-src tree is only needed to slice the private
structs, and native mode fetches exactly those three files
(`Zend/zend_closures.c`, `ext/opcache/ZendAccelerator.h`,
`ext/opcache/zend_file_cache.c`) from
`raw.githubusercontent.com/php/php-src/php-<version>/` automatically (or use
`--php-src=DIR` to point at a matching tree).

```bash
php tools/generator/generate.php --native   # generates for the running interpreter
```

Native mode generates **for the interpreter that runs it only**: host needs
`clang`, `cc`, `php-config` matching the exact running PHP version, and
ext-ffi (on Windows there is no `php-config`/`cc` — see the Windows section
below for what replaces them). For a `zts` target the running PHP must itself
be a matching `--enable-zts` **release** build of the same minor (emit.php
derives the thread-safety mode, the TSRM symbols and the layouts from the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lisachenko/z-engine](https://github.com/lisachenko/z-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
