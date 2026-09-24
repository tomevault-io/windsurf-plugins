---
trigger: always_on
description: Rust workspace, edition 2024, stable toolchain. Crates live in `crates/` and are
---

# Aster

Rust workspace, edition 2024, stable toolchain. Crates live in `crates/` and are
prefixed `aster-` (the exception is `symbol-extractor`). The `crates/aster-cli`
crate is the binary; everything else is a library.

Human-facing setup and PR etiquette live in `CONTRIBUTING.md`. This file is the
working agreement for agents editing this repo.

## Global rules

- No useless comments in code.
- Match the project's existing style and patterns before writing anything new.
- No em dashes (—) in any writing.
- Never add `Co-Authored-By: Claude/Cursor` trailers to commits or PRs, and
  strip them if found.
- Use Conventional Commits for every message.

## User-facing copy

Every string a user sees (TUI cells, panel cards, error boxes, status lines)
is written in plain, relatable language, never harness vocabulary. "Turns",
"verdict", "judge", "budget exhausted", "rate limited", "context window" are
internal words; users get "tries", "check", "too many requests", "stopped
after 20 tries". Errors always say what happened and what to do next in one
plain sentence, with the raw provider detail demoted below. Protocol field
names, config keys, env vars, and docs stay technical; only rendered copy is
covered by this rule.

## Rust conventions

- Read the crate you are touching and follow its existing patterns before
  introducing a new one.
- Inline variables into `format!` braces: `format!("{name}")`, not
  `format!("{}", name)`.
- Collapse nested `if` statements.
- Prefer method references over closures: `.map(str::trim)`, not
  `.map(|s| s.trim())`.
- Avoid bare `bool` and ambiguous `Option` parameters that force callers to
  write `foo(false)` or `bar(None)`. Prefer enums, newtypes, or named methods so
  the callsite reads on its own. `aster_policy::Mode` and
  `PermissionModeArg` are the shape to copy.
- Make `match` statements exhaustive. Avoid wildcard arms over enums we own, so
  adding a variant produces a compile error instead of silent fallthrough.
- New traits get a doc comment explaining their role and what implementors are
  expected to do. `McpInvoker` in `crates/aster-mcp/src/lib.rs` is the example.
- Prefer private modules with an explicit `pub` crate API. Keep the exported
  surface as small as the callers need.
- Do not create helper functions that are called exactly once.
- Keep the core dependency-light. No vector DB, no external services in the
  review path. A new heavy dependency needs a written justification in the PR.

## Module size

- Target modules under 500 LoC, excluding tests.
- Past roughly 800 LoC, add new functionality in a new module rather than
  extending the file, unless there is a documented reason not to.
- This applies hardest to the files that already attract unrelated changes:
  - `crates/aster-cli/src/chat.rs`
  - `crates/aster-cli/src/skills.rs`
  - `crates/aster-harness/src/lib.rs`
  - `crates/aster-cli/src/review.rs`
  - `crates/aster-mcp/src/lib.rs`
- When extracting from a large module, move the related tests and type docs with
  the code so the invariants stay next to what owns them.
- `chat.rs` is orchestration. Prefer new modules under `crates/aster-cli/src/`
  over new standalone functions there.

## Resist adding to `aster-cli`

`aster-cli` is the largest crate, so it is always the path of least resistance:
the imports are there, the helpers are there, and nothing forces you out. That is
how it got large.

Before adding a new concept, feature, or API to `aster-cli`, consider whether:

- An existing library crate is the right home. Tool execution, context
  assembly, session state, and policy decisions are library concerns, not CLI
  concerns.
- It is time to add a crate to the workspace. Refactor as needed to make that
  happen.

`aster-cli` should be argument parsing, terminal I/O, and wiring. If a change
would still be correct with no terminal attached, it probably belongs in a
library crate.

The same applies in review: push back on changes that grow `aster-cli` without
needing to.

## Model-visible context

Aster builds a context that is sent to the model on every inference request.
Every regression here is expensive and hard to see, so these are invariants, not
preferences.

1. **No history rewrite.** Context is built up incrementally. The only
   permitted rewrite is the summarize-and-replace path in `compact_if_needed`,
   and it must record a `record_summary` event so the transcript stays honest
   about what was dropped.
2. **Avoid changes that cause cache misses.** Anything prepended or inserted
   near the head of the request invalidates the provider's prompt cache for the
   whole conversation. Append instead. If a change moves or reorders early
   context, say so in the PR.
3. **Nothing unbounded.** Everything injected into context has a bounded size
   and a hard cap. Existing caps live as `const` at the top of their module:
   `MAX_TOOL_RESULT_CHARS`, `MAX_SEARCH_HITS`, `MAX_LIST_ENTRIES`,
   `COMPACT_BUDGET_CHARS` in `chat.rs`; `inventory_budget_tokens` in
   `aster-mcp`. New injections declare their own.
4. **No single item over ~10K tokens.**
5. **Flag any new individual item that can exceed ~1K tokens** in the PR
   description. Those need a human to look at them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zfinix/aster](https://github.com/Zfinix/aster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
