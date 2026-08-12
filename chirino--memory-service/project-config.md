---
trigger: always_on
description: A memory service for AI agents that stores messages exchanged with LLMs and users, supporting conversation replay and forking.
---

# Memory Service

A memory service for AI agents that stores messages exchanged with LLMs and users, supporting conversation replay and forking.

**Self-Updating Knowledge:**
When you discover something meaningful about this project during your work—architecture patterns, naming conventions, gotchas, dependency quirks, correct/incorrect assumptions in existing docs — update `AGENTS.md` (or the relevant skill file) immediately so future sessions benefit without re-discovering it. Specifically:

- **Correct** any skill or doc content you find to be outdated or wrong.
- **Refine trigger criteria** in skill descriptions if a skill was loaded but wasn't relevant to the task—tighten its description so it activates more precisely.
- Keep updates concise and factual. Don't bloat files with obvious or generic information.
- Module specific knowlege should be placed into a `FACTS.md` in the top level directory of that module to avoid poluting AGENTS.md

## Key Concepts
- **Agent apps mediate all operations**: Agent apps are the primary consumers. They sit between end users and the memory service, mediating all interactions.
- **Agent API**: For agent apps - manage conversations, append entries, retrieve context for LLMs. Some agent APIs are designed to be safely exposed to frontend apps (e.g., SPAs) for features like listing conversations, semantic search, viewing messages, and forking.
- **Admin API**: For administrative operations and system management.
- **User access control**: Conversations are owned by users with read/write/manager/owner access levels.
- **Data stores**: PostgreSQL, MongoDB; Redis, Infinispan (caching); PGVector, Qdrant (vector search).
- **Porting Server To Go**: we are porting the ./memory-service java module to ./main.go
- **dev mode**: `task dev:memory-service` runs the go-based memory service using [air](https://github.com/air-verse/air) for hot reloading on port 8082 and it's dependencies get started with docker compose.
- **Local developer tooling policy**: `compose.yaml` and `task dev*` prioritize zero-configuration ease of use over production security hardening; keep demo credentials and avoid mandatory secret-generation setup. Embedded MCP has the same single-user desktop assumption.

## Quick Reference

**Build**: `./java/mvnw -f java/pom.xml` (Maven Wrapper)

**Essential commands**:
- `./java/mvnw -f java/pom.xml test` - run Java/Quarkus/Spring tests
- `./java/mvnw -f java/pom.xml compile` - compile Java modules (always run after Java changes)
- `task verify:python` - regenerate Python gRPC stubs and validate the LangChain package build/install (runs in Docker)
- `task dev:memory-service` - backend dev mode (:8082)
- `go test ./internal/bdd -run TestFeaturesPgKeycloak -count=1` - Go BDD runner for Postgres + Keycloak OIDC integration
- `cd memory-service-mcp && go build -o mcp-server .` - build the standalone MCP server binary

**Key paths**:
- `contracts/` - OpenAPI (`contracts/openapi/`) and protobuf (`contracts/protobuf/`) sources of truth
- `main.go` + `internal/` - core Go implementation
- `deploy/dev/air.toml` - local Air live-reload config for `task dev:memory-service`
- `deploy/docker/prometheus.yml` - local Docker Compose Prometheus scrape config
- `java/quarkus/examples/chat-quarkus/` - Demo chat app (Quarkus)
- `java/spring/examples/chat-spring/` - Demo chat app (Spring)
- `frontends/chat-frontend/` - Demo chat app frontend (React)
- `internal/sitebdd/` - Documentation test module (MDX `<TestScenario>/<CurlTest>` to Go/Cucumber pipeline)
- `internal/cmd/mcp/` - MCP server integrated into main binary (`./memory-service mcp`)
- `memory-service-mcp/` - Standalone MCP binary wrapper (build with `cd memory-service-mcp && go build -o mcp-server .`; `.mcp.json` uses `${PWD}` for portable paths)
- Current Go MCP implementation is HTTP/OpenAPI-based through `internal/generated/apiclient`; it does not use gRPC today, so embedded MCP designs only need an in-process HTTP path unless they explicitly add new gRPC consumers.
- MCP CLI split: main binary now uses explicit `./memory-service mcp remote` and `./memory-service mcp embedded` subcommands, while `memory-service-mcp` remains a single-command remote wrapper.

**API gotchas**:
- Conversation search endpoint is `/v1/conversations/search` (not `/v1/search`).
- Fork creation is implicit on first append to a new conversation ID using `forkedAtConversationId` + `forkedAtEntryId`; `POST /v1/conversations/{conversationId}/entries/{entryId}/fork` is obsolete.
- Entry listing uses `forks=all` to return entries from all branches in a fork tree (not `allForks=true`).
- Conversation archive semantics: user/admin conversation deletes are replaced by PATCH/update with synthetic `archived`; conversation list filters now use `archived=exclude|include|only`; archived conversations remain readable until eviction hard-deletes them.
- Historical enhancement docs `013`, `014`, `017`, `062`, `068`, `073`, and `090` still contain pre-094 delete/archive terminology in their body text; use `docs/enhancements/implemented/094-archive-operations.md`, current OpenAPI/proto contracts, and current site docs as the source of truth for archive behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chirino/memory-service](https://github.com/chirino/memory-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
