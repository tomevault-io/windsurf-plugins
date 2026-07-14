---
trigger: always_on
description: > **R**etrieval **T**ool **F**or **M**anuals.
---

# RTFM

> **R**etrieval **T**ool **F**or **M**anuals.
> (The team knows what it really stands for. The README does not.)

A local, per-developer documentation retrieval tool. It indexes a folder of
documentation — Confluence MHTML exports (`.doc`), Word `.docx`, plain
Markdown (`.md`), PDF, Excel (`.xlsx`), CSV, and draw.io diagrams — into a
local OpenSearch instance and exposes it to any
MCP-capable LLM client (Claude Code, Claude Desktop, IDE integrations) over a
stdio MCP server. Instead of manually attaching docs to a chat, a developer asks
their LLM and it retrieves the relevant passages itself.

The tool answers two kinds of question well:

- **Technical lookup** — "What's the endpoint to GET this resource?" (lexical;
  the user's words appear verbatim in the docs)
- **Conceptual** — "What does Bundle mean?" (semantic; the answer may never
  repeat the user's phrasing)

---

## 1. Architecture

Three independent components. Keep them decoupled — this separation is a
deliberate decision, not an accident (see §2).

```
        ┌─────────────────────────────────────────────────────────┐
        │  docs/  (MHTML/.doc, .docx, .md — mostly static)          │
        └───────────────────────────┬─────────────────────────────┘
                                     │  read
                                     ▼
        ┌─────────────────────────────────────────────────────────┐
        │  rtfm  (console CLI, long-lived in watch mode)            │
        │   ├─ convert:  MHTML/docx/md → HTML → ReverseMarkdown     │
        │   ├─ chunk:    heading-aware, breadcrumb + overlap        │
        │   └─ index:    bulk upsert / delete into OpenSearch       │
        └───────────────────────────┬─────────────────────────────┘
                                     │  HTTP (bulk, delete-by-query)
                                     ▼
        ┌─────────────────────────────────────────────────────────┐
        │  OpenSearch  (single-node, Docker, persistent volume)     │
        │   index: rtfm-docs  (keyword + text + knn_vector fields)  │
        └───────────────────────────┬─────────────────────────────┘
                                     │  HTTP (search)
                                     ▼
        ┌─────────────────────────────────────────────────────────┐
        │  rtfm-mcp  (stdio MCP server)                             │
        │   tool: search_docs(query, top_k)  → ranked chunks        │
        └───────────────────────────┬─────────────────────────────┘
                                     │  stdio (MCP)
                                     ▼
        ┌─────────────────────────────────────────────────────────┐
        │  LLM client  (Claude Code / Claude Desktop / IDE)         │
        └─────────────────────────────────────────────────────────┘
```

**Why three pieces and not one:** OpenSearch must persist independently of any
client session. The watcher must run continuously to keep the index fresh. The
MCP server only lives for as long as the LLM client that spawned it — so it
cannot host the watcher. Each piece has a different lifecycle, so each is its
own process.

---

## 2. Decisions

Decisions already made and locked, with the reasoning, so we don't relitigate
them mid-build.

### 2.1 Platform: .NET throughout
Matches team skills (.NET-first shop). The whole thing — CLI, conversion,
indexing, MCP server — is C#. No polyglot, no per-developer toolchain sprawl.
Target **.NET 10** (LTS; ships the MCP server project template).

**Solution file: use the `.slnx` (XML) format, not the legacy `.sln`.** The
solution is `Rtfm.slnx`. Create it with `dotnet new sln --format slnx` and add
projects with `dotnet sln Rtfm.slnx add …`.

### 2.2 MCP server: official C# SDK, stdio transport
Use the official `ModelContextProtocol` NuGet package (the
`AddMcpServer().WithStdioServerTransport().WithToolsFromAssembly()` pattern;
tools are plain methods tagged `[McpServerTool]`). **stdio**, not HTTP — the
client spawns the server as a child process locally. No ports, no auth, no CORS.

> **stdout is sacred (footgun).** A stdio MCP server must write *only* MCP
> protocol messages to stdout. Any stray `Console.WriteLine`, build output, or
> default-console log corrupts the transport and the server silently fails to
> connect. **All logging goes to stderr** — configure the host with
> `LogToStandardErrorThreshold = LogLevel.Trace` (as in the SDK's own sample).
> This is the single most common reason a .NET stdio server "won't connect."

### 2.3 Store: local single-node OpenSearch in Docker
One container, one persistent volume, security plugin disabled for local use.
No cluster, no auth headaches. Shipped via `docker-compose.yml` so a dev runs
`docker compose up -d` once — or `rtfm init [--with-model]`, which runs compose
(`up -d --wait` against the healthcheck), verifies connectivity, and creates
the index + hybrid pipeline in one shot. The compose file is also *embedded* in
`Rtfm.Cli` (linked from the repo root — one source of truth) and materialized
under `LocalApplicationData/rtfm/` when init runs outside the repo; resolution
order is cwd → `RTFM_HOME` → embedded copy. That embedded copy is what lets a
packaged, repo-less `rtfm` (Phase 14) still bootstrap a machine.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a7ex-turcan/rtfm](https://github.com/a7ex-turcan/rtfm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
