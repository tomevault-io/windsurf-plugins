---
trigger: always_on
description: This agent is responsible for running the changelog tool, which collects commit information,
---

# Changelog Tool

This agent is responsible for running the changelog tool, which collects commit information,
classifies commits via LLM, identifies external contributors, and renders a changelog for review.

## Two-part classification

**Part A — main pass.** Every commit is classified by the LLM — there are no content heuristics
(no size thresholds, no keyword-based pre-filtering). The LLM returns one of: `feature`,
`breaking-change`, `optimization`, `refactor`, `minor`, `docs`, `unclear`. Only the first three
(`breaking-change`, `feature`, `optimization`) are changelog-eligible: `to_changelog=true` is
never valid for `refactor`/`minor`/`docs`/`unclear`, and the renderer's main-section order constant
only contains those three anyway (defense in depth, both enforced independently). Per-category
prompt guidance: `breaking-change` produces a separate `migration_guide` field (see below);
`feature` describes the component and what was done; `optimization` focuses on concrete numbers
(%, ns, bytes, counts) when the commit provides them, describing what/why/measured-effect, without
inventing figures when none are given.

**Part B — creative external pass.** External commits the main pass did NOT promote to the
changelog ("external leftovers": `is_external` and not `(to_changelog is True and changelog_line)`)
are sent to a second, separate LLM prompt (`EXTERNAL_CONTRIB_SYSTEM_PROMPT`), diffless (title +
message + changed-file list only, no diff, no oversize handling needed). For each one the LLM
writes a short `changelog_line` and assigns a `group`: `Build` / `Documentation` / `Fixes` /
`Other`. A genuine new feature is instead promoted (`promote_to_feature=true`) straight into the
main changelog's Features section, with `to_changelog=True` and `classification=feature` set.
`Fixes` entries always carry a `component`, like main-pass entries. Results are persisted in a
**separate** state file `external_state.json` (never `llm_state.json`) so a SHA processed by both
passes never has one result clobber the other.

## Human overrides never get silently dropped: the `Forced` section

Two distinct mechanisms let a human insist a commit must reach the changelog regardless of what
the LLM thinks:
1. A "Changelog entry"/"Changelog line" section verbatim in the commit message (main-pass prompt
   override, see above) — forces `to_changelog=True`, but the LLM still independently picks a
   `classification`, which may land on `refactor`/`minor`/`docs`/`unclear`.
2. A reviewer checking an `excluded.md` item with a non-empty `changelog_line` during `refine`
   (force-include, no LLM call) — this only sets `to_changelog`/`changelog_line`; the commit's
   existing `classification` (whatever the main pass assigned it) is left untouched.

In both cases the commit ends up with `to_changelog=True` and a non-empty `changelog_line`, but
its `classification` may not be one of the three main-section categories. Rather than let the A1
restriction silently drop such a commit, [`render_changelog()`](changelog_tool/render/changelog.py:54)
detects this exact combination and renders it in a dedicated `Forced:` catch-all section
(grouped by `component`, same shape as the main sections), positioned right after Optimizations
and before the external Build/Documentation/Fixes/Other sections. A human decision to include a
commit is never lost, independent of how the LLM classified it.

## Oversize commits and `needs_attention`

A commit whose prompt (including its diff) doesn't fit the configured `max_user_prompt_length`
even in an otherwise-empty batch is isolated into its own single-commit batch and re-analyzed
**without its diff** (title/message/changed-file-list only). Such commits are flagged
`needs_attention=true` and, if included in the changelog, get a visible
`⚠ analyzed without diff — verify` marker. Nothing is silently dropped or truncated.

## Breaking changes: two distinct fields

For `breaking-change` commits the LLM produces two separate fields:
- `changelog_line`: describes the breaking change itself;
- `migration_guide`: a separate, actionable recipe for how to fix/migrate.

Both are rendered in `changelog.md` as two parts: the `changelog_line`, followed by a nested
`Migration:` sub-line carrying the `migration_guide`.

## Doxygen-style code references in `changelog_line` and `migration_guide`

Both the main pass ([`SYSTEM_PROMPT`](changelog_tool/llm/prompts.py)) and the external pass
([`EXTERNAL_CONTRIB_SYSTEM_PROMPT`](changelog_tool/llm/prompts.py)) instruct the LLM to format all
code references in `changelog_line` (and `migration_guide` for the main pass) using Doxygen syntax:

- **userver code entities** (classes, functions, methods, namespaces, enumerations, macros, etc.):
  `@ref` with the fully-qualified name, e.g. `@ref userver::components::ComponentBase`.
- **Files and directories**: `@ref` with the path, e.g. `@ref userver/core/component.hpp`.
- **Inline code fragments** (identifiers, variables, parameters, config values): backticks, e.g.
  `config.yaml`, `max_concurrent_requests`.
- **Multi-line code blocks**: triple backticks with the language.
- **Markdown links** `[text](url)` and HTML tags are also acceptable when more natural than a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [userver-framework/userver](https://github.com/userver-framework/userver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
