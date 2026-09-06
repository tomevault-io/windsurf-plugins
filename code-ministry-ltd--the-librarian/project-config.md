---
trigger: always_on
description: You're an AI agent working on this repo. It's part of
---

# AGENTS.md

You're an AI agent working on this repo. It's part of
[The Librarian](https://github.com/code-ministry-ltd/the-librarian) — a portable
memory + handoff layer for AI agents, open source, designed for
production use by people we'll never meet. Read this before your first
commit. Follow it on every change.

## 1. What this repo is

The Librarian itself — the MCP server, the durable-memory storage,
the cross-harness handoff surface, the Next.js admin dashboard, the
CLI, and the five harness integrations (Claude Code, Codex, Hermes,
OpenCode, Pi) under `integrations/`. pnpm monorepo on Node 22.
This is the canonical source of truth for the cross-harness slash
commands and the memory state model documented in §2. The former
standalone plugin repos
([Claude Code](https://github.com/JimJafar/the-librarian-claude-plugin),
[Codex](https://github.com/JimJafar/the-librarian-codex-plugin),
[Hermes](https://github.com/JimJafar/the-librarian-hermes-plugin),
[Pi](https://github.com/JimJafar/the-librarian-pi-extension)) are
being archived (rethink D14) — never add new work there.

## 2. House rules

### Be honest about what you ran

Never claim "tests pass" without running them. Never say a build works
because it "should." If a step was skipped, say so. If something is
unverified, label it. Your next session, and every contributor reading
your PR, inherits whatever you said — make sure it's true.

### Privacy beats convenience

This is The Librarian. Privacy is the product, not a feature. Private
mode (the in-conversation `[librarian:private=on|off]` marker, rethink
D11) stops all memory writes — never bypass it, never "just for
debugging." Bearer tokens go in headers, never in URLs or logs or
error messages. The private-mode contract is shared across the primer,
`docs/slash-commands.md`, and every integration's command templates —
change all of them or none.

### Fail-soft, never block the user's turn

A Librarian / network / parse failure must never throw out of a harness
hook, never block a prompt from reaching the model, never leak a stack
trace into the model's context. Log to the local sidecar, return the
no-op response, move on. The Librarian server can be down for an hour
and the user's day shouldn't notice.

### The cross-harness contracts are sacred

Everything now lives in this one repo (rethink D14 — the old five-repo
coordination rule is dead), but these contracts stay consistent across
every harness surface under `integrations/` and the server in the same
PR. Never invent new ones unilaterally:

- **The protocols.** The primer (`vault/primer.md`, default content in
  `packages/core/src/primer.ts`) is the **canonical definition** of the
  handoff / takeover / learn / private-mode protocols (rethink D9). The
  slash commands (`/handoff`, `/takeover`, `/learn`, and the local-only
  `/toggle-private`) are optional sugar over it — contract in
  [`docs/slash-commands.md`](./docs/slash-commands.md), templates in
  each integration's command files. Change the primer, the doc, and the
  templates together or not at all.
- **Memory state model:** memories are `active | proposed | archived`.
  The retired verbs (`confirm_memory`, `reject_memory`,
  `resolve_conflict`) are gone for good — proposals are accepted or
  rejected by the admin via the dashboard (tRPC), never by an agent
  MCP call.
- **Handoff document shape:** five required headings — `Start & intent`,
  `Journey`, `Current state`, `What's left`, `Open questions`. The
  schema refuses documents missing any of them.
- **The 7-verb MCP surface:** `recall`, `remember`, `flag_memory`,
  `store_handoff`, `list_handoffs`, `claim_handoff`,
  `search_references` — pinned by the tool-registry test and the
  healthcheck. The Hermes/Pi adapters mirror the schemas and
  descriptions verbatim (drift-guard tests pin them); a server-side
  tool change must update them in the same PR.

### Respect your consumers

Open source means people depend on what we ship. Treat that with care.

- **Every PR is a release. Bump the version and write the CHANGELOG
  entry in the same PR.** There is no `## [Unreleased]` section — file
  your notes under a new dated `## [X.Y.Z] — YYYY-MM-DD` heading at the
  top of `CHANGELOG.md`, add its `[X.Y.Z]:` compare-link at the bottom,
  and bump the root `package.json` to match (PATCH / MINOR / MAJOR per
  [`docs/release.md`](./docs/release.md)). Every merge to `main` bumps
  the version — even an internal refactor or doc fix takes a PATCH. The
  merge itself cuts the tag + GitHub release automatically
  (`.github/workflows/release.yml`); the `check:release` guard fails any
  PR that leaves an `[Unreleased]` section, forgets the bump, or desyncs
  the version from the top CHANGELOG entry.
- **Error messages teach.** "Invalid input" is not an error message.
  "Expected ISO-8601 timestamp, got '2026-13-99'" is. Assume the
  reader is new and tired.
- **README is the contract.** If it says one-liner install, that has
  to work on a fresh machine. If it claims a feature, the feature
  exists.

### Open a PR, never push to main

Always branch and PR. One change per PR. Conventional commit subject
(`<type>(<scope>): <subject>`) and a body that explains the *why*; the
diff explains the *what*. When an AI agent meaningfully contributed,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-ministry-ltd/the-librarian](https://github.com/code-ministry-ltd/the-librarian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
