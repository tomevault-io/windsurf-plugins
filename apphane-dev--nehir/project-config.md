---
trigger: always_on
description: Instructions for AI agents working in this repository.
---

# AGENTS.md

Instructions for AI agents working in this repository.

## Shared agent assets

`.agents/` holds the skills, fence text, and gate hook scripts for this
repository; `.agents/README.md` documents them. Five gate skills activate on
their own at the moment their rule applies (tests, git mutations, success
claims, fix robustness, durable documents). Five workflows run only when invoked
as slash commands: `/nehir-bug-discovery`, `/nehir-retry-with-new-trace`,
`/nehir-delegate-lane`, `/nehir-finalize-change`, `/nehir-review-triage`. The
rules below remain authoritative; the skills are how they reach the point of
decision.

## Plans branch instructions

When working with planning documents in the dedicated plans-only worktree/branch,
read that worktree's `AGENTS.md` first and follow it. The plans branch has its
own document layout and stricter durable-doc rules (for example, no local
machine-specific paths in discovery/planning docs).

## Tests

Read `docs/TESTING.md` before adding, moving, or deleting tests. The hard
rules, in short:

- **Defer all test work to the latest stage.** Do not add, modify, rewrite,
  delete, move, compile, or run tests until the user either confirms the
  behavior works in their real repro or explicitly asks for test work. This is
  a sequencing rule, not a preservation rule: existing tests may encode the
  wrong contract and may be rewritten or deleted after the gate unlocks. The
  gate applies to new features, refactors, and bug fixes; a plan's test phase
  does not unlock it. Read `docs/TESTING.md` for the full policy.
- **New tests go into small per-behavior files.** The legacy monoliths
  (`AXEventHandlerTests.swift`, `NiriLayoutEngineTests.swift`, and the others
  listed in `docs/TESTING.md`) are frozen — never append tests to them.
- **Test hooks observe; they do not decide.** Do not add `ForTests`
  conditionals in `Sources/` that change a Nehir-owned decision (skip
  reconciliation, lifecycle, scheduling, fallback, or cleanup). Fake the OS
  boundary, not the algorithm.
- Run the suite with `mise run test`; keep it gated in CI.

## Temporary bug-tracing instrumentation

When bug-tracing code is built in a dedicated throwaway worktree and will be
removed before finalization, **never gate it** behind a feature flag, environment
variable, verbosity setting, or any other opt-in control. Emit the temporary
diagnostics unconditionally to the exact trace/capture sink the user will
inspect.

Before asking the user to reproduce the bug, verify that a known instrumentation
marker appears in the actual captured artifact. Seeing it in another logging
subsystem (for example, `os_log` when the user will provide a runtime-trace file)
does not count.

## Changesets

For user-visible changes, create a Changesets release-note fragment with:

```bash
mise run changeset <patch|minor|major|none> "User-facing summary"
```

Use `patch` for bug fixes, `minor` for new user-facing features, `major` for
breaking changes, and `none` for release-note-only changes. Add contributors
when needed with `--contributors handle1,handle2`. Issue reporters count as
contributors; include their GitHub handle when a change fixes a reported issue.
Mention the ticket/issue number in the changeset summary when one was involved.

Reference **only the nehir repo's own ticket number** (e.g. `Fixes #nnn`) in
changesets and commit messages — a bare `#nnn` auto-links to this repository on
GitHub, so it must point at the nehir issue, not upstream. **Do not** cite
upstream tickets (e.g. `OmniWM #nnn`, `BarutSRB/OmniWM#nnn`) in changesets or
commit messages; track upstream provenance in the nehir ticket body instead,
where it belongs.

In places where upstream tickets *are* cited (issue bodies, discovery and
planning documents), always use the full cross-repo form `BarutSRB/OmniWM#nnn`
— never `OmniWM #nnn` or bare `#nnn`. Bare `#nnn` means this repo's own
tracker; the two trackers share overlapping number ranges, so only the
`owner/repo#nnn` form is unambiguous.

Additional changeset rules:

- **Never guess the contributor or reporter.** Look up the actual GitHub handle
  from the nehir issue or PR before adding `--contributors`. Issue reporters
  count as contributors.
- **Choose the bump from the change, not the current version number.** Use
  `patch` for a user-facing fix, `minor` for a user-facing feature, `major` for
  a breaking change, and `none` only for release-note-only changes.
- **Write user-facing copy.** Describe the symptom and outcome in plain language;
  keep implementation detail and root-cause analysis in the ticket or discovery
  document.
- **Do not create duplicate fragments.** Before creating a changeset, check
  whether an existing fragment covers the same change. Update that fragment
  instead, preserving any contributor attribution already recorded there.

## Commit messages

Do not use Conventional Commits formatting (`fix:`, `feat:`, `chore:`, etc.).
Use concise plain-English commit subjects instead.

As with changesets, reference only the nehir repo's own ticket number (`#NN`)
never upstream tickets — upstream provenance lives in the ticket body.

## Git mutations require explicit per-action permission

Never mutate git state unless the user has explicitly approved that exact

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apphane-dev/nehir](https://github.com/apphane-dev/nehir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
