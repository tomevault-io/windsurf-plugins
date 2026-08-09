---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**qnop** — "Qualified Notes on Papers": an enterprise **document review** system. Reviewers (individual users or teams) mark up lines/regions of documents, comment, and run a coordinated review workflow (comments accepted/rejected → new document versions → finalized when no open annotations remain). Open-core: an AGPL Community edition plus commercial add-ons (e.g. AI features) and a possible SaaS.

**Scope of supported formats.** Community reviews **PDF and DOCX**. Everything else — Markdown (`qnop-ee#20`), images, and whatever follows — is **Enterprise** scope and lives in the private `qnop-enterprise` repository. Markdown was Community scope until 2026-08-05; see the amendment in ADR-0010 for what changed and why the seams did not. Design the ingest/anchoring/rendering seams so a new format is an added implementation, not a core rewrite.

Read `docs/ARCHITECTURE.md` and `docs/adr/` first — they hold the binding decisions and rationale.

## Working rules (binding — see ADR-0008)

1. **Issue first** — every change starts with a GitHub issue.
2. **Never commit or push to `main`** — it is integration-only/protected (ruleset deferred until the repo is public or the org is on Team — see ADR-0018; convention is binding now).
3. **Feature branch → PR** — branch names follow Conventional Branch (rule 9): `feat/…`, `fix/…`, `chore/…`, `hotfix/…`, `release/…`; the PR references its issue.
4. **Claude attribution everywhere** — commits get a `Co-Authored-By: Claude <noreply@anthropic.com>` trailer; issues and PRs get an attribution line in the body: `🤖 Co-Author: Claude (Opus 4.x) via Claude Code`.
5. **Record important architecture decisions as ADRs** in `docs/adr/` (template in its README). Add the ADR in the same PR as the change.
6. **Sign the CLA** (`CLA.md`, ADR-0016) — enforced on PRs by the CLA-Assistant workflow; maintainers/bots are allowlisted.
7. **English everywhere in the project** — issues, PR descriptions, commit messages, documentation, ADRs, and code comments are written in English. This holds even when the working chat language is German: chat may be German, but anything that lands in the repo, an issue, or a PR is English.
8. **Clean copyright on every source file** — the copyright + SPDX header from the root `license-header.txt` (`Copyright (c) 2026-present devtank42 GmbH`, AGPL-3.0-only). Enforced for Java via Spotless; see ADR-0019. Run `./gradlew spotlessApply` before committing.
9. **Conventional Commits & Conventional Branches — always** — every commit message, without exception, follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) (`<type>: <subject>`, types `feat|fix|refactor|docs|test|chore|perf|ci|build`). Branch names follow [Conventional Branch](https://conventionalbranch.org/): `<type>/<kebab-description>`, type ∈ `{feat, fix, hotfix, release, chore}` (`feat`/`fix` are the accepted short forms of `feature`/`bugfix`), lowercase + hyphens only, optional issue number — e.g. `feat/issue-123-new-login`. `main`/`master`/`develop` carry no prefix.
10. **Push to GitHub only with explicit approval** — never `git push` (any branch) on your own initiative. Committing locally is fine; pushing waits until the user approves it in the current conversation. A user request that itself includes pushing (e.g. "commit + push + pr" or an invoked command that says to push) counts as that approval.

Commits are signed off (`git commit -s`, DCO). See `CONTRIBUTING.md`.

## Current state

The Spring Boot server **boots and runs** (PostgreSQL + Liquibase + JPA, ADR-0020) with two shipped verticals:

**Identity & administration** (epic #7): local login with JWT access + rotating refresh tokens and revocation (ADR-0026), OIDC/OAuth2 providers, self-registration, email verification and password reset, auth rate limiting (ADR-0027), users & teams, application settings (ADR-0025), mail templates, branding upload with SVG sanitization (ADR-0028; assets as Postgres `bytea`, ADR-0024), profile avatars (ADR-0031), public profiles with slugs (#473/#486), ShedLock scheduling (ADR-0029) and optimistic concurrency control (ADR-0030).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qnophq/qnop](https://github.com/qnophq/qnop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
