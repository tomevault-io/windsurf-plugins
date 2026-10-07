---
trigger: always_on
description: Agent Foundation is built on Harness, an embeddable agent execution foundation. Harness UI is its interactive playground for individuals and trusted small teams; Service is the managed-agent runtime. Both embed Harness and own distinct access, persistence, and execution lifecycles.
---

# Repository Guide

Agent Foundation is built on Harness, an embeddable agent execution foundation. Harness UI is its interactive playground for individuals and trusted small teams; Service is the managed-agent runtime. Both embed Harness and own distinct access, persistence, and execution lifecycles.

## Sources of Truth

- [CONTRIBUTING.md](CONTRIBUTING.md) owns contribution workflow, setup, and validation. Read the sections relevant to the requested change and handoff.
- [DEVELOPMENT.md](DEVELOPMENT.md) owns code quality principles and component engineering standards. Apply [Code Quality and Design](DEVELOPMENT.md#code-quality-and-design) to features, bug fixes, refactoring, and reviews; read the component rules and owning specifications relevant to the change.
- [spec/repository-model.md](spec/repository-model.md) owns repository structure and workflow boundaries. Read it before changing either.
- [spec/README.md](spec/README.md) leads to the accepted product and architecture contracts. Keep proposals, discussion, and progress in GitHub Issues; changes are reviewed through pull requests.
- `docs/` contains Markdown user documentation and its `meta.json` navigation; the Fumadocs site in `frontend/apps/a13n-docs` publishes it. [Documentation Changes](CONTRIBUTING.md#documentation-changes) owns authoring conventions.
- [MAINTAINERS.md](MAINTAINERS.md) owns semantic reviewer routing.

Read the relevant contribution and engineering sections before changing that surface. Reuse sections already read unless they changed. This guide and skills summarize operational rules; they do not replace the owning contracts. Do not turn personal preferences or tool-specific defaults into repository requirements without an explicit project decision.

Write canonical repository content in English; localized user documentation and UI translation resources use their target language, as required by [CONTRIBUTING.md](CONTRIBUTING.md#repository-language).

For local Service work, use the stable Make targets and discover checkout-specific ports with `make dev-status`; [dev/service/README.md](dev/service/README.md) owns lifecycle and data boundaries.

## Scope and Authorization

When drafting or updating Issue and PR bodies, follow [Writing Issues and Pull Requests](CONTRIBUTING.md#writing-issues-and-pull-requests). This writing guidance is mandatory for agents and discretionary for human contributors.

- Carry requested changes through implementation and relevant validation. Resolve routine choices from the request and repository evidence; ask only when missing information materially affects correctness, scope, or authorization. Existing authorization carries across follow-ups.
- An audit or review is read-only unless fixes are requested. Local editing does not itself authorize committing, pushing, GitHub writes, merging, deploying, or releasing. Each action must be covered by the request or established authorization; loading a skill grants none of these permissions.
- Preserve unrelated work and secrets. History rewrites, destructive cleanup, and changes to shared or deployed state require authorization covering the concrete operation and target.
- Unresolved product, architecture, security, compatibility, or scope decisions follow the Issue-to-PR workflow. Complete independent, authorized work while those decisions remain open. Routine corrections do not require a new Issue, and the workflow does not authorize posting one on the user's behalf.
- Explicit user instructions take precedence over skill guidelines, subject to system and developer instructions. Resolve apparent conflicts using the request and existing authorization. If work remains blocked by an applicable skill instruction, link its `SKILL.md`, quote the requirement, and explain the missing decision or authority while continuing independent authorized work.

Keep diffs focused and update affected contracts, implementation, tests, docs, and automation together. Report the outcome, changed files, validation, and material limitations concisely.

## Package and Release Boundaries

Python 3.13 and `packages/*` use `uv`; Rust crates live under `crates/`. Component source directories and distributions use canonical `a13n-` names, while Python imports replace hyphens with underscores (for example, `packages/a13n-stream-protocol`, `a13n-stream-protocol`, and `a13n_stream_protocol`). Service SDKs and the companion remote CLI belong to independent repositories. Optional checkouts under ignored `sdk/` are not inputs to this repository's workspaces or validation; follow each SDK repository's own guide for SDK work.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [converge-ai-labs/agent-foundation](https://github.com/converge-ai-labs/agent-foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
