---
trigger: always_on
description: This document helps AI agents and automated systems interact with the Reflect repo safely and effectively. It summarizes setup, workflows, CI parity, testing, directories, and environment variables.
---

### Purpose

This document helps AI agents and automated systems interact with the Reflect repo safely and effectively. It summarizes setup, workflows, CI parity, testing, directories, and environment variables.

### What is Reflect

Reflect is a modern note‑taking tool with a TypeScript codebase. This repo contains Reflect V2, a rewrite of the original Reflect code-base to make it offline-first, markdown backed, and open source.

### Product Principles

Drawn from the product docs — read these for deeper context:
[V1 Overview](docs/reflect-v1-overview.md) · [V2 Product Vision](docs/reflect-v2-product-vision.md) · [V2 Grounding Brief](docs/reflect-v2-grounding-brief.md) · [Indexing Strategy](docs/reflect-v2-indexing-strategy.md) · [Sync Strategy](docs/reflect-v2-sync-strategy.md)


- **Daily notes first.** The app opens to today's note. All capture flows into the daily note by default.
- **Association over hierarchy.** `[[Wiki Links]]` replace folders. The note graph is the organizing model; there are no folders.
- **Markdown is the source of truth.** Notes are `.md` files (`daily/YYYY-MM-DD.md`, `notes/`). SQLite under `.reflect/` is a rebuildable projection of the notes — with one durable exception: the `chat_*` tables hold AI chat history, which is not derivable from markdown. Index wipes and rebuilds must leave them untouched.
- **No Reflect-hosted APIs.** LLM calls go directly to user-approved providers (OpenAI, Anthropic, etc.). Sync goes to GitHub/iCloud/Git. Never proxy through Reflect infrastructure.
- **BYOK AI.** AI features use user-supplied keys. Never assume Reflect operates AI infrastructure.
- **`private: true` is a hard block.** Notes with this frontmatter flag must never have their content sent to any external service — AI, transcription, or otherwise. Enforce at every call site.
- **Keyboard-native UX.** Every core workflow must be reachable from the keyboard. This is product identity, not polish.
- **Minimal UI.** Do less, and do it well. Don't add surfaces that compete with the editor.
- **Secrets in the OS keychain.** API keys and credentials never go in markdown, Git, or `.reflect/`.
- **Portable data.** Full export (JSON, markdown, HTML) must work from day one.
- **No Electron.** Desktop shell is Tauri.
- **MIT open-source core.** Write as if the code is public and will be critiqued.

### Agent workflow

- **Verify before answering.** When answering factual questions about what the code
  does, read the relevant source first and trace behavior to the final output. If
  you have not verified something, say so instead of guessing.
- **Plan proportionally.** For non-trivial, ambiguous, or high-risk changes, form a
  short plan before editing and ask for sign-off when the direction affects public
  APIs, migrations, release behavior, or broad UX. Simple localized fixes can
  proceed once the relevant context is understood.
- **Use a dedicated worktree or branch.** Check `git status` before editing and
  before staging. Preserve unrelated user changes; ask before publishing if the
  worktree is dirty, the PR scope is ambiguous, or staging would include changes
  you did not make.
- **Prefer the clean design.** Optimize for the correct open-source shape rather
  than the smallest diff. Avoid compatibility shims, dual paths, or legacy behavior
  unless the product/release context requires them.
- **Verify locally.** Run typecheck, lint, and targeted tests for the code you
  touched. If a required check cannot run, report the reason and the residual risk.
- **Publish completed work.** When a requested implementation is complete and
  verified, create or use an appropriate branch, commit the intended changes, push,
  open a normal ready-for-review PR, and wait for CI/checks, Bugbot, review
  comments, merge conflicts, and other blockers to settle.

### Development workflow

Development happens on `master` (the only long-lived branch); branch from it and
target it with PRs. release-please keeps a beta and a stable Release PR open side by
side; merging one publishes that channel. Between stable releases the version carries
a prerelease suffix (`0.7.0-beta.3`), which the release pipeline publishes as GitHub
pre-releases. See [docs/macos-distribution.md](docs/macos-distribution.md).

PR titles must be conventional commits (`feat:` / `fix:` / `chore:` …, enforced by
CI). The title becomes the squash-commit message, drives the release-please version
bump, and — for `feat`/`fix` — is the user-facing changelog entry, so write it
as behavior, not implementation. Do not use `feat!:` or `BREAKING CHANGE:` footers;
see [CONTRIBUTING.md](CONTRIBUTING.md).

The app version lives solely in `apps/desktop/package.json`, maintained by
release-please through Release PRs. Never hand-edit that version, the changelogs
(`apps/desktop/CHANGELOG.md`, `apps/desktop/CHANGELOG.beta.md`), or the manifests
under `.github/release-please/`.

1. Make your changes
2. Run typecheck (`pnpm typecheck`)
3. Run lint (`pnpm lint`) — fix any errors; `pnpm fix` auto-fixes where possible
4. Run specific tests for your changes (`pnpm test --run path/to/test`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-reflect/reflect-open](https://github.com/team-reflect/reflect-open) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
