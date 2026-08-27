---
trigger: always_on
description: rgctl is designed so agents answer **structural questions** from a pre-built graph instead of reading whole files into context.
---

# rgctl for AI agents

rgctl is designed so agents answer **structural questions** from a pre-built graph instead of reading whole files into context.

**Installation:** [docs/installation.md](docs/installation.md) (prerequisites, modes, setup)  
**Full JSON reference:** [docs/json-api.md](docs/json-api.md) (also on the site: [sshaaf.github.io/rgctl/docs/json-api/](https://sshaaf.github.io/rgctl/docs/json-api/))  
**Copy-paste recipes:** [docs/agent-recipes.md](docs/agent-recipes.md)  
**Human walkthrough:** [docs/user-guide.md](docs/user-guide.md)  
**Docs hub:** [docs/README.md](docs/README.md) · [site docs](https://sshaaf.github.io/rgctl/docs/)

Do **not** open the browser dashboard unless the user asks for a visual UI. In an IDE with `serve --mode mcp` already connected, prefer the MCP catalog. Otherwise default to CLI `-f json`.

Install the project skill once (Claude Code + Cursor dirs under the repo):

```bash
rgctl -r "$REPO" install --skill
```

---

## Agent workflow

```text
1. rgctl discover . --full       # or plain discover; --full adds CFG/dashboard/semantic
2. rgctl -f json <command>      # compact facts on stdout
3. Parse schema_version + payload   # never scrape stderr for JSON
```

Set `REPO` to the repository root (where indexed artifacts live — `{repo}/.rgctl/` with `--no-daemon`, or `~/.rgctl/cache/{reponame}/` via the default daemon):

```bash
export REPO=/path/to/repo
rgctl -r "$REPO" -f json gql 'MATCH (n:Function) RETURN n LIMIT 20'
```

---

## High-value commands (low token cost)

| Intent | Command |
|--------|---------|
| Full session (graph + CFG + dashboard + semantic) | `rgctl discover PATH --full` (queryable after stage 1; status in `.rgctl/pipeline_status.json`) |
| HTTP session (auto-pipeline) | `rgctl serve` — `GET /api/status`; `--no-pipeline` restores fail-fast |
| MCP session (stdio) | `rgctl serve --mode mcp` — tools `rgctl_status`, `rgctl_query`, `rgctl_search`, `rgctl_impact`, `rgctl_metrics`, `rgctl_cpg`, `rgctl_check`. Default query/search `limit` 20. Unready tools return pipeline status JSON. Guide: [docs/guides/mcp-server.md](docs/guides/mcp-server.md) |
| Inventory functions | `rgctl -f json gql --macro-name all_functions unused` |
| List communities | `rgctl -f json gql --macro-name all_communities unused` |
| Find symbol by pattern | `rgctl -f json gql "MATCH (n:Function) WHERE n.name LIKE '*Service*' RETURN n LIMIT 20"` |
| Find by FQN (not `n.name`) | `rgctl -f json gql "MATCH (n:Class) WHERE n.qualified_name = 'com.example.Foo' RETURN n"` |
| Community members | `rgctl -f json gql "MATCH (f:Function) WHERE f.community_id = '12' RETURN f LIMIT 20"` |
| Natural-language function search | `rgctl semantic index` then `rgctl -f json semantic query "checkout flow" --limit 10` |
| Community semantic search | `rgctl -f json semantic query "checkout" --scope community --limit 10` |
| Impact before editing | `rgctl -f json blast-radius <Symbol> [--depth N]` |
| Architectural hotspots | `rgctl -f json metrics --pagerank` |
| Call neighborhood | `rgctl -f json gql "MATCH (a:Function)-[:CALLS*1..3]->(b:Function) RETURN a,b LIMIT 50"` |
| Doc headings / cross-links | `discover` indexes `.md` / `.mdx` by default; GQL on `:Module` with `kind=heading` and `REFERENCES` — see [markdown-context.md](docs/markdown-context.md) |
| Obsidian vault from docs | `rgctl -r "$REPO" discover . -l markdown` then `export --export-format obsidian --export-output "$REPO/vault" --query all` — see [markdown-context.md](docs/markdown-context.md#obsidian-vault-export) |
| Doc section semantic search | `rgctl semantic index --scope docs --embedder hash` then `rgctl -f json semantic query "checkout flow" --scope docs --limit 10` (query scope does not filter — index must be doc-scoped) |
| Hybrid CPG status / CALL / PDG / slice | `rgctl -f json cpg status` then `cpg function\|calls\|pdg\|slice` (needs `discover --with-cfg` for PDG/slice) |
| Field mutations (cart / DTO safety) | `rgctl -f json cpg mutations --type ShoppingCart --exclude-ctors` (ecommerce CoolStore; or any type name; needs `--with-cfg`) |
| Data flows / slice (CPG) | `rgctl -f json cpg flows FILE --line N --variable V --function F [--direction forward\|backward] [--with-alias]` |
| Loop-carried DFG tags | `rgctl discover . --with-cfg --with-dfg-loops` (tags `DataDependency.loop_carried` in PDG) |
| AST skeleton | `rgctl discover --with-ast-skeleton` then `rgctl -f json cpg ast <Symbol>` |
| CPG export | `rgctl cpg export --format graphson --output cpg.json [--path-contains src/]` |
| Migration plan | `rgctl discover . --with-cfg --with-security --with-taint --with-dashboard --with-harmonic --export-migration-hints` then read `.rgctl/migration_plan.json` (or dashboard copy) |
| CI gate on changes | `rgctl -f json check --policy-file policy.json` (exit 1 = violations) |

---

## Repeated queries in one session

**Option A — HTTP (recommended):**

```bash
rgctl -r "$REPO" serve --open
# POST http://127.0.0.1:8080/api/query  {"query":"MATCH (n:Function) RETURN n LIMIT 5"}
```

See [docs/http-api.md](docs/http-api.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sshaaf/rgctl](https://github.com/sshaaf/rgctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
