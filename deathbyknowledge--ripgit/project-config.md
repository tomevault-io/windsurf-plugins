---
trigger: always_on
description: This file is for AI coding agents. It covers the architecture, key design decisions, important constraints, and gotchas that are not obvious from reading the code in isolation.
---

# ripgit — agent context

This file is for AI coding agents. It covers the architecture, key design decisions, important constraints, and gotchas that are not obvious from reading the code in isolation.

## What this is

A self-hosted git server running on Cloudflare Durable Objects. Each repository is one DO with a SQLite database. The Worker entry point routes `/:owner/:repo/*` to the right DO by name. An optional TypeScript auth worker in `examples/github-oauth/` sits in front via Service Binding.

## Repository layout

```
src/
  lib.rs        Worker entry point. Routes /:owner/:repo/* to DO, /:owner/ to profile page.
                Also contains Actor/auth helpers, check_write_access, list_repos (KV query),
                handle_issue_action (issue/PR POST handler).
  pack.rs       Git pack file parser (two-pass: index + resolve). ResolveCache (Arc-based,
                budget-limited). Pack generator for upload-pack.
  git.rs        Smart HTTP protocol handlers: receive-pack (push) and upload-pack (fetch/clone).
                Pack size gate (50 MB). Calls into pack.rs and store.rs.
  store.rs      All SQLite operations: blob storage (xpatch delta compression), commit/tree
                storage, FTS index rebuild, commit graph build, ref management.
  api.rs        Read-only JSON API endpoints. parse_search_query() handles @prefix: syntax.
  diff.rs       Recursive tree diff, line-level unified diffs (similar crate), commit comparison.
  web.rs        Server-rendered HTML for all 9 web pages. layout() is the shared shell.
                Key helpers are pub(crate): layout(), html_escape(), html_response(),
                format_time(), render_markdown(), resolve_default_branch(), render_file_diff().
  schema.rs     Schema initialisation (runs in DO::new). All CREATE TABLE/INDEX statements.
  issues.rs     Issues and PR storage + merge logic. CRUD functions, three-way tree merge,
                BFS merge-base finder, git object serialization (SHA-1 via sha1_smol),
                parse_form() URL-decode utility.
  issues_web.rs Server-rendered HTML pages for issues and PRs. Uses web::layout() and helpers.

examples/github-oauth/
  src/index.ts  Auth worker: GitHub OAuth flow, session cookies, agent tokens, forwards to
                ripgit via Service Binding with X-Ripgit-Actor-* headers.
  src/types.ts  ActorProps, Env types.
  src/github.ts GitHub OAuth utilities.

scripts/
  push-test.sh  Incremental push script for large repos. Splits packs at 30 MB.
```

## Key constraints

**DO SQLite authorizer** — Cloudflare's DO SQLite blocks `PRAGMA` statements and other privileged operations. Use `sql.database_size()` (the `SqlStorage` method) for DB size — raw `PRAGMA page_count`/`PRAGMA page_size` will return `SQLITE_AUTH: not authorized`. All regular `SELECT`/`INSERT`/`UPDATE`/`DELETE` work fine.

**100 parameter limit** — DO SQLite allows at most 100 bound parameters per statement. Tree entry INSERTs are batched 25 per statement (4 params each = 100). Don't exceed this limit when adding batch operations.

**2 MB row limit** — DO SQLite rows can't exceed 2 MB. Large blobs that compress to over 2 MB are chunked across a `blob_chunks` overflow table and reassembled transparently. See `store::store_blob` and `store::read_blob`.

**128 MB DO memory limit** — the pack processing budget is designed around this. Pack body limit: 50 MB. ResolveCache budget: 20 MB. Pack generator buffers all objects before writing, so fetch/clone of large repos is a memory concern.

**Response::redirect requires absolute URLs** — `Response::redirect("/path", 302)` fails in Cloudflare Workers (Rust). Use `Response::error("", 302)` and set the `Location` header manually, or build a `new Response(null, { status: 302, headers: { Location: url } })` equivalent. There is a `unauthorized_401()` helper in `lib.rs` showing this pattern.

**DO names are permanent** — the DO is named `"{owner}/{repo}"`. This is its storage identity forever. Renaming a user or repo would orphan the DO. Don't add logic that renames DOs.

## Authentication model

The auth worker sets trusted headers before calling ripgit via Service Binding:

```
X-Ripgit-Actor-Name     GitHub username (or agent owner's username for agents)
X-Ripgit-Actor-Id       stable ID: "github:12345" or "agent:uuid"
X-Ripgit-Actor-Kind     "user" | "agent"
X-Ripgit-Actor-Scopes   comma-separated: "repo:read,repo:write,admin,..."
X-Ripgit-Actor-Owner    for agents: the owning user's actorId
```

`actor_from_request()` in `lib.rs` reads these headers. `X-Ripgit-Actor-Name` is what ripgit uses for ownership checks — it must equal the `owner` segment in the URL for writes to be allowed.

**Agent tokens** — when an agent token is created, `ownerActorName` (the owner's GitHub username) is stored alongside `actorName` (the token display name). `forwardToRipgit` in the auth worker sets `X-Ripgit-Actor-Name` to `ownerActorName` for agents, not `actorName`. This is critical — without it, `check_write_access` would compare the token name against the repo owner and always fail.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deathbyknowledge/ripgit](https://github.com/deathbyknowledge/ripgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
