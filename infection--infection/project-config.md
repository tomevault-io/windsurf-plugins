---
trigger: always_on
description: Operating guide for AI agents contributing to Infection. It is read once per session, so it is
---

# AGENTS.md

Operating guide for AI agents contributing to Infection. It is read once per session, so it is
intentionally dense. Each line must help an agent avoid a known mistake. Claims are anchored
to files - trust the tree over your training data. This codebase was heavily re-architected
through 2025-2026, so old assumptions can be wrong. When your change makes a line here false
- a path moves, a version gate shifts, a convention evolves - update that line in the same
task. If you find a stale line while reading, fix it too.

## What you are working on

Infection is the mutation testing framework for PHP - millions of Packagist installs, and
PHPUnit, PHP-CS-Fixer, and PHPStan are all developed with it. It parses source into ASTs
(nikic/php-parser - API
refresher: `vendor/nikic/php-parser/README.md` and its `doc/` folder), applies
small mutations, and checks whether the project's tests - and optionally a static analyser -
notice. A bug here silently corrupts other projects' quality gates. A slowdown here multiplies
by thousands of mutant processes per user run. Reviewers weigh correctness, memory, and
per-process overhead equally.

The project tests itself with Infection: CI runs Infection against this repository with a
minimum-MSI gate (`.github/workflows/mt.yaml`) and annotates escaped mutants inline on PRs.
That fact explains most of the coding rules below: code is written so that every mutation of
it dies.

## Vocabulary

The canonical glossary (with citations to the mutation-testing literature) is transcluded
below. Use its terms in code, tests, and PR titles; reviewers rename code that does not.
Two Infection-specific notes on top of it: a `Mutation` object is a *serializable*
description of a change (created during analysis, applied later, possibly in another
process), and a mutator is concretely `canMutate()` + `mutate()`.

@doc/nomenclature.md

## The execution pipeline

One pass, phase by phase (diagram: `doc/nomenclature.md#execution-phases`):

1. **CLI entry** - `bin/infection`, `src/Command/RunCommand.php`. Parses options, then binds
   them into the container via `Container::withValues(...)`.
2. **Container** - `src/Container/Container.php` (~1,260 lines). One flat registry of lazy
   closure factories plus a typed getter per service. It is built on `sanmai/di-container`.
3. **Engine** - `src/Engine.php` orchestrates everything below.
4. **Source collection** - `src/Source/` (`SourceCollector` implementations, including the
   git-diff collector); `src/Configuration/PositionalPathsClassifier.php` classifies
   positional CLI paths into source vs test files.
5. **Artefact collection** - initial test run with coverage
   (`src/Process/OriginalPhpProcess.php` re-enables xdebug/pcov for exactly this child).
   Then coverage-xml + junit.xml are ingested into lazy `Trace` objects
   (`src/TestFramework/Coverage/`, `src/TestFramework/Tracing/`).
6. **AST parsing + enrichment** - `src/PhpParser/NodeTraverserFactory.php` runs a single
   ordered visitor stack. It labels eligibility, resolves names, connects parents and next
   statements, attaches reflection and lazy covering-test lookups, and marks unchanged,
   user-ignored, or untested code as ineligible.
7. **Mutation generation + heuristic suppression** - `src/Mutation/FileMutationGenerator.php`,
   `src/Mutator/NodeMutationGenerator.php`; mutators live under `src/Mutator/<Category>/`.
8. **Mutant materialisation + evaluation** - `src/Mutant/MutantCodeFactory.php` splices the
   replacement node by token positions and prints a minimal diff.
   `src/Process/Runner/ParallelProcessRunner.php` streams mutant processes at N threads. An
   escaped mutant may get a follow-up static-analysis process
   (`src/StaticAnalysis/` - PHPStan and Mago adapters).
9. **Reporting** - metrics in `src/Metrics/` (Welford-based running variance for timings),
   loggers/reporters in `src/Logger/` and `src/Reporter/` (legacy) plus the newer
   `src/Report/` framework that is gradually replacing them.

## Repo map

- `src/` - production code. PSR-4 `Infection\`. Nothing under `src/` may depend on `tests/`
  or benchmarks (PHPat-enforced); shipped test helpers live in `src/Testing/` for
  this reason.
- `tests/phpunit/` - unit/integration tests, mirroring `src/` one-to-one. Every concrete
  source class must have a canonical test named after it (PHPat rule).
- `tests/Architecture/PHPat/` - architecture and convention fitness rules (finality, `@internal`, canonical
  tests, IO-vs-`integration`-group, event conventions, src-not-depending-on-tests). They run
  under PHPStan via `devTools/phpstan.neon`, not under PHPUnit. The selectors' own tests are
  in `tests/phpunit/Architecture/`.
- `tests/phpunit/AutoReview/` - convention tests run by `phpunit_autoreview.xml`: mutator
  API shape, Definition presence, env-var hygiene, Makefile consistency, no mutable public
  properties (DTO whitelist in `tests/phpunit/AutoReview/ProjectCode/ProjectCodeProvider.php`).
- `tests/e2e/` - one directory per scenario; anatomy, the `tests/e2e_tests` runner, and the
  `tests/add_new_e2e` scaffold are covered by CONTRIBUTING.md, transcluded at the end of
  this section. On top of that: fixtures pin `"threads"` (usually 1 - parallel output is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infection/infection](https://github.com/infection/infection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
