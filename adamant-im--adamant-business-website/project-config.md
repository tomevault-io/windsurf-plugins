---
trigger: always_on
description: This document defines how AI agents must work in this repository.
---

# cryptofoundry Business Website: AI Agent Operating Manual

This document defines how AI agents must work in this repository.

## Mission

This repository hosts the cryptofoundry business website — the public-facing site for cryptofoundry.

**cryptofoundry** creates, maintains, and develops delightful software in the crypto space. It is an organization that provides such services to interested clients and receives compensation in cryptocurrency.

cryptofoundry is a **separate and independent organization** from the ADAMANT developer community. It actively participates in ADAMANT development, but it is not the same entity as the ADAMANT community or its governance.

Agent output must optimize for:

1. Clarity and accuracy of public-facing content
2. Security and privacy in forms, integrations, and deployment
3. Maintainability and contributor-friendly structure
4. Consistency with ADAMANT organization conventions where this repository follows them

If tradeoffs are required, preserve correctness and security first.

## Language Policy

- Developers may communicate with AI in any language
- All repository artifacts must be in English only
- Write all code, comments, commit messages, docs, and PR text in English

## Writing Style

- Use concise, operational wording over marketing language
- Write the organization name as **cryptofoundry** (all lowercase) in all repository artifacts — not `Cryptofoundry`, `CryptoFoundry`, or `CRYPTOFOUNDRY`
- In bullet and numbered lists, do not add a trailing period when an item contains one sentence
- If an item contains two or more sentences, end every sentence with a period

## Markdown Lint Rules for AI-Generated Docs

- For every Markdown list, keep one blank line before the list and one blank line after the list
- Always keep a blank line between a heading and the list that follows it to satisfy MD032 (`blanks-around-lists`)
- Use fenced code blocks with matching opening and closing fences and include a language tag when applicable
- Follow other best-practice Markdown rules used in this repository

## Product Context and Values

cryptofoundry builds and maintains self-hosted crypto software, bots, payments, and infrastructure for clients. It is independent from the ADAMANT developer community, while many of its engineers also contribute to the ADAMANT open-source ecosystem (since 2016).

When making decisions in this repository, agents should:

- Keep public content accurate, current, and easy to maintain
- Use the lowercase name **cryptofoundry** consistently
- Do not present cryptofoundry as identical to, or synonymous with, the ADAMANT community or ADAMANT governance
- Avoid introducing tracking, analytics, or hidden third-party data collection unless explicitly requested and justified
- Prefer simple, accessible implementations over premature complexity
- Preserve alignment with ADAMANT organization documentation and governance where applicable to shared workflows (issues, labels, PRs)

## Sources of Truth

Use these sources when implementing or reviewing changes:

- This repository: `README.md`, current code, and passing validation commands
- ADAMANT organization agent baseline: [adamant/AGENTS.md](https://github.com/Adamant-im/adamant/blob/dev/AGENTS.md)
- Org-wide issue and label governance: [Adamant-im/.github](https://github.com/Adamant-im/.github)
- Recommended issue title prefixes: [org discussion #5](https://github.com/orgs/Adamant-im/discussions/5)
- Recommended labels: [org discussion #1](https://github.com/orgs/Adamant-im/discussions/1)

If sources disagree:

1. Treat current repository behavior and passing validation as implementation truth
2. Do not silently ignore mismatches; document them and propose synchronized fixes

## Issue, Label, and PR Conventions

Follow organization-wide conventions:

- Governance repository: [Adamant-im/.github](https://github.com/Adamant-im/.github)
- Prefix guidance: [org discussion #5](https://github.com/orgs/Adamant-im/discussions/5)
- Label guidance: [org discussion #1](https://github.com/orgs/Adamant-im/discussions/1)

### Issue workflow

When creating an issue in this repository:

1. Search existing issues first to avoid duplicates
2. Use org issue forms (Bug / Feature request / Task) from org defaults where available
3. Start the title with one concise prefix
4. Apply labels from the org label catalog (`labels.json`) when they exist in this repository
5. Link related issues and PRs explicitly

### Recommended title prefixes for Issues

Use one or two prefixes maximum:

- `[Bug]` bug, crash, wrong behavior
- `[Feat]` new functionality
- `[Enhancement]` improvement of existing functionality
- `[Refactor]` internal refactoring without behavior change
- `[Docs]` documentation updates
- `[Test]` testing work
- `[Chore]` maintenance and routine technical tasks
- `[Task]` general task (including operations, release, or admin work)
- `[Composite]` multi-part task with sub-tasks
- `[UX/UI]` user experience or interface work
- `[Proposal]`, `[Idea]`, `[Discussion]` mostly for forum-level ideation

### Label policy

- `labels.json` in `Adamant-im/.github` is the source of truth for names, colors, and descriptions
- Use a minimal but informative set:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Adamant-im/adamant.business-website](https://github.com/Adamant-im/adamant.business-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
