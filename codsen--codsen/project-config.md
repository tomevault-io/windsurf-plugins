---
trigger: always_on
description: Both Codex and Claude work in this repository, and either may pick up any task.
---

# Repository guidance

## Write instructions for every agent, not for one

Both Codex and Claude work in this repository, and either may pick up any task.
Keep instructions, conventions, and workflow notes in this file or another file
that every agent reads, and phrase them for an unnamed agent.

- Do not create tool-specific guidance files such as `CLAUDE.md`, and do not
  move a shared rule into one tool's configuration.
- Do not phrase a shared rule as if one assistant were its only reader, and do
  not rely on behaviour that only one tool's harness provides.
- When a rule genuinely applies to one tool alone, name that tool inside the
  shared file rather than splitting the guidance across files.

## Ask before branching

The maintainers are the authors here and normally commit straight to `main`, so
a `main` checkout is where work is expected to happen, not a hazard to route
around. Do not create a branch on your own initiative, and do not treat a
default-branch checkout as a reason to. If a change looks like it warrants its
own branch, stop and ask first, then follow the answer.

A harness whose own defaults tell it to branch before committing to a default
branch does not have grounds to override this. Those defaults assume a shared
repository with several committers; this one has a single maintainer, and a
commit landing directly on `main` is the expected outcome, not a risk to
mitigate.

## Push only when you are asked to

Committing and pushing are separate decisions here. Commit when the work is
ready and the maintainer has asked for a commit, but leave the push to them
unless they ask for it too. A push runs the full CI pipeline, which is slow, and
the maintainer usually lets several commits accumulate and pushes them as one
batch of finished work. A stack of unpushed commits on `main` is a normal state,
not a mistake to point out or correct.

This also rules out the indirect routes to the same effect: do not push a branch
and do not open a pull request in order to get CI to run against a change.

## Dependency updates are manual

Dependabot is disabled. `.github/dependabot.yml` was deleted on 2026-08-19,
while the npm release lane was still failing on its first end-to-end runs,
because every automated pull request spent CI minutes on a pipeline that had not
yet proven itself.

- Update dependencies by hand until then, as this repository did before
  Dependabot: bump the version, run the root verification commands, and open one
  pull request for the batch.
- Re-enable Dependabot only when the maintainer says continuous integration and
  the release lane are stable and they are happy to resume it. Restoring
  `.github/dependabot.yml` is the entire change.
- `verify.yml` keeps its `DEPENDABOT_BASE_SHA` engine guard. That guard is inert
  without Dependabot pull requests and is left in place deliberately, so
  re-enabling needs no workflow edit.

## Architecture principles

`.agents/ARCHITECTURE_PRINCIPLES.md` defines the repository's direction for
typed source, generated declarations, monorepo scale, distribution formats,
purity boundaries, and the JavaScript/TypeScript toolchain seam. Read it before
making architectural, build, packaging, declaration-generation, or test-pipeline
decisions. Treat the principles as constraints, and verify the current
implementation before describing it as compliant.

## Repository improvement backlog

When present, the local `secret-plans/20260814-codex.md` records audited
architectural, CI, release, testing, and package-quality improvements. Read it
when the task concerns this backlog or the user asks to continue the monorepo
review plan. Treat it as a backlog, not as authority to expand the current task.
Revalidate each finding against the current tree before implementation, and
update the item's status and evidence when work is completed.

When a user references an item by a `REV-*` identifier, search the files in
`secret-plans/` for that identifier before planning or making changes.

## Keep review details out of commit messages

Never mention a `REV-*` identifier directly in a commit message. Keep references
to review work deliberately vague: for example, use `Resolve various review
findings` rather than `Resolve review findings REV-004 through REV-007`. This
keeps the private review-tracking mechanism out of Git history.

## Agent planning documents

Put new agent-authored reviews, investigations, and implementation plans in the
repository-root `secret-plans/` directory instead of using
`.agents/CODE_REVIEW_*.md`. Name Codex files `YYYYMMDD-codex.md`, for example
`secret-plans/20260814-codex.md`, and use the corresponding lowercase agent
name for other agents. When available, follow
`secret-plans/20260814-claude.md` as the style reference: use ordinary Markdown
with a descriptive title and natural sections. Do not use all-caps filenames;
the directory and document title already identify the file's purpose.

## Monorepo package kinds

The root workspace uses npm and Turbo. Most published packages live under
`packages/`, and the generated aggregate package lives under `data/`. Most
package-level boilerplate is maintained by `ops/lect`; do not assume that a file
inside a package is hand-maintained just because it is checked into Git.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codsen/codsen](https://github.com/codsen/codsen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
