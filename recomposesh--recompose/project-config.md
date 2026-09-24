---
trigger: always_on
description: - Research current industry best practices for the topic (web search), independent of this codebase.
---

# recompose project rules

## Before starting any work

- Research current industry best practices for the topic (web search), independent of this codebase.
- If a clear standard path exists, bring the codebase into conformance with it first, then do the work.
- When a request sounds like a capability of a platform or tool already in use (GitHub, Electron, pnpm, and so on), search for the built-in or off-the-shelf solution first. Write a custom implementation only after that search comes up empty.

## Git workflow

- `main` stays protected. Never commit to it, locally or remotely.
- Every job (feature, fix, docs, config, skills) gets its own worktree and branch, and lands through a PR. One job = one branch.
- **Worktrees live at `.claude/worktrees/<name>`, never beside the repository.** A sibling directory reads as a second project to anyone browsing, and `EnterWorktree` refuses to switch into a path outside that directory, which strands a subagent that needs to reach it. The repository's own tooling already assumes the convention: `.vale.ini` and `cspell.json` both exclude `.claude/worktrees`.

## CodeRabbit reviews

- Before acting on a finding, compare it with official docs and the actual code, and never apply it without checking first.
- Addressed a finding → reply on its thread naming the fixing commit, then resolve the thread immediately.
- Rejecting or deferring a finding → reply with the reasoning and leave the thread unresolved so CodeRabbit can respond; resolve only when the exchange settles.
- No conversation stays unresolved at the end of the day: when CodeRabbit acknowledges an exchange but leaves the thread open, resolve it yourself.

## Skills

- Use the `ponytail` skill for everything, since every task starts by invoking it.
- Every session starts by invoking `karpathy-guidelines` before any other work.

## Prose style

- **Never use an em dash.** Don't patch one out with a colon: rewrite the sentence so it reads as if it never had one.
- All authored markdown passes Vale: Microsoft base style with rules promoted to error, plus the house rules in `.vale/styles/recompose/` (`docs/adr/0025-vale-prose-gate.md`). New vocabulary lands in the committed accept list through the PR diff. Implementation plans under `docs/superpowers/plans/` sit outside both prose gates (Vale and cspell) as internal execution artifacts.

## Comments

- **Never write code comments.** Code must explain itself through naming and structure.
- The only exception: a constraint or invariant that code genuinely can't express (for example, "Electron requires this before app.ready"). If in doubt, don't write it.
- Content decides whether a `@summary` docstring belongs, never visibility. Write one on any declaration, exported or not, when it records a constraint, an invariant, a rejected alternative, or a cross-cutting reason the code can't state. Never write one that narrates what the code does, restates the name, or describes the diff. The ban on inline explanations holds either way (record 0115).

## Architecture decisions

- Every technical decision becomes an Architecture Decision Record (ADR) under `docs/adr/`, written via the `architecture-decision-records` skill. No undocumented decisions.

## `README.md`

- Whenever README.md needs creating or updating, use the `create-readme` skill.

## Commits

- Every commit goes through the `caveman-commit` skill. No exceptions.

## Feature development

- Every feature starts with `/feature-cycle <description>`. The `feature-cycle` skill classifies the tier as trivial, standard, or full, the maintainer confirms it, and the phases run from there.
- The skill calls `superpowers` as its executor library: `superpowers:subagent-driven-development` runs the Test-Driven Development (TDD) implementation and `superpowers:brainstorming` supplies the brainstorm discipline.
- Trivial work (config tweaks, docs, single-file fixes) keeps its escape hatch: the `trivial` tier exits the pipeline, so just do it.
- Dispatch independent work in parallel by default. Running one worker after another needs a named blocker, and only three count: one worker reads what another produces, two workers own the same file, or one worker inspects what another writes. Every dispatch names the files it owns and says that the others run on disjoint files.

## Test-driven and behavior-driven development

- Follow @.claude/rules/tdd-bdd.md, which lays out inside-out (Detroit/classicist) TDD with Behavior-Driven Development (BDD)-style behavior specs. Test code changes if and only if behavior changes.

## Testing

- Write tests at every layer of the test pyramid: unit, integration, e2e.
- **Never run Vitest from the repository root.** There is no root test project, and the root config refuses with the commands that work (`docs/adr/0162-vitest-refuses-to-run-from-the-repository-root.md`). Run `pnpm test` for every package, `pnpm --filter <package> exec vitest run` for one package, and add `--project <name>` for one of the desktop projects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recomposesh/recompose](https://github.com/recomposesh/recompose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
