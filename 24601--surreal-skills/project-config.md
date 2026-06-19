---
trigger: always_on
description: Expert SurrealDB 3 architect and developer skill. SurrealQL mastery, multi-model data modeling (document, graph, vector, time-series, geospatial), schema design, security, deployment, performance tuning, SDK integration (JS, Python, Go, Rust, Java, .NET, C, PHP, Swift, Kotlin, Ruby), Surrealism WASM extensions, SurrealML scope coverage (preview), SurrealMCP for AI agent hosts, LangChain Python integration, editor tooling (LSP, tree-sitter, CodeMirror, VS Code/JetBrains/Neovim/Zed), and ecosystem
---


# SurrealDB 3 Skill

Expert-level SurrealDB 3 architecture, development, and operations. Covers SurrealQL, multi-model data modeling, graph traversal, vector search, security, deployment, performance tuning, SDK integration, and the wider SurrealDB ecosystem, including SurrealKit, SurrealMCP, n8n, CodeMirror, and agent-skill surfaces.

## For AI Agents

Get a full capabilities manifest, decision trees, and output contracts:

```bash
uv run {baseDir}/scripts/onboard.py --agent
```

See [AGENTS.md]({baseDir}/AGENTS.md) for the complete structured briefing.

| Command | What It Does |
|---------|-------------|
| `uv run {baseDir}/scripts/doctor.py` | Health check: verify surreal CLI, connectivity, versions |
| `uv run {baseDir}/scripts/doctor.py --check` | Quick pass/fail check (exit code only) |
| `uv run {baseDir}/scripts/schema.py introspect` | Dump full schema of a running SurrealDB instance |
| `uv run {baseDir}/scripts/schema.py tables` | List all tables with field counts and indexes |
| `uv run {baseDir}/scripts/onboard.py --agent` | JSON capabilities manifest for agent integration |

## Prerequisites

- **surreal CLI** -- `brew install surrealdb/tap/surreal` (macOS) or see [install docs](https://surrealdb.com/docs/surrealdb/installation)
- **Python 3.10+** -- Required for skill scripts
- **uv** -- `brew install uv` (macOS) or `pip install uv` or see [uv docs](https://docs.astral.sh/uv/getting-started/installation/)

Optional:

- **Docker** -- For containerized SurrealDB instances (`docker run surrealdb/surrealdb:v3`)
- **SDK of choice** -- JavaScript, Python, Go, Rust, Java, Kotlin, .NET, C, PHP, Swift, or Ruby

> **Security note**: This skill documents package-manager and container installs
> only. Prefer auditable installs through Homebrew, apt/dnf, Cargo, npm, or
> Docker rather than remote one-line shell installers.

## Quick Start

> **Credential warning**: Examples below use `root/root` for **local development
> only**. Never use default credentials against production or shared instances.
> Create scoped, least-privilege users for non-local environments.

```bash
# Start SurrealDB in-memory for LOCAL DEVELOPMENT ONLY
surreal start memory --user root --pass root --bind 127.0.0.1:8000

# Start with persistent RocksDB storage (local dev)
surreal start rocksdb://data/mydb.db --user root --pass root

# Start with SurrealKV (time-travel queries supported, local dev)
surreal start surrealkv://data/mydb --user root --pass root

# Connect via CLI REPL (local dev)
surreal sql --endpoint http://localhost:8000 --user root --pass root --ns test --db test

# Import a SurrealQL file
surreal import --endpoint http://localhost:8000 --user root --pass root --ns test --db test schema.surql

# Export the database
surreal export --endpoint http://localhost:8000 --user root --pass root --ns test --db test backup.surql

# Check version
surreal version

# Run the skill health check
uv run {baseDir}/scripts/doctor.py
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SURREAL_ENDPOINT` | SurrealDB server URL | `http://localhost:8000` |
| `SURREAL_USER` | Root or namespace username | `root` |
| `SURREAL_PASS` | Root or namespace password | `root` |
| `SURREAL_NS` | Default namespace | `test` |
| `SURREAL_DB` | Default database | `test` |

These map directly to the `surreal sql` CLI flags (`--endpoint`, `--user`, `--pass`, `--ns`, `--db`) and are recognized by official SurrealDB SDKs.

## Core Capabilities

### SurrealQL Mastery

Full coverage of the SurrealQL query language: `CREATE`, `SELECT`, `UPDATE`, `UPSERT`, `DELETE`, `RELATE`, `INSERT`, `LIVE SELECT`, `DEFINE`, `REMOVE`, `INFO`, subqueries, transactions, futures, and all built-in functions (array, crypto, duration, geo, math, meta, object, parse, rand, string, time, type, vector).

See: `rules/surrealql.md`

### Multi-Model Data Modeling

Design schemas that leverage SurrealDB's multi-model capabilities -- document collections, graph edges, relational references, vector embeddings, time-series data, and geospatial coordinates -- all in a single database with a single query language.

See: `rules/data-modeling.md`

### Graph Queries

First-class graph traversal without JOINs. `RELATE` creates typed edges between records. Traverse with `->` (outgoing), `<-` (incoming), and `<->` (bidirectional) operators. Filter, aggregate, and recurse at any depth.

See: `rules/graph-queries.md`

### Vector Search

Built-in vector similarity search using HNSW indexes (the v3 vector index type; brute-force is available as a fallback during index build/rebuild). Define vector fields, create indexes with configurable distance metrics (cosine, euclidean, manhattan, minkowski), and query with `vector::similarity::*` functions. Build RAG pipelines and semantic search directly in SurrealQL.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [24601/surreal-skills](https://github.com/24601/surreal-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
