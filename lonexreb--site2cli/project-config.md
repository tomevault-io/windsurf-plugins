---
trigger: always_on
description: Turn any website into a CLI/API for AI agents.
---

# site2cli

Turn any website into a CLI/API for AI agents.

## Architecture

Progressive Formalization: 3-tier system that auto-graduates from browser automation (Tier 1) → cached workflows (Tier 2) → direct API calls (Tier 3).

## Project Structure

```
src/site2cli/
├── cli.py              # Typer CLI entry point
├── config.py           # Configuration management
├── models.py           # Pydantic v2 data models
├── registry.py         # SQLite site registry
├── router.py           # Tier router (picks best execution method)
├── discovery/
│   ├── capture.py      # CDP-based network traffic capture
│   ├── analyzer.py     # LLM-assisted pattern analysis
│   ├── spec_generator.py  # OpenAPI 3.1 spec (JSON + YAML)
│   ├── client_generator.py # Python client code generation
│   ├── js_client_generator.py # Zero-dep ES-module JS client (.mjs)
│   ├── coverage_report.py  # Dark-theme HTML coverage report with gap detection
│   └── trace.py        # Portable Trace JSON (save_trace / load_trace) for offline replay
├── browser/
│   ├── retry.py        # Async retry with delay for browser actions
│   ├── wait.py         # Rich wait conditions (network-idle, selector, stable)
│   ├── cookie_banner.py # Cookie consent auto-dismissal (3 strategies)
│   ├── detectors.py    # Auth/SSO/CAPTCHA page detection
│   ├── a11y.py         # Accessibility tree extraction + indexed [@N] notation
│   ├── context.py      # Unified browser context factory (profiles + sessions)
│   └── session.py      # Named browser session persistence and reuse
├── output_filter.py    # Output filtering (grep, limit, keys-only)
├── generators/
│   ├── cli_gen.py      # Dynamic CLI command generation
│   ├── mcp_gen.py      # MCP server generation
│   └── agent_config.py # Agent config generation (Claude MCP, generic)
├── content/
│   ├── converter.py    # HTML to markdown/text conversion, main content extraction
│   ├── chunker.py      # RAG chunking (fixed / sentence / heading strategies)
│   └── pdf.py          # PDF parsing via pdfplumber (text / markdown / page count)
├── search/
│   └── engine.py       # DuckDuckGo search + chained scrape / extract
├── crawl/
│   ├── crawler.py      # Async BFS site crawler with resume and streaming
│   ├── links.py        # Link extraction and normalization from HTML
│   └── robots.py       # robots.txt parser and URL filtering
├── extract/
│   └── extractor.py    # LLM-powered structured extraction with schema validation
├── monitor/
│   ├── watcher.py      # Change detection with snapshot comparison and webhooks
│   └── differ.py       # Line-level diff engine (stdlib difflib)
├── screenshot/
│   └── capture.py      # Full-page and element screenshots via Playwright
├── auth/
│   ├── manager.py      # Auth flow management (Playwright cookie format)
│   ├── cookies.py      # Cookie CRUD, import/export (Playwright-compatible)
│   ├── profiles.py     # Chrome/Firefox profile detection & import
│   ├── device_flow.py  # OAuth device flow (RFC 8628)
│   └── providers.py    # Pre-configured OAuth providers (GitHub, Google, Microsoft)
├── tiers/
│   ├── browser_explorer.py  # Tier 1: LLM-driven browser
│   ├── cached_workflow.py   # Tier 2: Recorded workflow replay
│   └── direct_api.py        # Tier 3: Direct API calls
├── daemon/
│   ├── server.py       # Background browser daemon (JSON-RPC over Unix socket)
│   └── client.py       # Daemon client for CLI commands
├── mcp/
│   └── server.py       # Unified MCP server for ALL discovered sites
├── orchestration/
│   ├── data_flow.py    # JSONPath-like data extraction between pipeline steps
│   ├── loader.py       # YAML/JSON pipeline loading
│   └── orchestrator.py # Sequential pipeline executor with error policies
├── health/
│   ├── monitor.py      # API health checking
│   └── self_heal.py    # LLM-powered breakage repair
└── community/
    └── registry.py     # Community spec sharing

experiments/
├── experiment_8_live_validation.py   # Live validation against 5 real APIs
├── experiment_9_api_breadth.py       # Breadth test across 10 diverse APIs
├── experiment_10_unofficial_api_benchmark.py  # Coverage vs known unofficial APIs
├── experiment_11_speed_cost_benchmark.py      # Speed, cost, throughput benchmarks
├── experiment_12_mcp_validation.py   # Deep MCP server validation
├── experiment_13_spec_accuracy.py    # Spec accuracy vs ground truth
├── experiment_14_resilience.py       # Health monitoring & resilience
├── experiment_15_live_browser_validation.py  # Real Playwright browser → CDP capture pipeline
└── run_all_experiments.py            # Master runner for all experiments
```

## Conventions

- Python >=3.10, type hints everywhere
- Pydantic v2 for all data models
- async/await for I/O-bound operations
- Typer for CLI, Rich for output formatting
- SQLite for local storage (no server deps)
- ruff for linting

## Testing

```bash
pytest                    # 553 unit/integration tests (no network)
pytest -m live            # 6 live tests (hits jsonplaceholder + httpbin)
pytest -v                 # Verbose output
```

**Total: 559 tests (553 + 6 live), all passing in <8s.**

**Test files:**
- `test_analyzer.py` — Traffic analysis & grouping (23 tests)
- `test_cli.py` — CLI commands via CliRunner (16 tests)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lonexreb/site2cli](https://github.com/lonexreb/site2cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
