---
trigger: always_on
description: March is a statically-typed functional language (ML/Elixir hybrid) compiled with OCaml 5.3.0.
---

# March compiler

March is a statically-typed functional language (ML/Elixir hybrid) compiled with OCaml 5.3.0.

## Keeping specs up to date

**IMPORTANT:** When implementing a feature, always update `specs/todos/` and `specs/progress/` in the same commit:
- `specs/todos/` contains one open item per file, named `YYYY-MM-DD-slug.md` (date logged). See `specs/todos/README.md`.
- `specs/progress/` contains one completed item per file, same naming convention. See `specs/progress/README.md`.
- When an item is finished, `git mv` its file from `specs/todos/` to `specs/progress/` (or delete it and add a new
  dated file in `specs/progress/`) in the same commit that lands the fix; don't leave a stale open file behind.
- Don't hand-maintain a running "Current State" test count anywhere; run `scripts/run-tests.sh` for the live number.
- **Tree-sitter grammar.** A PR that changes `lib/parser/parser.mly` or `lib/lexer/lexer.mll` either extends
  `tree-sitter-march/grammar.js` or lists its new fixtures in `tree-sitter-march/known-failures.txt` (and a new
  keyword in `tree-sitter-march/keyword-allowlist.txt`) with a `specs/todos/` entry, in the same PR.
  `scripts/check-tree-sitter.sh` (CI job `tree-sitter`) enforces it.

These directories are the canonical record of what exists. Do not let them go stale. One item, one file; this
structure exists specifically so two PRs filing or closing different items never conflict with each other.

**Language reference: edit `specs/lang/`, never `docs/`.** `specs/lang/` is the canonical
language reference. The chapter pages the site serves (`docs/actors.md`, `docs/types.md`, …;
every `specs/lang/*.md` with Jekyll front matter) are GENERATED from it by
`scripts/gen-lang-docs.py` and committed. Edit the `specs/lang/` chapter, run
`scripts/gen-lang-docs.py`, and commit both; doc-lint Check F fails on a hand-edited or
un-regenerated `docs/` chapter. `docs/pagefind/` is still bot-owned: don't regenerate it in a PR.

**Doc freshness lint.** `scripts/check-docs.sh` (run in CI) guards the current-truth docs
(root guides, `docs/`, `specs/features/`, the agent SKILL) against two kinds of rot: dead
compiler-source pointers (e.g. a path that moved) and stale stdlib module counts. It also
checks the conformance-corpus INDEX counts, the quarantine aliases against their inventory
todo (Check E), and, with `CHECK_STDLIB_HTML=1` where the generated pages are expected
fresh, that every public stdlib symbol is anchored on its generated page (Check D). A
sibling, `scripts/check-runtime-sources.sh`, polices every hand-maintained runtime C link
list (drivers, JIT, dune rules) against `runtime/sources.list`; adding a runtime `.c`
means adding one manifest line with a role. `scripts/check-actor-rc-stores.sh` (also CI)
forbids a plain store to an actor record's refcount word (word 0); change it only
through `march_incrc`/`march_decrc`. It does
**not** lint the historical corpus (`specs/plans/`, dated design specs, `specs/todos/`,
`specs/progress/`). If a current doc must reference a since-removed file or a frozen
count, say so in words ("no longer exists", "removed") or add a `doc-lint:ignore-count` /
`doc-lint:ignore-file` marker; don't silently let the pointer rot.

## Maintaining the changelog

`CHANGELOG.md` (repo root, [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format)
is the user-facing digest of what shipped, a different audience than `specs/progress/`
(implementer-level detail, one file per fix). When a user-visible fix, feature, or
behavior change lands, add a bullet under `## [Unreleased]` in the same commit (`### Added`
/ `### Fixed` / `### Changed` / `### Documentation` as appropriate; see existing entries).
Skip purely internal refactors with no observable effect. When a release is tagged, rename
`[Unreleased]` to the new version + date and start a fresh empty `[Unreleased]` above it;
don't backfill history for versions that predate the file.
Once a version is tagged and a later release supersedes it, move its section verbatim to
`changelog/X.Y.Z.md` (heading kept, so `scripts/changelog-section.sh` still finds it) and
add it to the "Past releases" list at the top of `CHANGELOG.md`.

## Build & test

What CI runs, and what each job is for: `.github/workflows/README.md`.

The opam switch is `march`. `opam` and `dune` are available directly in PATH; no wrapper needed.

**NEVER use `eval $(opam env ...)` or any opam env setup prefix.** Run `dune`, `opam`, etc. directly without any preamble.

**Worktree note:** a bare `dune build`/`dune runtest` run from inside a `.claude/worktrees/*`
checkout can silently target the WRONG tree. Dune resolves its workspace root by walking up
to the *outermost* ancestor directory containing a `dune-project` — since a worktree lives
nested inside the main checkout's directory tree, that's the main repo's root, not the
worktree's own `dune-project`. The main repo's root `dune` file excludes `.claude`
(`(dirs (:standard \ .claude))`), so the build resolves to `Error: No rule found for alias
.claude/worktrees/<name>/default` — an error easy to misread as "nothing to build" rather
than "wrong tree entirely." **Always pass `--root .` explicitly** when building/testing from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [march-language/march](https://github.com/march-language/march) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
