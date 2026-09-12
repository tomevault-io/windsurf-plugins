---
trigger: always_on
description: This repository's lint rules are documented in `planned-rules/`
---

# Implementing rules from `planned-rules/`

This repository's lint rules are documented in `planned-rules/`
before they are implemented. Each markdown file describes one
rule's intent, configuration knobs, examples, implementation
notes, and difficulty. Cross-cutting conventions (parser style,
lint-name namespacing, proc-macro-synthesis suppression) live in
`planned-rules/IMPLEMENTATION_CONVENTIONS.md`.

This guide tells you how to implement those rules and how to
keep the catalogue in sync as you go.

## Before you write code

Read these first, in this order:

1. **The rule's own file**, `planned-rules/<rule-name>.md`. It
   specifies the lint's identifier, configuration, the precise
   trigger predicate ("What to lint"), suggested fixes, and an
   estimated difficulty. Don't second-guess the design without
   first checking the rule file's existing examples and rationale
   — the design has usually been argued over already in the PR
   that produced the planning file.
2. **`planned-rules/README.md`** — the index of all rules, plus
   the
   [out-of-scope list](planned-rules/README.md#out-of-scope-cannot-be-linted-by-dylint)
   at the bottom. The index entry is the one-sentence summary;
   check that the rule you're implementing still says what you
   think it says.
3. **`planned-rules/IMPLEMENTATION_CONVENTIONS.md`** — applies
   to every rule. Among the cross-cutting conventions it covers:
   - **Parser style.** Non-trivial string scanners (URLs,
     emails, format templates, markdown spans, serde-attribute
     type literals) are written as parser-combinator-style
     `take_*` functions, not regex.
   - **Naming a lint after the anti-pattern.** A lint is named for
     the violation it fires on — never the fix, the remedy, or the
     preference — so it reads correctly under `#[deny(...)]` and
     `#[allow(...)]`. Follow Clippy's idiom, and mirror a Clippy
     lint's name only when the rule is a genuine *refinement* of it
     (not a contradiction or a complement). The name must claim no
     more than the trigger checks.
   - **Lint name namespacing.** Every lint registers under the
     `perfectionist` tool namespace via
     `rustc_session::declare_tool_lint!`. A per-rule `dylint.toml`
     table is *addressed* by its config key, so its header has to
     be the namespaced name, quoted —
     `["perfectionist::path_qualification_mismatch"]` — in a
     planning file's `## Configuration` fence exactly as in a
     consumer's `dylint.toml`; `#[allow(...)]` / `#[deny(...)]`
     likewise take the `perfectionist::`-qualified path rustc
     resolves. Where the rule is merely *named*, the unqualified
     `path_qualification_mismatch` is correct — in prose, and in a
     `[perfectionist]` `enable` / `disable` entry. See the
     conventions file for the per-context spelling.

If the rule is one of several that share a helper — markdown
exclusion, format-string parsing, URL discovery and unicode-width
measurement are examples, not the whole set — check whether the
helper already exists in the codebase before writing a new one.
Sibling-rule references in the planning files identify shared
infrastructure.

## One rule per file, one `Config` per rule

The catalogue is organised so that each rule has exactly one
source file at `src/rules/<rule_name>.rs` and exactly one `Config`
struct keyed by the rule's full namespaced name. The convention
has these consequences for the implementer:

1. **Before writing code, check whether the rule is actually one
   rule.** A planning file that bundles several independently-
   triggered checks under one banner is usually better
   implemented as several rules. If the sub-checks can be cleanly
   separated — distinct trigger predicates, disjoint
   configuration, no shared diagnostic — split the planning file
   into one rule per sub-check before you start. (Historical
   example: an early `single_letter_names` rule bundled four
   independently-configured checks for generics, `let` bindings,
   function parameters, and closure parameters; each lives in its
   own file today. See
   <https://github.com/KSXGitHub/perfectionist/pull/43>)

2. **When writing code, give each rule its own file and `Config`.**
   The file name matches the rule name (snake_case, no
   `perfectionist::` prefix). `CONFIG_KEY` is the full namespaced
   name (`perfectionist::<rule_name>`). The `Config` struct holds
   only the fields the rule actually reads — fields nominally
   "about" the rule but consumed by a different rule belong in
   that other rule's `Config`. If two rules genuinely share a
   helper function or type, factor it into `src/common.rs` (for
   trivial cross-rule utilities) or a dedicated
   crate-internal module rather than co-housing the rules in one
   file.

3. **When a rule grows past one screenful, split it into a
   directory module beside the flat `.rs` entry.** This crate
   forbids the `mod.rs` form (via `clippy::mod_module_files`,
   enabled in `Cargo.toml`), so the layout is `src/rules/<rule>.rs` next to
   `src/rules/<rule>/<concern>.rs`. The flat `.rs` entry keeps the
   `declare_tool_lint!` block, the `Register` impl, the
   `EarlyLintPass` / `LateLintPass` driver, and any process-wide
   state (`static PENDING_VIOLATIONS`, etc.). Common

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KSXGitHub/perfectionist](https://github.com/KSXGitHub/perfectionist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
