---
trigger: always_on
description: This document defines how AI agents should work in this repository.
---

# ADAMANT Forging Pool: AI Agent Operating Manual

This document defines how AI agents should work in this repository.

## Mission

ADAMANT Forging Pool calculates and transfers delegate voter rewards, serves a voter dashboard, stores local reward history, and sends admin notifications.

Agent output must optimize for:

- Correct reward and payout behavior
- Secret safety for pool credentials, notification tokens, and local config
- Reliability under node outages and partial API failures
- Decentralized node access and operator self-hostability
- Open-source maintainability and contributor clarity

If tradeoffs are required, preserve payout safety and secret safety first.

## Language Policy

- Developers may communicate with AI in any language
- All repository artifacts must be in English only
- Write code, comments, docs, issue text, PR text, commit messages, and release notes in English

## Writing Style

- Prefer concise, operational wording over marketing language
- In JSDoc param descriptions, bullet lists, and numbered lists, do not add a trailing period when an item contains one sentence
- If an item contains two or more sentences, end every sentence with a period
- Keep Markdown lists surrounded by blank lines and use fenced code blocks with language tags when practical

## JSDoc Policy

- Write JSDoc for functions you add or materially change
- Document each function's purpose, parameters, and return value when the return shape is not trivially obvious
- Add `@param` entries for all parameters and describe meaningful value semantics, not only types
- Reuse existing typedefs when available instead of inventing ad hoc inline object descriptions
- Keep JSDoc aligned with current behavior and update it in the same patch when behavior changes

## Sources of Truth

Use these sources when implementing or reviewing changes:

- Current repository code and passing tests
- `README.md`, `CONTRIBUTING.md`, and `config.default.jsonc`
- Root `package.json`, `server/package.json`, and `web/package.json`
- ADAMANT docs: <https://docs.adamant.im>
- ADAMANT Node and API schema references: <https://github.com/Adamant-im/adamant> and <https://schema.adamant.im>
- AIPs: <https://aips.adamant.im> and <https://github.com/Adamant-im/AIPs>
- Organization-wide issue, PR, and label governance: <https://github.com/Adamant-im/.github>
- Recommended issue title prefixes: <https://github.com/orgs/Adamant-im/discussions/5>
- Recommended labels for issues and discussions: <https://github.com/orgs/Adamant-im/discussions/1>

If sources disagree, treat current code and tests as the implementation truth for current behavior, then document the drift and propose a synchronized fix.

## GitHub Workflow

Follow ADAMANT organization conventions:

- Search existing issues before creating a new one
- Prefer organization issue forms when they are available
- Use concise issue title prefixes, with one or two prefixes maximum
- Use labels from the organization label catalog, not ad hoc names
- Target PRs to `dev`, not `master`
- Link related issues and PRs explicitly, using closing keywords when appropriate
- Use temporary Markdown files in `.ai-ignored/` for multi-line CLI input such as issue bodies, PR bodies, and commit messages

Issue prefixes:

- `[Bug]` for bugs, crashes, and unexpected behavior
- `[Feat]` for new functionality
- `[Enhancement]` for improvements without a brand new feature
- `[Refactor]` for refactoring without intended behavior changes
- `[Docs]` for documentation work
- `[Test]` for test additions or test improvements
- `[Chore]` for maintenance, tooling, dependencies, or CI work
- `[Task]` for general tasks
- `[Composite]` for multi-part work with sub-tasks
- `[UX/UI]` for interface and user-experience work
- `[Proposal]`, `[Idea]`, and `[Discussion]` for idea-level topics that are often better suited for Discussions than Issues

Label policy:

- `labels.json` in `Adamant-im/.github` is the source of truth for label names, casing, descriptions, and colors
- Keep default GitHub labels lowercase, such as `bug`, `enhancement`, and `documentation`
- Keep custom organization labels capitalized when the org uses capitalized names, such as `Security`, `Privacy`, `Task`, `Composite task`, and `UX/UI`
- For most issues, use a small but informative set: one type label, one or more domain labels, and an optional priority label when justified
- Do not invent legacy workflow labels for tracking state when GitHub Projects already owns that workflow

PR conventions:

- Use the organization PR template sections when preparing PR text
- Use Conventional Commit style for PR titles, for example `Docs: Update AI instructions`
- Do not use issue-style square-bracket prefixes in PR titles
- Keep PR title type aligned with the nature of the change, such as `Docs:`, `Fix:`, `Feat:`, `Refactor:`, `Test:`, or `Chore:`
- Include testing or verification steps and mention meaningful risk areas

## System Map

- Root package: repository metadata, setup hooks, dependency installation, and web build orchestration
- `server/`: backend runtime, ADAMANT API access, config reading and validation, reward distribution, payouts, cron scheduling, local storage, notifications, and HTTP API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FjgarciaMac/pool](https://github.com/FjgarciaMac/pool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
