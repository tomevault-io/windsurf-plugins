---
trigger: always_on
description: This document helps AI coding agents understand the project structure, architecture, and conventions.
---

# SlopSearX — Agent Guide

This document helps AI coding agents understand the project structure, architecture, and conventions.

## Project Structure

```
slopsearx/
├── engines/            # Engine adapter plugins (one file per engine, 50 total)
│   ├── arxiv.py           brave.py           crates.py
│   ├── censys.py          clinicaltrials.py  courtlistener.py (removed)
│   ├── crtsh.py           cve.py             dehashed.py
│   ├── dockerhub.py       duckduckgo.py      edgar.py
│   ├── epss.py            exploitdb.py       fred.py
│   ├── github.py          google.py          greynoise.py
│   ├── hackernews.py      hibp.py            huggingface.py
│   ├── intelx.py          internetarchive.py mitreattack.py
│   ├── musicbrainz.py     nominatim.py       npm.py
│   ├── nvd.py             openalex.py        openfda.py
│   ├── openlibrary.py     otx.py             oyez.py
│   ├── pubchem.py         pubmed.py          pypi.py
│   ├── reddit.py          rubygems.py
│   ├── semanticscholar.py shodan.py          stackexchange.py
│   ├── tmdb.py            uniprot.py         urlhaus.py
│   ├── virustotal.py      vulncheck.py       wikipedia.py
│   ├── abuseipdb.py       ashby.py           greenhouse.py
│   └── lever.py
├── slopsearx/          # Core library
│   ├── adapter.py      # EngineAdapter base class + ScrapeAdapter
│   ├── service.py      # Normalized search pipeline (SearchService, ScopeResolver, AppContext)
│   ├── capabilities.py # Runtime capability catalog, intent profiles, MCP policy
│   ├── snapshot.py     # Opaque search snapshots for cursor pagination
│   ├── research.py     # Async research jobs (Valkey-backed)
│   ├── mcp/            # MCP server (FastMCP): tools, resources, prompts, auth,
│   │                   #   remote gateway mode (gateway.py), OAuth 2.1 server
│   │                   #   (oauth.py), and the gateway's OAuth client flow (oauth_client.py)
│   ├── merger.py       # Fan-out, deduplication, ranking
│   ├── config.py       # Layered config (env + file + defaults)
│   ├── ratelimit.py    # Distributed rate limiting (Valkey)
│   ├── cache.py        # Response cache
│   ├── formatter.py    # SearXNG JSON + YAML+Markdown formatters
│   └── server.py       # HTTP API (uvicorn/FastAPI) — thin adapter over service.py
├── spec.md             # Full architectural specification
├── tests/
├── docs/MCP_SERVER.md  # MCP server install/config/usage docs (users + agents)
├── CONTRIBUTING.md
├── AGENTS.md
└── README.md
```

## Key Architecture Rules

1. **The adapter interface is the primary invariant.** Every engine is one file, registered via `@register_engine`. Adding an engine requires zero changes to the orchestrator.
2. **Adapters never raise exceptions.** All errors are classified and returned in `AdapterResponse.status`. The orchestrator never sees an unhandled exception from any adapter.
3. **Internal schema is decoupled from wire format.** The `SearchResult` dataclass is the internal model. SearXNG JSON is one output formatter among many.
4. **Valkey is the only shared state.** No local volumes, no persistent DB, no per-replica state beyond what Valkey provides.
5. **Scrape engines use HTTP + HTML parsing.** No headless browsers. DDG and Google adapters use `httpx` + `lxml` for HTML parsing — the same approach SearXNG uses.
6. **README.md reflects every engine.** Adding or removing an engine file requires updating the Engines table in `README.md`. The table lists every registered adapter with its type, auth, and categories.
7. **One shared policy gate.** Every search-capable MCP path (generic `slopsearx_search`, `slopsearx_search_targeted`, jobs, security, science) and the scope-preview tool and research query planning reach a single fail-closed gate (`_enforce_policy` in `slopsearx/mcp/tools.py`) before any engine dispatch. Sensitive engines (`hibp`, `dehashed`) are unreachable on every path unless the uniform sensitive-engine grant `MCP_TARGETED_SENSITIVE_ALLOWED` is set. The specialist grants (`MCP_GRANT_JOBS/SECURITY/SCIENCE/RESEARCH`) enable their tools; they do **not** by themselves grant sensitive-engine access. A mixed sensitive + non-sensitive explicit engine list fails closed atomically.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magnus919/SlopSearX](https://github.com/magnus919/SlopSearX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
