---
trigger: always_on
description: <!-- Link references — defined up front so agents see all targets in one scan -->
---

# Agent Instructions for InQL

<!-- Link references — defined up front so agents see all targets in one scan -->

[incan-repo]: https://github.com/dannys-code-corner/incan-programming-language
[incan-agents]: https://github.com/dannys-code-corner/incan-programming-language/blob/main/AGENTS.md
[incan-contributing]: https://github.com/dannys-code-corner/incan-programming-language/blob/main/CONTRIBUTING.md
[readme]: README.md
[contributing]: CONTRIBUTING.md
[architecture]: docs/architecture.md
[rfcs-index]: docs/rfcs/README.md
[rfc-template]: docs/rfcs/TEMPLATE.md
[writing-rfcs]: docs/contributing/writing_rfcs.md
[issue-templates]: .github/ISSUE_TEMPLATE/
[ci-workflow]: .github/workflows/ci.yml
[incan-toml]: incan.toml
[metadata-incn]: src/metadata.incn
[lib-incn]: src/lib.incn
[tests-dir]: tests/
[release-notes]: docs/release_notes/
[incan-docsite-loop]: https://github.com/dannys-code-corner/incan-programming-language/blob/main/workspaces/docs-site/docs/contributing/tutorials/book/08_docsite_contributor_loop.md
[incan-agents-docs-workflow]: https://github.com/dannys-code-corner/incan-programming-language/blob/main/AGENTS.md#docs-site-workflow-mkdocs-material

**InQL** is the typed **data logic plane** for [Incan][incan-repo]: relational queries, schema-aware table transformations, and streaming-shaped relational work, with a clear split from orchestration and engine-specific runtime in the authoring model. **This repository** holds the InQL **Incan library package** (`.incn` sources) and **normative RFCs** under `docs/rfcs/`. The **Incan compiler** (Rust) that implements parsing, checking, and lowering for InQL surfaces lives in the **Incan** repository.

This document guides AI agents and contributors working **in this repo**. For compiler implementation, Rust conventions, and the full toolchain pipeline, use **[Incan `AGENTS.md`][incan-agents]** and **[Incan `CONTRIBUTING.md`][incan-contributing]**.

> **CRITICAL — THE USER DECIDES WHAT IS RELEVANT.** Scope, PR boundaries, and which files belong on a branch are the **maintainer’s call**, not the agent’s. Do not dismiss work as “noise” or “hygiene” to remove or revert it. Ask when in doubt.
>
> **FORBIDDEN without explicit user approval that quotes the exact paths or commands:** anything that overwrites or deletes uncommitted work — including `git checkout -- <path>`, `git restore <path>`, `git clean`, `git reset --hard`, `stash drop`, or equivalent. If you believe something should be split, reverted, or omitted from a PR, **say so and ask**; do not run destructive git operations on your own initiative.

## Key references

| Topic                                               | Location                                                                                           |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Project overview                                    | [README.md][readme]                                                                                |
| Contributing (human workflow)                       | [CONTRIBUTING.md][contributing]                                                                    |
| Repo vs compiler placement                          | [docs/architecture.md][architecture]                                                               |
| Normative InQL design                               | [docs/rfcs/][rfcs-index]                                                                           |
| InQL RFC file template                              | [docs/rfcs/TEMPLATE.md][rfc-template]                                                              |
| Writing InQL RFCs (how-to)                          | [docs/contributing/writing_rfcs.md][writing-rfcs]                                                  |
| GitHub issue templates                              | [.github/ISSUE_TEMPLATE/][issue-templates]                                                         |
| Incan agent rules (Rust, compiler pipeline, skills) | [Incan `AGENTS.md`][incan-agents]                                                                  |
| Incan docs-site conventions                         | [Contributor loop][incan-docsite-loop] · [Markdown / MkDocs in AGENTS][incan-agents-docs-workflow] |

## What belongs where

| Change                                                                                        | Primary repo            |
| --------------------------------------------------------------------------------------------- | ----------------------- |
| InQL **RFC** text, `README`, `docs/*` (except normative rules in `__research__/`)             | **This repo**           |
| InQL **library** API and tests in `.incn`                                                     | **This repo**           |
| Lexer/parser/typechecker/lowering/**Rust** for InQL syntax, `query {}`, `DataSet` integration | [**Incan**][incan-repo] |

Normative behavior is defined in **`docs/rfcs/`**. If package code and an RFC disagree, treat it as a bug unless the RFC is explicitly superseded.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dannys-code-corner/InQL](https://github.com/dannys-code-corner/InQL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
