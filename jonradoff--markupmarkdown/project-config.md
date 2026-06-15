---
trigger: always_on
description: Notes for anyone — human or AI — picking up this codebase.
---

# CLAUDE.md

Notes for anyone — human or AI — picking up this codebase.

The project is **markupmarkdown**: a Google-Docs-style commenting app for markdown files, with an MCP server so agents can join the same review loop as people. If you're forking it and want to know what's load-bearing, what's idiomatic for this project, and what'll bite you if you change it without thinking — that's what's below.

---

## Mental model

This is one Go binary + one React SPA + MongoDB Atlas. No queue, no S3, no Redis, no separate worker. The intentional design is "small enough to read on a Sunday." Don't add infrastructure unless the feature genuinely demands it.

The product is now best described as **"Google Docs for Markdown."** Originally it shipped as a commenting-only tool; the editor + GitHub round-trip (manual revisions, soft edit lock, 3-way merge, push as PR or direct commit) brought it to a full review-and-ship surface. When you're scoping a new feature, anchor on that mental model — what would Google Docs do here? — but always preserve the design principle: edits happen on the actual markdown text, not a visual mirror. The file in the repo stays the source of truth.

Three audiences share the same data model:

1. **Humans** — cookie session (`mm_session`), full account privileges.
2. **Scripts** — Personal Access Tokens (`mmk_…`), scope-restricted, used via REST.
3. **Agents** — same Personal Access Tokens, used via the MCP server at `/mcp`. Any Bearer-token request is treated as agent-authored (see `actorKindFor` in [backend/internal/api/comments.go](backend/internal/api/comments.go)).

All three paths route through the same access checks, rate limits, and validation. There is no "agent-only" code path that skips guards.

---

## Layout

```
backend/
├── cmd/markupmarkdown/main.go     # entrypoint (config load, store init, register routes)
├── internal/
│   ├── api/                       # HTTP handlers (gorilla/mux)
│   │   ├── api.go                 # API struct + route table — start here when you need to find a handler
│   │   ├── auth.go                # cookie sessions, GitHub OAuth, tokenInfo plumbing
│   │   ├── scope.go               # enforceScope helper (single source of truth for read/write/admin)
│   │   ├── validate.go            # shared comment/reply/anchor validation (REST + MCP share this)
│   │   ├── tokens.go              # personal-access-token CRUD + activity endpoint
│   │   ├── tokenlog.go            # sampled per-token activity logging (~1/min per action)
│   │   ├── comments.go            # comment + reply handlers, agent identity stamping/resolution
│   │   ├── documents.go           # doc CRUD, URL ingest, soft delete
│   │   ├── revisions.go           # AI revision preview (SSE) + accept
│   │   ├── mcpapi.go              # implements mcpserver.API — bridge from MCP into the rest of API
│   │   ├── plaintext_cache.go     # memoized goldmark plain-text extraction for MCP anchoring
│   │   ├── events.go              # SSE hub broadcasts ("comments-updated", "doc-updated", …)
│   │   ├── limits.go              # all rate-limit buckets + concurrency caps live here
│   │   ├── notifications.go       # @-mentions / reply notifications
│   │   ├── access.go              # checkDocAccess / checkCommentAccess (used by every protected handler)
│   │   ├── secrets.go             # Anthropic API key storage (AES-GCM via secrets.Vault)
│   │   └── static.go              # SPA handler with OG meta injection
│   ├── mcpserver/                 # Model Context Protocol server, mounted at /mcp
│   ├── store/                     # MongoDB collection accessors + queries (no business logic)
│   ├── models/                    # Go structs with bson + json tags (the source of truth for shapes)
│   ├── render/                    # goldmark wrappers (HTML + plain text + safe sanitization)
│   ├── ai/                        # Anthropic Messages API client for AI revision
│   ├── auth/                      # GitHub OAuth helpers
│   ├── secrets/                   # AES-GCM Vault for per-user secrets
│   ├── safefetch/                 # SSRF-guarded outbound HTTP (used by URL ingest)
│   ├── limits/                    # token-bucket + counter + per-key semaphore primitives
│   ├── httperr/                   # internal error sanitization (log full error, return {id, generic msg})
│   └── config/                    # YAML + env config loader
│
frontend/
├── src/
│   ├── App.tsx, main.tsx          # router + theme + root layout
│   ├── api.ts                     # typed REST client (every endpoint goes through here)
│   ├── types.ts                   # frontend types — keep in sync with backend models
│   ├── auth.tsx                   # auth context provider
│   ├── pages/                     # Home, Document, Trash
│   └── components/                # SPA components (TokensModal, CommentCard, ReviseModal, etc.)
│
skills/markupmarkdown/SKILL.md     # canonical agent integration guide
                                   # also embedded into the Go binary and served at /SKILL.md
fly.toml + Dockerfile              # single-process production deploy
```

---

## Conventions

### Build + lint + test checks (run before every commit)

```
cd backend  && go build ./... && golangci-lint run ./... && go test ./...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonradoff/markupmarkdown](https://github.com/jonradoff/markupmarkdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
