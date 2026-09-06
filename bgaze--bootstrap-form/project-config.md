---
trigger: always_on
description: Bootstrap 4/5 forms builder for Laravel 12+. Composer **package** (library, not an app): builds forms via a
---

# bgaze/bootstrap-form

Bootstrap 4/5 forms builder for Laravel 12+. Composer **package** (library, not an app): builds forms via a
`BF` facade, Blade directives and Blade x-components, rendering HTML through its **own owned renderer** (no
third-party form/HTML dependency). Renders **Bootstrap 5 by default**; Bootstrap 4 is **fully supported for
backward compatibility** (`bootstrap_version` config, or per form/field). Public open-source (GitHub /
Packagist).

> **This file is public.** It lives in a public repository and ships inside every consumer's `vendor/`
> directory. Keep it self-contained: repo-relative paths and public URLs only — no absolute or machine-local
> paths, no personal tooling, no internal process references. Anything a reader outside this repository
> cannot reach does not belong here.

## Stack

| Item      | Value                                                                                                       |
|-----------|-------------------------------------------------------------------------------------------------------------|
| Language  | PHP ≥ 8.2 (`declare(strict_types=1)`, native types)                                                          |
| Framework | Laravel 12 or 13 (illuminate/* `^12.0 \|\| ^13.0`)                                                           |
| Autoload  | PSR-4 `Bgaze\BootstrapForm\` → `src/`                                                                        |
| Tests     | PHPUnit 11 + Orchestra Testbench (`^10.0 \|\| ^11.0`) — byte-exact HTML characterization suite in `tests/`    |
| Quality   | PHPStan level 5 + Larastan (`phpstan.neon`, `src` only) · Pint, `laravel` preset                              |

## Versioning & releases

- `master` carries the **current major** (v4). The `v1` / `v2` / `v3` branches are the older-major maintenance
  lines; there is no `v4` branch — v4 work lands on `master`.
- **Tag only after the GitHub Actions pipeline is green** (phpunit matrix + PHPStan + Pint).
- **Published tags are immutable** — never force-move one; cut a new patch instead.
- **Every tag gets a matching GitHub Release** with human-readable notes (`gh release create`), the newest
  marked `--latest`. Packagist consumes the tag; the Release is the public changelog and the watcher
  notification.
- v4 dropped the historical `laravelcollective/html` dependency in favor of the internal, iso-rendering
  HTML/form layer described below.

## Architecture

- `src/BootstrapFormServiceProvider.php` — registers the `BF` facade, the Blade directives and the `bf`
  x-component namespace; publishes `src/config/config.php`.
- `src/BootstrapForm.php` — builder entry point, backing the `BF` facade (`Bgaze\BootstrapForm\Support\Facades\BF`).
  Exposes the owned units via `html()` / `elements()` / `fieldValue()` / `context()`.
- `src/Inputs/` — field types (Text, Check, CheckChoice, File, Range, Select).
- `src/Support/` — **owned HTML/form layer** (successor of the collective-html dependency):
  - `Html` — stateless attribute/tag serialization primitive (SSOT of attribute order & escaping).
  - `FieldValue` — value binding resolver (old input, model, checked/selected state).
  - `FormContext` — per-form binding state (bound model, CSRF token, url/view/session services).
  - `FormElements` — element & form-open renderer, composing `Html` + `FieldValue` + `FormContext`.
  - `Options` — SSOT partitioning raw options into settings vs HTML attributes (+ the `~` literal escape).
  - `Attributes` — ordered attribute value object; `~` (`LITERAL_PREFIX`) emits an HTML attribute whose name
    collides with a setting. Plus `Input` and traits `HasAddons` / `HasSettings`.
- `src/View/Components/` — the x-components. They **delegate to the facade**, so their output is byte-identical
  to it; the projection rules (`label:`, `group:`, `input:`, `option:`, `optgroup:`, boolean shortcuts) live in
  `Concerns/`.
- `src/Support/Drivers/` — **version drivers**: `VersionDriver` (abstract, shared tokens) + `Bootstrap4Driver` /
  `Bootstrap5Driver` (version deltas) + `DriverManager` (resolves by version). All Bootstrap component classes and
  the structural divergences (input-group, custom-file, check/switch) live here — **no Bootstrap class literal exists
  outside a driver**. `Input` subclasses consume the driver.

## Bootstrap version

- `config/config.php`: `bootstrap_version` (4 | 5, **default 5**) selects the driver; layout-level, app-tunable
  options live under version sections `bootstrap4` / `bootstrap5`. Component classes are native/fixed (driver code),
  not configurable.
- Resolution: global default ← per-form override (`BF::open(['bootstrap_version' => 4])`) ← per-field override. A
  per-field override switches the driver (component classes); layout settings stay inherited from the form.
- **B4 is frozen** (compatibility only); **B5 is the default** and where new work happens. `custom` is a Bootstrap 4
  concept (native vs custom controls) and is a **no-op in Bootstrap 5** (styles unified); it stays a recognized
  setting in both versions so it is never emitted as an HTML attribute.
- Bootstrap 5 inline forms are **best-effort** (B5 reworked inline layout); vertical and horizontal are fully supported.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgaze/bootstrap-form](https://github.com/bgaze/bootstrap-form) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
