---
trigger: always_on
description: Psalm plugin for Laravel. It boots a real Laravel application (the analyzed project's own `bootstrap/app.php`, with Orchestra Testbench as the package fallback), then hooks Psalm's event system and registers stubs to type Laravel's magic. It also ships taint sources, sinks, and escapes for security analysis.
---

# psalm-plugin-laravel

Psalm plugin for Laravel. It boots a real Laravel application (the analyzed project's own `bootstrap/app.php`, with Orchestra Testbench as the package fallback), then hooks Psalm's event system and registers stubs to type Laravel's magic. It also ships taint sources, sinks, and escapes for security analysis.

Active majors:
- `master` is 4.x (PHP 8.2+, Laravel `^12.14 || ^13.3`, Psalm 7 beta)
- `3.x` is the Psalm 6 line (Laravel `^11.35+`), backports only

Taint: Psalm 6 runs in exactly one mode per invocation: plain `psalm` reports type issues only, `psalm --taint-analysis` reports taint issues only, so full coverage takes two runs. Psalm 7 runs taint BY DEFAULT and emits type and taint issues together in one run, on a rewritten engine with different internals; `psalm-laravel init` additionally writes `runTaintAnalysis="true"` to make that explicit.

## Read before re-deriving

These documents are maintained and source-verified. Consult them before exploring source; prefer them over re-deriving from code.

| Topic | Where |
|---|---|
| How to add a stub or handler; boot flow, hook phases, stub layout/merging, version dirs, SKIPIF gating, experimental issue lifecycle | `docs/contributing/README.md` |
| Design rationale: handler loading, producer narrowing, suppression, version policy, taint philosophy, performance budget | `docs/contributing/decisions.md` |
| How Laravel `__call` / `__callStatic` / macros / forwarding resolve, and the handlers modeling them | `docs/contributing/laravel-magic-call-patterns.md` |
| Taint annotation authoring | `docs/contributing/taint-analysis.md` |
| Psalm annotation syntax reference | `docs/contributing/types.md` |
| Keeping self-analysis at 100% type coverage | `docs/contributing/type-coverage.md` |
| Handler, stub, and test authoring patterns (mined from this codebase, with evidence) | `docs/contributing/code-patterns.md` |
| Type test (.phpt) format and assertions | `tests/Type/README.md` |
| Application fixture and archetype models (reuse before adding) | `tests/Application/README.md` |
| Custom issue documentation, one page per `src/Issues/` class | `docs/issues/<Name>.md` + `docs/issues/index.md` |
| Plugin XML config flags (opt-in issues, `columnFallback`, ...) | `docs/config.md` |
| Step-debugging a handler | `docs/contributing/xdebug.md` |

Ground truth for stub signatures is Laravel source in `vendor/laravel/framework/`, never Laravel's own PHPDoc and never another tool's stubs.

## Commands

Use these forms by default (verbose output only when debugging):

```bash
composer test:unit -- --no-progress --colors=never --display-errors --display-warnings  # PHPUnit unit tests
composer test:type -- --no-progress                                    # type tests (psalm-tester)
composer test:app                                                      # plugin on a fresh Laravel app
composer psalm -- --no-progress --no-suggestions --output-format=compact  # self-analysis
composer cs                                                            # auto-fix code style (quiet flags baked in)
composer rector -- --no-progress-bar --no-ansi                         # rector refactoring

# single test file, cheaper than the suite
./vendor/bin/phpunit tests/Unit/PluginConfigTest.php
./vendor/bin/phpunit --filter=AuthTest tests/Type/
```

Gotchas:

- The final pre-commit psalm gate must DROP `--no-suggestions` to match CI, which runs bare `psalm` (`.github/workflows/psalm.yml`) and reports what the flag hides locally. Keep the flag for iteration only.
- Before citing Psalm source or defaults, confirm `vendor/composer/installed.json` carries the Psalm major that composer.json requires: a stale vendor from an interrupted install answers every source read and probe with the PREVIOUS major's behavior, convincingly.

## Git and PRs

- Base PRs on `master`. Psalm-6-only bugs base on `3.x`.
- The `style: auto-fix` workflow commits style fixes back to pushed branches. Run `composer rector` and `composer cs` locally BEFORE pushing, and `git pull --ff-only` before any further local edits after a push.
- Worktrees: the symlinked `vendor/` PSR-4 autoloader points at the primary checkout's `src/` and `stubs/`. Edits in a worktree are invisible to Psalm until `composer install` runs inside that worktree. Write and Edit tools must target worktree-absolute paths, never the primary root.
- Commits follow Conventional Commits. The subject describes the change, not the issue it closes; issue ref is required in PR body and optional in commit message body.

## Hard rules

Each rule exists because violating it shipped a bug. The pointer holds the full story.

1. Every handler in `Plugin::registerHandlers()` keeps its paired `require_once`. Rationale: decisions.md, "Class Loading and Discovery".
2. A stub that re-declares a class must copy the `extends` / `implements` / `use` header verbatim from Laravel source. Psalm wipes the reflected interface list on re-declaration. See "Stub merging" in the contributing README.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psalm/psalm-plugin-laravel](https://github.com/psalm/psalm-plugin-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
