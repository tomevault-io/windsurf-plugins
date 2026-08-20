---
trigger: always_on
description: These instructions apply to the entire repository.
---

# AGENTS.md — Pioneer DDJ-FLX4 Mixxx Mapping

## Scope

These instructions apply to the entire repository.

This repository contains the actively developed Pioneer DDJ-FLX4 controller
mapping for Mixxx. The long-term goal is to contribute a reviewed variant as a
complete replacement for the DDJ-FLX4 mapping currently shipped with Mixxx.

The repository is **not** merely a staging area for an upstream patch. `main`
remains the canonical standalone/development version of the mapping.

## Human ownership and AI-assistance policy

This project may use Codex or other AI tools as an assistant for analysis,
refactoring, code generation, documentation, and review.

For work intended for Mixxx upstream, follow Mixxx's current AI-agent policy:

- Do not autonomously create, update, reopen, or submit pull requests.
- Do not autonomously create, update, or reopen issues, bug reports, or feature
  requests.
- Do not autonomously reply to Mixxx pull-request review comments.
- Do not autonomously `git commit` or `git push` changes intended for upstream.
- The human contributor must review and deliberately approve the final changes.
- Functional controller changes must be tested by a human with the real
  DDJ-FLX4 and a real DJ setup before upstream submission.
- Text written autonomously by an AI agent for an upstream PR, commit-message
  body, code comment, or documentation must follow the disclaimer requirements
  in the current Mixxx `AGENTS.md`.

When working only in this repository, edit files when explicitly asked, but do
not commit, push, create branches, or publish anything unless the human user
explicitly requests that action.

## Repository / branch model

### `main`

`main` is the complete standalone and development version.

Changes belong in `main` when they are generally beneficial and do not exist
only to satisfy upstream packaging or policy. Examples:

- ES7 / Mixxx runtime compatibility fixes
- bug fixes
- lint fixes
- removal of dead code
- timer / connection cleanup
- clearer internal structure
- safer state handling
- concise, useful comments
- removal of temporary development notes
- documentation corrections
- refactors that intentionally preserve behavior

Do not deliberately reduce functionality in `main` merely to make a future
Mixxx pull request smaller.

### `mixxx-pr`

A persistent `mixxx-pr` branch will be created from a cleaned-up `main`.

It contains only changes that are specifically required or deliberately chosen
for the upstream Mixxx version, for example:

- upstream metadata
- upstream file naming / paths
- removal of standalone branding such as `(Custom)`
- upstream-specific defaults
- upstream-specific packaging decisions
- behavior changes requested or justified specifically for inclusion in Mixxx

General fixes should normally be made in `main` first and then merged into
`mixxx-pr`.

Prefer:

```text
main -> fix/refactor -> merge into mixxx-pr
```

Do not let `mixxx-pr` become an independently maintained second implementation.

### Mixxx fork

The final Mixxx pull-request branch is prepared separately in the user's fork of
`mixxxdj/mixxx`.

The intended controller replacement paths are:

```text
res/controllers/Pioneer-DDJ-FLX4.midi.xml
res/controllers/Pioneer-DDJ-FLX4-script.js
```

The current Mixxx 2.6 mapping already uses those paths and the function prefix
`PioneerDDJFLX4`.

The preferred final Mixxx commit should represent the finished mapping
replacement, not the entire development history of this standalone repository.

## Upstream target

Unless explicitly changed by the human contributor, evaluate upstream
compatibility against the Mixxx `2.6` branch.

Do not silently switch the target to `main`, `2.5`, or another branch.

When a Mixxx rule differs between branches, the target branch is authoritative
for technical checks that affect that branch.

## JavaScript runtime compatibility

Mixxx controller mappings run in `QJSEngine`.

For Mixxx 2.4+ the documented scripting target is ES7, excluding ES6 modules.

The Mixxx `2.6` ESLint configuration explicitly uses:

```text
ecmaVersion: 7
sourceType: script
```

Therefore:

- Do not introduce JavaScript syntax newer than ES7.
- Do not use ES modules (`import`, `export`).
- Optional chaining (`?.`) is not allowed.
- Nullish coalescing (`??`) is not allowed.
- Check all new syntax against ES7 rather than assuming that a modern Node.js or
  browser feature is available.
- Do not rely on Node.js APIs or browser DOM APIs.
- Use Mixxx-provided controller APIs only where they are actually available in
  the targeted Mixxx version.

Compatibility changes that preserve intended behavior belong in `main`.

## ESLint and JavaScript style

The Mixxx target branch's `eslint.config.cjs` is the primary machine-readable
source of truth for JavaScript linting.

For Mixxx 2.6 it includes, among other rules:

- `eqeqeq`: error
- function-expression style: error
- unused variables: error
- atomic-update checks: error
- 4-space indentation: warning
- `curly`: warning
- Unix line endings
- semicolons
- double-quote preference
- `no-var`: warning
- `prefer-const`: warning

Important: older Mixxx wiki text contains some JavaScript style guidance that
predates the current ESLint configuration. If an old prose convention conflicts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ElHanko/mixxx-ddj-flx4-mapping](https://github.com/ElHanko/mixxx-ddj-flx4-mapping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
