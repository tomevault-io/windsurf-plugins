---
trigger: always_on
description: This guide applies to work in **agentscope-go**, the Go implementation of
---

# Contributing with coding agents

This guide applies to work in **agentscope-go**, the Go implementation of
[AgentScope](https://github.com/agentscope-ai/agentscope). It is written for
coding agents and the people reviewing their contributions. The contributor
remains responsible for the accuracy, scope and quality of submitted work.

Read this file first, then the relevant parts of the architecture map in
[CLAUDE.md](CLAUDE.md). Use these sources for their respective responsibilities:

| Topic | Source |
|---|---|
| Contributor setup and PR process | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Architecture and implementation conventions | [CLAUDE.md](CLAUDE.md), the relevant source and tests |
| API stability and known limitations | [STABILITY.md](STABILITY.md) |
| Executable checks and tool configuration | [Makefile](Makefile), [CI workflow](.github/workflows/ci.yml), [lint configuration](.golangci.yml), [go.mod](go.mod) |
| PR checklist | [PR template](.github/PULL_REQUEST_TEMPLATE.md) |
| Community conduct | [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) |
| Private vulnerability reporting | [SECURITY.md](SECURITY.md) |
| Release history and user guides | [CHANGELOG.md](CHANGELOG.md), [docs/](docs/) |

When prose and implementation disagree, inspect the source and tests, identify
the discrepancy, and correct the affected documentation. Do not describe an
intended design as shipped behavior or weaken a check to make a change pass.

## Repository essentials

- The module is `github.com/agentscope-ai/agentscope-go/v2`; keep `/v2` in
  repository imports. Library code is under `pkg/agentscope/`, runnable
  examples under `examples/`.
- `go.mod` declares Go 1.25.0. Keep changes compatible with Go 1.25; a successful
  build on a newer toolchain alone does not establish minimum-version support.
- The project is Apache-2.0 licensed. Source files do not carry SPDX or license
  headers; do not add them.
- For capabilities already present in Python AgentScope, inspect the upstream
  implementation before designing the Go change. Record the relevant upstream
  link and explain deliberate differences; parity does not require copying
  Python APIs or dependencies.
- Check stability commitments in `STABILITY.md` before changing an exported API,
  configuration default, serialized format or observable behavior.

## Working on a contribution

1. **Establish the starting point.** Inspect the branch, worktree status and
   relevant history. Read the issue and existing discussion; check for related
   PRs before duplicating work. Preserve unrelated edits and commits. Never
   reset, clean or overwrite someone else's work to obtain a clean baseline.
2. **Verify the problem.** Trace the affected call path and reproduce the failure
   where practical. Record the commit, configuration and environment used.
   Distinguish source inspection, local fixture tests and live-provider results.
   Existing tests passing is not evidence that an untested behavior is correct.
3. **Keep the change reviewable.** Prefer a focused fix or feature with a clear
   outcome. Separate independent improvements and avoid unrelated formatting,
   dependency upgrades or refactors. Discuss substantial API or architectural
   choices before committing to an implementation.
4. **Test the behavior.** For changed or new code behavior, add a regression or
   acceptance test and observe the relevant failure before implementing it.
   Exported APIs need tests, including in packages without existing tests.
   Exercise boundaries and error paths; do not write assertions that merely
   repeat the implementation.
   Documentation-only edits need fact, link and example checks, not artificial
   behavior tests.
5. **Update the affected documentation.** Keep public API and behavior changes
   consistent with the relevant guides, README sections and architecture map.
   Update example listings when examples change. Keep factual corrections in
   `README.md` and `README.es-ES.md` in the same commit; follow `CONTRIBUTING.md`
   for translation of new feature sections. Add an `[Unreleased]` changelog entry
   for notable changes, or explain in the PR why one is unnecessary.
6. **Review and validate.** Complete the checks and independent evaluator review
   below. Review the final diff, including generated files and documentation,
   before staging only the intended changes.

Use a fork and a topic branch from current `main` for normal contributions
(`fix/...`, `feat/...` or `docs/...`), then open a PR against `main`. An explicit
maintainer instruction to commit or push directly to `main` changes the delivery
route, not the quality gate. Before pushing, fetch and check the destination;
incorporate intervening changes and repeat affected checks and review. Do not
force-push shared branches.

Local inspection, edits and validation can proceed within the requested scope.
Posting comments, opening PRs, pushing commits or contacting people requires
authorization for that action. An explicit request already supplies that
authorization; do not ask for it again. Repository access alone is not permission
to publish on a contributor's behalf.

## Issues, reviews and community communication

Follow `CODE_OF_CONDUCT.md`. Write in the language of the discussion where

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentscope-ai/agentscope-go](https://github.com/agentscope-ai/agentscope-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
