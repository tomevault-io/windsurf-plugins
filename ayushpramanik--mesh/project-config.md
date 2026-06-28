---
trigger: always_on
description: > Agent-native version control. Git was built for humans committing a few times a day.
---

# Mesh

> Agent-native version control. Git was built for humans committing a few times a day.
> Mesh is built for AI agents committing thousands of times a day, in parallel.

---

## What Mesh is

Mesh is an open-source orchestration layer that sits between AI coding agents and git hosts
(GitHub, GitLab). It solves three problems that don't exist in human-paced development but
become critical at agent scale:

1. **Workspace explosion** — agents each clone the full repo. Mesh replaces N clones with one
   shared git object store and N lightweight worktrees (~10ms to spin up vs ~60s clone).

2. **PR throughput ceiling** — agents saturate CI queues and GitHub rate limits. Mesh queues,
   batches, and schedules PRs using merge trains so throughput is bounded by compute, not tooling.

3. **Silent semantic conflicts** — two agents edit different files that import each other. Git
   sees no conflict; Mesh's AST-level conflict predictor catches it before work even starts.

The long-term vision is bigger: Mesh is the foundation for a new VCS primitive designed
around parallelism as the default, structured machine-readable provenance, and continuous
merge instead of the PR-gated model.

---

## Architecture

```
agents / IDE plugins / CLI
         │
    gRPC + protobuf          ← typed agent protocol, language-agnostic
         │
   mesh daemon (Go)          ← Unix socket, manages all local state
    ┌────┴─────────────────────────────────────────────┐
    │  workspace manager     conflict predictor         │
    │  (go-git worktrees)    (Tree-sitter AST graphs)   │
    │                                                   │
    │  PR orchestrator       GitHub/GitLab client       │
    │  (queue + merge train) (go-github + GraphQL)      │
    └────┬─────────────────────────────────────────────┘
         │
    SQLite (local)           ← workspace state, PR queue, agent registry
    git object store         ← single shared clone on disk
         │
    Postgres (hosted)        ← multi-repo, multi-user SaaS tier
```

---

## Repository layout

```
mesh/
├── cmd/
│   ├── mesh/               # CLI entrypoint (cobra)
│   └── meshd/              # Daemon entrypoint
├── internal/
│   ├── workspace/          # Worktree lifecycle (create, gc, snapshot)
│   ├── conflict/           # Tree-sitter AST diff + dependency graph
│   ├── queue/              # PR queue, merge train scheduler
│   ├── git/                # go-git wrapper (thin, keep side-effect-free)
│   ├── github/             # go-github + GraphQL client
│   ├── daemon/             # gRPC server, request handlers
│   └── store/              # SQLite schema + queries (sqlc generated)
├── proto/
│   └── mesh/v1/            # .proto definitions for agent protocol
├── dashboard/              # React + TypeScript web UI
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── pages/
│   └── package.json
├── docs/
├── scripts/
│   ├── dev-setup.sh
│   └── gen-proto.sh
├── CLAUDE.md               # this file
├── go.mod
└── Makefile
```

---

## Tech stack

| Layer | Technology | Why |
|---|---|---|
| Daemon + CLI | Go 1.25+ | Single static binary, fast startup, goroutine concurrency, large OSS contributor pool |
| Git operations | go-git | Pure Go, no CGO, no system git version dependency |
| Agent protocol | gRPC + protobuf | Typed, versioned, streaming, generated clients for Python/TS/Go |
| AST conflict detection | Tree-sitter (Go bindings) | 50+ languages, incremental, fast, used by VS Code/Neovim |
| Local state | SQLite via sqlc | Zero-config, embedded, sqlc generates type-safe Go from SQL |
| GitHub/GitLab API | go-github + GraphQL | go-github for REST, GraphQL for batch operations and merge queue |
| Dashboard | React 18 + TypeScript | — |
| Dashboard styling | Tailwind CSS | — |
| Live updates | Server-Sent Events | Dashboard is read-heavy; SSE is enough, no WebSocket complexity |
| Hosted DB | Postgres | SQLite → Postgres is a driver swap when sqlc targets both |

---

## Core concepts

### Workspace
An ephemeral, isolated environment for one agent to complete one task. Backed by a
`git worktree` — shares the `.git` object store, has its own working tree and branch.
Created in ~10ms. Automatically cleaned up when the task finishes or errors.

Key invariant: **a workspace is always on exactly one branch, owned by exactly one agent**.
Never share workspaces between agents.

### Intent
Before an agent starts work, it registers an **intent**: a structured declaration of which
files, packages, or symbols it expects to modify. The conflict predictor cross-references all
active intents and either clears the new one, delays it, or surfaces a warning.

Intents live in SQLite and are keyed by workspace ID. They are best-effort predictions, not
locks. The system will still catch actual conflicts at AST diff time.

### PR queue
Agents do not call the GitHub API directly. They submit to the Mesh PR queue. The queue:

- Deduplicates (same branch submitted twice → no-op)
- Orders by priority (user-set) then submission time
- Groups non-conflicting PRs into merge train batches
- Rate-shapes submissions to avoid GitHub abuse detection
- Retries on transient errors with exponential backoff


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AyushPramanik/Mesh](https://github.com/AyushPramanik/Mesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
