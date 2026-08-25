---
trigger: always_on
description: enables `version.adjustSemverBumpsForZeroMajorVersion` (default `true`, and this repo does
---

# AGENTS.md -- angular-typechecker

Agent-agnostic instructions for any AI coding agent working in this repository.
(Claude Code loads this via the `@AGENTS.md` reference at the top of `CLAUDE.md`.)

## Changing this file

**Any change to `AGENTS.md` MUST be code-reviewed.** This file governs how every AI agent
works in this repository, so an inaccurate, ambiguous, or unverified instruction propagates
silently into all future agent behavior. The review may be satisfied EITHER by an explicit
independent review before commit, OR by the mandatory `/gsd-code-review` step that runs
during phase execution (the `code_review_gate`), which reviews every source file changed in
the phase -- including this one. Either way, an `AGENTS.md` change is not "done" until a
code review has checked it for factual accuracy against the actual codebase and tooling,
clarity, and internal consistency, and every finding is resolved. (This rule exists because
a release-mechanics claim in this file was once wrong about the 0.x semver bump shift and
about `--skip-publish` semantics -- review is what caught both.)

## Conventional Commits drive the changelog and the released version

This repository releases `angular-typechecker` to npm with **`nx release`** configured
for **`version.conventionalCommits: true`** (see `nx.json` -> `release`). That means the
NEXT version number AND the generated changelog are computed **from the commit log** --
not chosen by hand. Every commit you write is release input. Follow these rules so the
release machinery behaves predictably.

### Commit message format

```
type(scope): short imperative description

optional body explaining what and why

optional footer (e.g. BREAKING CHANGE: ..., Refs: ...)
```

- `type` is required and lowercase. `scope` is optional but, when present, is rendered
  verbatim in the changelog (see the scope-hygiene rule below).
- A breaking change is marked EITHER by a `!` before the colon (`feat(core)!: ...`) OR by
  a `BREAKING CHANGE:` footer.

### How each type influences the version bump

`nx release` maps conventional-commit types to a SemVer bump. The version bump for a
release is the HIGHEST bump implied by any qualifying commit since the previous release
tag.

**IMPORTANT -- this repo is pre-1.0, so the bumps are shifted DOWN one level.** Nx 23
enables `version.adjustSemverBumpsForZeroMajorVersion` (default `true`, and this repo does
NOT override it; verified in nx 23.0.1 `config.js` and in `.planning/research/FOLLOWUP-FINDINGS.md`).
While the current version is `0.x`, every bump nx computes is lowered one step:
`major -> minor`, `minor -> patch`, `patch -> patch`. So the operative mapping right now is:

| Commit type                                                           | Standard (post-1.0) | EFFECT NOW (0.x, this repo) | In the changelog?                   |
| --------------------------------------------------------------------- | ------------------- | --------------------------- | ----------------------------------- |
| `feat`                                                                | minor               | **patch** (0.0.1 -> 0.0.2)  | Yes (Features)                      |
| `fix`                                                                 | patch               | patch (0.0.1 -> 0.0.2)      | Yes (Fixes)                         |
| `feat!` / `fix!` / `BREAKING CHANGE:`                                 | major               | **minor** (0.0.1 -> 0.1.0)  | Yes (Breaking Changes)              |
| `perf`                                                                | none                | none                        | Yes (Performance) -- shown, no bump |
| `docs`, `chore`, `refactor`, `test`, `build`, `ci`, `style`, `revert` | none                | none                        | No (hidden by default)              |

Two consequences to internalize:

- **While in 0.x, `feat` and `fix` both produce a patch bump** -- they are
  indistinguishable for the VERSION (they still land in different changelog sections).
  A breaking change is what cuts a new minor (e.g. `0.1.0`). This stays true until the
  first `1.0.0`, after which the standard column applies.
- **A release window that contains only no-bump types (`docs`/`chore`/`perf`/etc.)
  produces NO version bump** -- `nx release` reports no releasable change. Only `feat`,
  `fix`, and breaking changes move the version.

### Always confirm with a dry run

Because the 0.x adjustment surprises people, never assume the computed version. Preview it
with the UNIFIED command:

```
npx nx release --dry-run
```

The dry run prints BOTH the version nx will pick and the changelog it will write, sourced
from the commit log. Treat its output as the source of truth.

**Always use the unified `nx release` command, NOT the `nx release version` subcommand.**
Newly verified against nx 23.0.1: the `version` subcommand REJECTS the top-level
`release.git` block in `nx.json` and errors out (it tells you to move git options under
`release.version.git` / `release.changelog.git`). Only the unified `nx release` (and its
`--dry-run`) honors the top-level `release.git` block this repo relies on, so it is the only
command that previews and cuts with the correct `commit`/`tag`/`push` behavior. Use the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LayZeeDK/angular-typechecker](https://github.com/LayZeeDK/angular-typechecker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
