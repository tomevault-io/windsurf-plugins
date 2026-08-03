---
trigger: always_on
description: Go MCP server for the *Agentic Wiki* (Karpathy pattern + OKF). The agent never touches files directly: it operates via MCP tools; the server enforces the invariants. → `docs/overview.md`
---

# AGENTS.md — Cartographer

Go MCP server for the *Agentic Wiki* (Karpathy pattern + OKF). The agent never touches files directly: it operates via MCP tools; the server enforces the invariants. → `docs/overview.md`

**Where to look for what** (not duplicated here):

| Need | Source of truth |
|---|---|
| Full doc map + reading paths | `docs/index.md` (read that first, then only the relevant pages) |
| Project status and milestones | `docs/roadmap.md` |
| Why behind a choice (AD/D entries) | `docs/decisions.md` — Grep for the entry (`## D<n>`), never read in full |
| List of MCP tools | `docs/control-plane.md` §API MCP |
| `CARTOGRAPHER_*` env vars, YAML config, deploy | `docs/deployment.md` |
| Client subcommands / TUI | `docs/configurator.md` |
| Test strategy and pre-release checklist | `docs/testing.md` |
| Go conventions | `docs/conventions.md` |

## Commands

```
make build           # → bin/cartographer
make test            # go test ./...
make vet             # go vet ./...
make fmt             # gofmt -w .
make run             # stdio with demo KB
make run-http        # HTTP on :8080 with demo KB
make smoke           # build + quick stdio test
make smoke-http      # operator-level HTTP smoke test (creates temp KBs via curl)
make docker          # build Docker image
make clean           # removes bin/ and demo-kb/
```

```bash
./bin/cartographer serve --kb /path/to/kb --init      # local stdio server
./bin/cartographer serve --config config.yaml         # HTTP/stdio server (YAML, see config.example.yaml)
./bin/cartographer connect|disconnect|status|sync     # multi-provider client (HTTP)
./bin/cartographer                                    # TUI dashboard (if TTY), otherwise usage
```

Protocol: JSON-RPC 2.0. Stdio = newline-delimited. HTTP = POST /mcp. Logs on stderr.

## Code map

```
cmd/cartographer/             # main.go (subcommand dispatch), serve.go, bootstrap.go,
                             # agents.go/connect.go/disconnect.go/status.go/sync.go/clientsync.go (multi-provider client),
                             # connectform.go (bubbletea form shared by CLI/TUI for connect),
                             # tui.go (interactive dashboard, cartographer with no arguments)
internal/config/             # Server YAML Config: Load/FromEnv/ApplyFlags (flag>env>YAML>default)
internal/okf/                # ConceptID, frontmatter (stdlib-only YAML parser), content-hash
internal/kb/                 # kb.go (Open/Init/Read/Write/Validate/Walk), graph.go, gate.go,
                             # gitsync.go (per-KB lock, CommitOp, SyncIn/SyncOut), conflicts.go (conflict registry)
internal/search/             # in-memory inverted index
internal/sqlindex/           # persisted SQLite index: FTS5 trigram + embedding cache
internal/lint/               # Run(kb, scope, scopeNeighbors)
internal/gitx/               # git wrapper (commit, rebase, push, stash, conflicts)
internal/mcpserver/          # server.go (Run stdio, HTTPHandler, MultiKB), protocol.go, httpserver.go,
                             # tools.go (registry + RegisterKBTools with Deps), tools_{read,search,write,
                             # governance,skill,sync}.go, liveindex.go, gitwrap.go (lock+commit+sync on writes),
                             # readonly.go (per-tool r/rw classification)
internal/audit/              # append-only JSONL hash-chain + Ed25519 signature
internal/auth/               # TokenStore, Middleware, scopes, RBAC
internal/embed/              # Embedder interface, OllamaEmbedder, vector Store
internal/skill/               # LoadSkill, Catalog, Validate (SKILL.md)
internal/sops/               # Decrypt, ResolveRefs, EnvForSkill
internal/configurator/       # multi-provider adapter (HTTP only): Claude Code, Codex, Kiro, OpenCode
internal/provisioning/       # Manifest, Lock/LockFile, Diff, Apply, MergeArtifacts
internal/agents/             # Detect() agents installed on the machine (claude/opencode/codex/kiro)
internal/clientconfig/       # .cartographer.yaml (server_url, connected agents, etc.)
internal/client/             # minimal MCPClient (JSON-RPC 2.0 over HTTP) for the client subcommands
```

## Code navigation

graphify graph in `graphify-out/` (not versioned). Structural questions about the code (where does X live, who calls Y, what does Z depend on) are answered by querying **the graph first** — `graphify query "<question>" --budget <n>`, `graphify path "<A>" "<B>"`, `graphify explain "<node>"` — then reading only the indicated `file:line` locations. No broad Grep/Explore if the graph can answer (for an exact, already-known symbol a targeted grep is equivalent); the `/graphify` skill is only for building/updating the graph, never for queries. Specific rules:

- the graph gives the **where/how it connects**; the **why** lives in `docs/decisions.md` (Grep the entry), current status in `docs/` (map in `docs/index.md`);
- in mandates to `dev`/OpenCode include the `file:line` pointers already derived from the graph: the subagent should not re-explore from scratch;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeppeTemp/cartographer](https://github.com/BeppeTemp/cartographer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
