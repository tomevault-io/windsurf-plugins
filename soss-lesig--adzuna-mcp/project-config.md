---
trigger: always_on
description: This is the adzuna-mcp project. A Model Context Protocol server wrapping the Adzuna job search API, built for use with AI assistants during a UK job search. Second in a series of job board MCP servers (after reed-mcp), designed to be combined later in an aggregator layer.
---

# CLAUDE.md

## Part A: Standing Instructions

This is the adzuna-mcp project. A Model Context Protocol server wrapping the Adzuna job search API, built for use with AI assistants during a UK job search. Second in a series of job board MCP servers (after reed-mcp), designed to be combined later in an aggregator layer.

The architectural reasoning behind every decision in this project lives in the Obsidian vault at `/Users/drewbs/dev/drewbs-vault/mayus-vault/projects/adzuna-mcp/DECISIONS.md`. The full Adzuna API reference is at `docs/api-reference.md` in the same vault. Read both before making any structural changes. If a decision needs revisiting, add a new dated entry to DECISIONS.md rather than mutating an existing one.

### Ground Rules

- **Explain before executing.** Walk through planned changes before writing anything. One step at a time.
- **Never commit or push without explicit sign-off.** Stage changes, show `git status`, wait for confirmation.
- **British English.** No long em dashes in any written content.
- **Conventional commits.** `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`. Short messages, detail in PR descriptions later.
- **Branch naming:** `feature/kebab-case`, `fix/kebab-case`.
- **Earn complexity.** No TypeScript, no monorepo, no tests yet. The scope is deliberately tight. If you think something needs adding, check `ROADMAP.md` in the vault for whether it's already been deferred or explicitly declined.
- **Vault-currency checkpoint.** Before starting any numbered build step, confirm the vault is current. If a decision was made, a lesson was learned, or something unexpected happened since the last vault write, write it to `DECISIONS.md` and/or `docs/session-notes.md` before proceeding.

### Language and Tooling

- **JavaScript, ESM.** `"type": "module"` in package.json. No TypeScript.
- **Node 20+** with the `--env-file=.env` flag for env loading. Never use the `dotenv` package.
- **MCP SDK:** `@modelcontextprotocol/sdk` (the official one). Dual transport: stdio and Streamable HTTP.
- **HTTP transport uses Express** for the minimal routing around `/mcp`.

### Code Documentation (enforced)

These four rules are enforced on every source file. The goal is that a reader who has never seen this codebase can understand any file on first read.

1. **JSDoc every exported function and class.** Document parameters with types, the return value, and any errors thrown. Single-line JSDoc is fine for one-arg helpers; multi-line for anything non-trivial.
2. **Per-file header comment at the top of every source file.** One paragraph, maximum five lines. Explains what the file is, why it exists, and how it relates to the rest of the codebase. Written in prose, not a bulleted list.
3. **Inline comments only for non-obvious choices.** Explain the *why*, not the *what*.
4. **Decisions made during coding go in `DECISIONS.md` in the vault.** If an implementation choice emerges that was not planned, add a new dated entry.

### Code Walkthrough (enforced)

The vault file `docs/code-walkthrough.md` is a prose tour guide to the codebase. **Update rule: the walkthrough is updated in the same commit as the code change it describes, or in a chained commit immediately after.** Never at session close. Walkthrough and code must not drift.

### Key File Locations

- **Repo:** `/Users/drewbs/dev/projects/repos/adzuna-mcp`
- **Vault:** `/Users/drewbs/dev/drewbs-vault/mayus-vault/projects/adzuna-mcp/`
  - `README.md` - project overview
  - `DECISIONS.md` - architectural decisions log (read this for the "why")
  - `ROADMAP.md` - build queue and deferred items
  - `docs/api-reference.md` - full Adzuna API endpoint and parameter mapping
  - `docs/session-notes.md` - live session log (to be created)
  - `docs/code-walkthrough.md` - prose tour of the codebase (to be created)
- **Sibling project:** `/Users/drewbs/dev/projects/repos/reed-mcp` (reference for patterns)

### Adzuna API Key Differences from Reed

- **Two env vars:** `ADZUNA_APP_ID` and `ADZUNA_APP_KEY` (not a single key).
- **Auth via query params**, not Basic Auth header. Client appends `app_id` and `app_key` to every request URL.
- **Distance in kilometres**, not miles.
- **No separate job detail endpoint.** Search returns all available fields. Description is a snippet only.
- **Pagination via page number in URL path** (`/search/1`, `/search/2`), not offset.
- **Location is a nested object** with `display_name` and `area` array, not a flat string.
- **Documented rate limits:** 25/min, 250/day, 1000/week, 2500/month.
- **Parameter names use snake_case** (`results_per_page`, `sort_by`, `what_exclude`). We translate to camelCase in our Zod schemas for consistency with reed-mcp's developer experience.

### Session Close Checklist

1. Append a dated entry to `docs/session-notes.md` in the vault.
2. Tick off completed items in `ROADMAP.md`.
3. If any architectural decision was made, add a new entry to `DECISIONS.md`.
4. Verify `docs/code-walkthrough.md` reflects the current state of the code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soss-lesig/adzuna-mcp](https://github.com/soss-lesig/adzuna-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
