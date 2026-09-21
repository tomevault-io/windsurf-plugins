---
trigger: always_on
description: Read `README.md` first: the stub-shadowing pattern, the sync machinery, and
---

# turbo-ext — instructions for working on the native extension

Read `README.md` first: the stub-shadowing pattern, the sync machinery, and
the seven **Design rules for new ports** there are binding. This file is the
operational checklist on top of them.

## Before porting anything: estimate, then decide

Ports pay off by absorbing *call frames*, priced at roughly 40ns per absorbed
userland frame. Count calls first (SPX: `SPX_ENABLED=1` on a self-analysis
run) and multiply — a site called 400× per run can never pay; a site absorbing
millions of tiny calls can. Three finished, correct, all-tests-green tier-1
ports were reverted because they measured ≈0% — being correct is not the bar,
being ≥0.5% faster is. When the estimate is marginal, don't port.

## Shadowing new code — do the steps in this order

1. **Extract the PHP code** into a dedicated class (static methods are fine)
   under `src/`, called unconditionally from the original sites — no
   turbo-conditional branches in callers. Find **all** call sites (beware:
   `grep "\$this->foo"` in double quotes sends `\$` to grep and silently
   matches nothing — use single quotes). Run the full test suite now, before
   any native work.
2. **Check it has no parent class** — the stub shell extends the native
   class and PHP is single-inheritance, so the collector rejects it. If it is
   a DI service, its native `__construct` arginfo must declare the real
   parameter class names (rule 6 in README): Nette autowires by reflecting
   the constructor, and erased types fail container compilation for every
   shadowed service at once.
3. **Implement natively**: one class per `.cpp` in `src/`, namespace
   `PHPStanTurbo`, class **non-final**, `instanceof`-style checks instead of
   exact class-entry comparisons. Hot classes are registered with the raw
   Zend API in `main.cpp`'s MINIT (raw handler pointers, nothing that
   allocates per call). Reuse the `pt_*` helpers in
   `support.h`/`support.cpp` before writing new ones.

   **Style**: the logic lives in a C++ handle class in `namespace
   phpstan_turbo` that mirrors the PHP twin method for method (see
   `TrinaryLogic.cpp` as the reference; `and`/`or` keyword clashes get a
   trailing underscore); registration goes through the `reg::Class` builder
   in `reg.h` — one `cls.method("name", flags, requiredArgs, { args... },
   lambda)` declaration per method, where the lambda body is only
   ZEND_PARSE_PARAMETERS glue + one delegation line (see TrinaryLogic.cpp).
   Never introduce per-call argument boxing in a registration path — raw
   handler pointers only. Use the zero-cost
   wrappers in `zv.h` — borrowed `zv::Ref` views vs owned move-only
   `zv::Val` RAII values (UNDEF `Val` = pending exception), `zv::ArrRef`
   range-for instead of hand-rolled `ZEND_HASH_FOREACH` (it handles the
   packed layout of PHP 8.2+ — never walk Buckets by hand). Zero-cost is
   the bar: no virtuals, no exceptions, no allocations the raw form would
   not make; where an abstraction is not provably free, keep the raw zend
   form and say so in a comment. New generic helpers go into `zv.h`
   following its conventions, never as one-offs.
4. **Class names the native code needs** go through
   `Runtime::configure()`, fed from the generated `vendor/turbo-class-map.php`:
   add the key to `pt_class_refs` in `support.cpp` and mark the referenced
   class with `#[ReferencedByTurboExtension(key: '...')]` (vendored PhpParser
   classes are hardcoded in `build/TurboAttributeCollector.php` instead —
   `tests/smoke.php` holds the map against the real compiled table via
   `Runtime::classRefs()`). A referenced class that is itself shadowed is one
   the native code *instantiates*: its table entry gets no default name, and
   the resolved name is the stub subclass, so created instances satisfy the
   original type hints.
5. **Mark the class** with `#[ShadowedByTurboExtension(turboClass:
   'PHPStanTurbo\Foo', implementation: __DIR__ . '/../turbo-ext/src/Foo.cpp')]`
   and run `composer dump-autoload` — `build/generate-turbo-stubs.php`
   regenerates the stub shells in `vendor/turbo-stubs.php`, the manifest of
   shadowed pairs in `vendor/turbo-shadowed-classes.json` and the class map
   in `vendor/turbo-class-map.php` from the attributes (shadowed classes
   living in vendor/ cannot carry the attribute and are hardcoded in
   `build/TurboAttributeCollector.php`).
6. **Check method parity**: `php bin/side-by-side.php` must pass (it also
   re-derives the generated `vendor/turbo-*` files from the attributes and
   byte-compares them, so a stale autoloader dump fails there).
7. **Extend `tests/smoke.php`** with differential coverage (native result
   must equal the PHP implementation's result on the same inputs) and
   register the class in `$covered` next to its checks — the completeness
   check at the end fails for any shadowed class with no registered
   coverage.
8. **Verify**: strict build, smoke test,
   `php -d extension=$PWD/turbo-ext/phpstan_turbo.so turbo-ext/tests/signature-parity.php`
   (arginfo parameter names must match the PHP twin exactly — named arguments),
   full `make tests` with the extension loaded, and byte-identical analysis
   output with the extension loaded vs. not loaded. Anything touching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpstan/turbo-ext](https://github.com/phpstan/turbo-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
