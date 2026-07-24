---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Critical Rules (read first)

### Keep Commit Messages and CHANGELOG Entries Short

**One-sentence commit subject. One-sentence CHANGELOG bullet.** Commit bodies may include short bullets for distinct key changes — one line each, no prose paragraphs, no root-cause explanations unless the diff truly can't explain itself. Match the existing voice in `[Unreleased]` and recent `git log --oneline`. If your draft is longer than the surrounding entries, cut it before submitting.

**One feature = one CHANGELOG bullet, no matter how many commits built it.** A multi-commit feature (e.g. a whole panel landed over a dozen PRs) gets a single user-facing line, not one bullet per commit. Do NOT append a new bullet for every follow-up commit to the same feature — edit the existing bullet instead. The `[Unreleased]` section must read like a short release summary, not a commit log.

**Never put internal scaffolding in the CHANGELOG.** No table/column names, IPC channel names, service/store class names, env-var plumbing, migration registration, poll intervals, or "internal scaffolding for the upcoming X" bullets. The changelog answers "what can I now do / what's fixed" for a user — if a line names a symbol or a file, it's wrong. Internal-only changes (typecheck, tests, refactors, doc/agent tweaks, lint, dep bumps with no behavior change) get NO entry.

**At release time, condense — don't ship the dev-time bullets verbatim.** `[Unreleased]` accumulates verbose per-commit bullets during development. Before tagging, collapse them: merge a feature's scattered bullets into one line, drop scaffolding, squash near-duplicates. If the release notes are longer than the equivalent section in a recent shipped version, cut harder.

### Write and Run Tests for Behavioral Changes

**Any change to runtime behavior ships with a unit test** — a new test, or an extension of an existing one. Pure refactors already covered by tests, formatting, docs, and config-only changes are exempt. Before pushing, run the gate locally: `npm run typecheck && npm run test:prepush`. The repo's pre-push hook runs this automatically; it installs on `npm install` (or `npm run hooks:install`). Never push to `main` with a red suite — CI on `main` is a backstop, not the gate. For high-risk areas (sync/collab, main-process init, IPC, restart-to-verify bugs) the test comes **first** and must fail before the fix — see [end-to-end-verification.md](./.claude/rules/end-to-end-verification.md).

### Use @floating-ui/react for All Popover/Tooltip/Menu Positioning

See [floating-ui.md](./.claude/rules/floating-ui.md). Never manually calculate `position: fixed` coordinates — always use `@floating-ui/react` with `FloatingPortal`.

### No Dynamic Imports in Electron Main Process

**NEVER convert static imports to dynamic `await import()`** unless absolutely necessary (confirmed circular reference) AND the user has approved it. Dynamic imports cause `__ELECTRON_LOG__` double-registration crashes and side-effect timing issues. All MCP servers and services in `index.ts` use static top-level imports. The only allowed exception is `bootstrap.ts` importing `index.ts` (see [MAIN_PROCESS_INIT.md](./packages/electron/MAIN_PROCESS_INIT.md)).

### CollabV3 Data Isolation — DOs for Customer Data, D1 for Entity Management Only

**Never store customer, org, or team-sensitive data in the D1 shared database.** Customer data (team metadata, member roles, key envelopes, tracker items, documents, sessions) must live in Durable Objects where each entity gets its own isolated SQLite instance. D1 is only for cross-entity management lookups (e.g., git remote hash → org ID mapping). See `packages/collabv3/CLAUDE.md` for the full policy.

### Never Use Environment Variables as Implicit API Key Sources

**NEVER read API keys from `process.env` as a fallback for provider authentication.** API keys must come only from values the user explicitly configured in Nimbalyst settings (the electron-store `apiKeys` object or project-level overrides).

Past incident: a user had `ANTHROPIC_API_KEY` in a `.env` file for unrelated work. Nimbalyst silently picked it up via `process.env`, auto-persisted it, and billed the user's personal Anthropic account $100+ instead of their Nimbalyst subscription.

- No env fallbacks in `getApiKeyForProvider` — only `globalApiKeys[provider]` or project-level overrides
- No auto-import into the settings store
- Provider availability checks must only consider explicitly-stored keys

If you are tempted to add `|| process.env.SOME_API_KEY` as a convenience fallback, **stop**.

### Personal JWT vs Team JWT — Never Interchange Them

Stytch B2B gives a user a **different member id per org**. The **personal JWT** (`getPersonalSessionJwt()` / `personalUserId`) is for **personal sync ONLY** (the personal index room + session/prompt/draft/settings sync to the **mobile app**). The **team JWT** (`getSessionJwt()` / `getOrgScopedJwt(orgId)`) authorizes **ALL team collaboration** (tracker rooms, schema sync, document rooms, team room, project-access gate). Conflating them is this codebase's most-repeated sync bug.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimbalyst/nimbalyst](https://github.com/Nimbalyst/nimbalyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
