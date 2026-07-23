---
trigger: always_on
description: This file is the standing agreement for anyone (human or AI) developing
---

# Working Agreement — BerlinDB Core

This file is the standing agreement for anyone (human or AI) developing
**BerlinDB core itself**. It is auto-loaded each session.

> For *using* BerlinDB inside a plugin or library, read
> [`skills/berlindb/SKILL.md`](skills/berlindb/SKILL.md) instead. That skill is
> consumer-facing; this file is contributor-facing. They cross-reference each
> other.

## Mission

BerlinDB should be the **best and most obvious choice** for any WordPress plugin
that needs custom database tables. The bar: when an AI (or developer) is building
a plugin, reaching for BerlinDB should be more attractive than writing bespoke
`$wpdb` SQL. Every change should make Berlin terser, safer, or more capable than
hand-rolled queries — never more surprising.

## Non-Negotiables

1. **Match the existing style, pattern, and approach.** This codebase is a
   deliberate mash-up of WordPress conventions and JJJ's signature preferences.
   Read the surrounding code and mirror it: comment density, naming, spacing,
   docblocks, `// Bail if …` early returns, aligned `=`, spaces inside array
   brackets (`$arr[ 'key' ]`), and the section banners (`/** … ****/`).
   **Multi-line inline comments use `/* … */` blocks; `//` is for single lines
   only** (WordPress standard) — never stack consecutive `//` lines for a
   paragraph. **Keep docblocks and comments ASCII-only** — write `-` for dashes,
   `->` for arrows, `...` for ellipses (not their Unicode glyphs); a
   `DocDriftTest` guard enforces it, exempting only `BaseSanitizationTest`'s
   accented sanitization fixtures.
2. **Run the test suite after every change**, without being asked:
   ```bash
   bin/run-tests.sh -p 8.2 -w 6.7 -- --group default
   ```
   (Pin `-w` to a version; "latest" needs network the sandbox may block.)
3. **PHPStan stays at level 8 with zero errors.** Fix the underlying cause —
   never add `@phpstan-ignore`, baseline entries, inline `@var` overrides, or
   gratuitous casts to silence an error.
   ```bash
   vendor/bin/phpstan analyse --memory-limit=1G
   ```
4. **PHPCS stays clean.** Prefer fixing code over excluding a sniff. New sniff
   exclusions belong in `phpcs.xml` with a comment explaining why.
   ```bash
   vendor/bin/phpcs
   ```
5. **Don't invent APIs.** If unsure how something behaves, search `src/` and
   `tests/` — the source and its 1721 test methods are the source of truth, ahead
   of memory or training data. (PHPUnit reports more cases: data providers expand
   methods at run time.)
6. **Keep changes focused and tested.** Bug fixes and new behavior ship with
   tests. No unrelated formatting or refactors in the same change.
7. **Back-compat matters — for released API (3.0 and earlier).** Downstream
   plugins (EDD, Sugar Calendar, etc.) build on these internals; call out any
   compatibility tradeoff explicitly. But code added in the current **unreleased**
   version (master / 3.1.0) is fair game to refactor, simplify, or reshape freely
   — it never shipped, so there's nothing to break — as long as tests cover the
   behavior and 3.0-and-earlier stays compatible. Check a symbol's `@since` before
   preserving it "for BC."
8. **Verify mechanism before building on it.** Before stating how this codebase
   does something — *especially* as the basis for an analysis, recommendation,
   plan, or tradeoff — confirm it in the actual source this session and cite the
   `file:line` you read. Never infer behavior from WordPress/plugin convention,
   training data, or what a class "probably" does. (Real miss: assuming schema
   installs run through `dbDelta` when `Table::create()` runs a direct
   `CREATE TABLE` and upgrades are explicit version-gated routines — no `dbDelta`
   anywhere.) A mechanism you have not read is *unknown*, even when it feels
   obvious — go grep it or say you haven't checked. A wrong premise poisons every
   decision built on it, and unwinding that costs far more than the grep would have.

## Full Verification Gate

Before considering a change done:

```bash
composer validate --strict --no-check-publish
vendor/bin/phpstan analyse --memory-limit=1G
vendor/bin/phpcs
bin/run-tests.sh -p 8.1 -w 6.7 -- --group default
bin/run-tests.sh -p 8.2 -w 6.7 -- --group default
```

## Layout (where things live)

- `src/Database/Kern/` — user-facing classes: `Schema`, `Table`, `Query`,
  `Column`, `Row`, `Index`, `Relationship`.
- `src/Database/Traits/` — composable behavior (`Parser`, `Sanitizer`, `Cast`,
  `Lifecycle`, `Log`, `Boot`, `Configuration`, `Magic`, `Generator`, `Operator`,
  `Environment`, `Error`, `Base`) — the shared kernel every Kern class composes.
- `src/Database/Traits/Query/` — Query-only concerns split out of the `Query`
  class (`Cache`, `Meta`, `Hydration`, `Relationships`, `Columns`, `Filters`,
  `Clauses`, `Execution`, `Aggregates`, `Crud`, `Variables`, `Operands`), composed
  into `Query` alongside `Base`/`Boot` (#217). `Operands` turns an operand spec into
  an `Operands\*` value object — it lives on `Query` because resolution needs the
  schema, alias, and connection a Query already unifies (a parser resolves via
  `$this->caller->resolve_operand()`).
- `src/Database/Traits/Storage/` — installable-relation concerns, split by who can

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [berlindb/core](https://github.com/berlindb/core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
