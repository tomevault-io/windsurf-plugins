---
trigger: always_on
description: Winter core sits on top of [Winter Storm](https://github.com/wintercms/storm) (installed at `vendor/winter/storm/`), which sits on top of Laravel, which sits on top of Symfony components. The helper you want almost certainly already exists at one of those layers. **Search Storm → Laravel → Symfony before writing any "small utility"**, often with safer edge-case handling than a fresh implementation would have.
---

# Agent guide — Winter CMS core

Winter core sits on top of [Winter Storm](https://github.com/wintercms/storm) (installed at `vendor/winter/storm/`), which sits on top of Laravel, which sits on top of Symfony components. The helper you want almost certainly already exists at one of those layers. **Search Storm → Laravel → Symfony before writing any "small utility"**, often with safer edge-case handling than a fresh implementation would have.

## Where to look (in this order)

1. **Storm itself** — each `vendor/winter/storm/src/<Module>/README.md` catalogues that module's public API:
   - `Filesystem/` — `PathResolver` (`resolve`, `within`, `join`, `standardize`), `Filesystem` (extends Illuminate's; adds `isAbsolutePath`, `symbolizePath`, `existsInsensitive`, `chmodRecursive`)
   - `Support/` — strings, arrays, class loading
   - `Network/`, `Html/`, `Parse/`, `Database/`, `Halcyon/`, `Auth/`, etc.
   - Path helpers (always loaded): `themes_path()`, `plugins_path()`, `media_path()`, `uploads_path()`, `temp_path()` — use these instead of `base_path('themes')` etc.

2. **Laravel (Illuminate)** — everything Laravel ships is available:
   - `Illuminate\Support\Str` — `Str::startsWith/endsWith/contains/before/after/between/slug/camel/snake/kebab/studly/random/uuid/limit/mask/finish/start/of/headline/title`. Use instead of regex one-liners.
   - `Illuminate\Support\Arr` — `Arr::get/set/has/forget/only/except/dot/undot/flatten/pluck/wrap/first/last/where`. Use instead of nested foreach.
   - `Illuminate\Support\Collection` (via `collect()`) — chainable map/filter/reduce.
   - `Illuminate\Filesystem\Filesystem` — `deleteDirectory()`, `cleanDirectory()`, `copyDirectory()`, `moveDirectory()`, `allFiles()`, `glob()`, `isDirectory()`, `prepend()`, `append()`, `replace()`, `hash()`.
   - Global helpers: `data_get/set/fill`, `value`, `tap`, `optional`, `transform`, `head`, `last`, `class_basename`, `now`, `today`, `e`, `__`/`trans`, `cache`, `config`, `env`, `app`, `resolve`, `route`, `url`, `report`, `rescue`, `retry`, `throw_if`/`unless`, `abort`/`abort_if`/`unless`.
   - Facades: `Cache`, `Config`, `DB`, `Event`, `File`, `Hash`, `Http`, `Lang`, `Log`, `Mail`, `Queue`, `Redis`, `Route`, `Schema`, `Session`, `Storage`, `URL`, `Validator`, `View`.

3. **Symfony components** at `vendor/symfony/`: `console`, `css-selector`, `error-handler`, `event-dispatcher`, `finder`, `http-foundation`, `http-kernel`, `mailer`, `mime`, `process`, `routing`, `string`, `translation`, `uid`, `var-dumper`, `yaml`. Most commonly reached for:
   - `Symfony\Component\Finder\Finder` — `Finder::create()->files()->name('*.less')->in($dir)` replaces RecursiveIteratorIterator chains.
   - `Symfony\Component\Filesystem\Filesystem` — `dumpFile()` (atomic write), `mirror()`, `mkdir()` (idempotent), `remove()`, `symlink()`.
   - `Symfony\Component\Process\Process` — safe external command execution instead of `exec()`/`shell_exec()`.
   - `Symfony\Component\Yaml\Yaml` — strict YAML.
   - `Symfony\Component\String\` — Unicode-aware strings.
   - `Symfony\Component\Uid\Uuid`/`Ulid` — UUID/ULID generation.

`grep -rl 'function <thing>' vendor/winter/storm/src/ vendor/laravel/framework/src/ vendor/symfony/` is a 10-second check.

## Concrete substitutions worth memorising

Paths and filesystem:

| If you reach for… | Use this instead |
|---|---|
| `realpath()` + null-check + slash-trim | `\Winter\Storm\Filesystem\PathResolver::resolve()` |
| `str_starts_with($path, $root)` to gate file access | `PathResolver::within($path, $root)` — separator-boundary safe |
| Manual `base_path('themes')` / `base_path('plugins')` | `themes_path()` / `plugins_path()` (Storm's autoloaded helpers) |
| Recursive `rmrf` in tests | `\File::deleteDirectory($path)` (Laravel facade) |
| Detect absolute path | `(new \Winter\Storm\Filesystem\Filesystem())->isAbsolutePath($path)` |
| Custom path-symbol resolution (`~/...`) | `(new \Winter\Storm\Filesystem\Filesystem())->symbolizePath($path)` |
| `str_replace('\\', '/', $path)` (cross-platform comparison) | `(new \Winter\Storm\Filesystem\Filesystem())->normalizePath($path)` |
| `str_replace('/', DIRECTORY_SEPARATOR, $path)` (handing to OS API) | `\Winter\Storm\Filesystem\PathResolver::standardize($path)` |
| Atomic file write (avoid partial-write races) | `(new \Symfony\Component\Filesystem\Filesystem())->dumpFile($path, $contents)` |
| Find files matching a pattern | `\Symfony\Component\Finder\Finder::create()->files()->name('*.ext')->in($dir)` |

Strings and arrays:

| If you reach for… | Use this instead |
|---|---|
| `preg_match('/^prefix/', $s)` | `Str::startsWith($s, 'prefix')` (accepts array of prefixes) |
| Manual `strpos !== false` | `Str::contains($s, $needle)` |
| `strtolower`-then-replace slug generation | `Str::slug($s)` |
| Random hex/string for tmp paths, tokens | `Str::random()` / `Str::uuid()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wintercms/winter](https://github.com/wintercms/winter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
