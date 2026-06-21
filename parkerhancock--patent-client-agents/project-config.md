---
trigger: always_on
description: Async Python library and MCP server for patent, trademark, litigation, fee, and
---

# Patent Client Agents

Async Python library and MCP server for patent, trademark, litigation, fee, and
substantive-law research.

## Development

```bash
uv sync --group dev              # Install dependencies
uv run pytest                    # Run the offline/VCR replay suite
uv run ruff check src/ tests/    # Lint
uv run ruff format src/ tests/   # Format
```

## Testing

Tests use `vcrpy` to replay recorded HTTP interactions. Cassettes live at
`tests/cassettes/` with filenames derived from each test's nodeid.

```bash
uv run pytest                                # Replay from cassettes (default)
uv run pytest --vcr-record=once              # Record missing cassettes
uv run pytest --vcr-record=all               # Re-record all cassettes
uv run pytest --run-live-uspto --vcr-record=once  # Allow missing USPTO cassettes to record
uv run pytest --run-live-jpo --vcr-record=once    # Allow missing JPO cassettes to record
uv run pytest --cov=patent_client_agents --cov-report=term-missing  # Coverage
```

When recording new cassettes, ensure auth headers are scrubbed. The conftest
redacts `authorization`, `x-api-key`, `api_key`, and `apiKey` automatically;
verify with `gitleaks protect --staged` before committing.

## Architecture

Shared HTTP scaffolding (`BaseAsyncClient`, cache, retry, exceptions, logging,
MCP server plumbing) lives in the separately-published
[`mcp-data-core`](https://github.com/parkerhancock/mcp-data-core) package
(in this monorepo at `tools/mcp-data-core/`). `patent-client-agents` depends
on it and re-imports primitives via `from mcp_data_core import ...`.

```
src/
  patent_client_agents/
    uspto_odp/              # USPTO Open Data Portal — applications, PTAB, petitions
                            #   (needs USPTO_ODP_API_KEY)
    uspto_applications/     # High-level wrapper over ODP applications endpoints
    uspto_publications/     # USPTO Public Search (PPUBS)
    uspto_assignments/      # USPTO Assignment Center records
    uspto_petitions/        # USPTO petitions API
    uspto_bulkdata/         # USPTO bulk data product catalog
    uspto_office_actions/   # Office-action rejections, citations, full text
    epo_ops/                # EPO Open Patent Services (needs EPO_OPS_API_KEY/SECRET)
    google_patents/         # Scrapes Google Patents (no API key)
    jpo/                    # Japan Patent Office (needs JPO_API_USERNAME/PASSWORD)
    kipo_kipris/            # KIPO Korea — KIPRIS Plus REST API for patents,
                            #   utility models, trademarks, designs.
                            #   BYOK per ToS §11; needs KIPO_KIPRIS_API_KEY.
    ip_australia_common/    # Shared OAuth + host scaffolding for the AU OAuth APIs
    ip_australia_patents/   # IP Australia Patents (needs IPAUSTRALIA_CLIENT_ID/SECRET)
    ip_australia_trademarks/# IP Australia Trade Marks (needs IPAUSTRALIA_CLIENT_ID/SECRET)
    ip_australia_designs/   # IP Australia Designs (needs IPAUSTRALIA_CLIENT_ID/SECRET)
    ip_australia_bulk/      # IP RAPID bulk catalog on data.gov.au (no auth)
    ipo_in_statutes/        # IPO India — Patents/Designs/TM/Copyright Acts
                            #   + Patent Rules in one SQLite/FTS5 corpus.
    ipo_in_mppp/            # IPO India Manual of Patent Practice &
                            #   Procedure (MPPP v3.0, 2019). Static corpus.
    dpma_statutes/          # DPMA Germany — PatG/MarkenG/GebrMG/DesignG/
                            #   UrhG/GeschGehG in one SQLite/FTS5 corpus.
    legifrance_ip/          # Légifrance IP — CPI (patents/TM/designs/
                            #   copyright) + Code de commerce L.151 (trade
                            #   secrets) in one SQLite/FTS5 corpus.
    tw_trade_secrets/       # Taiwan Trade Secrets Act (營業秘密法,
                            #   EN translation) Arts. 1/2/3/10/11/13/13-1.
                            #   Single-statute SQLite/FTS5 corpus.
    tipo_opdata/            # TIPO Taiwan OpenData REST — patents, utility
                            #   models, designs, trademarks (biblio-only).
                            #   Needs TIPO_API_KEY (single ``tk`` UUID).
    inpi_pi/                # INPI France — National Trademarks (ST.66
                            #   v1.0) + Designs (ST.86 v1.0). TM + Design
                            #   only; FR patents via EPO OPS. BYOK
                            #   session-bearer + XSRF auth — needs
                            #   INPI_USERNAME + INPI_PASSWORD.
    cpc/                    # CPC classification lookups (via EPO OPS)
    mpep/                   # Manual of Patent Examining Procedure search
    skills/ip_research/     # Skill content packaged with the wheel; downstream
                            #   consumers can read it via importlib.resources.
archive/                    # Deprecated modules kept for reference, not shipped.
                            #   (patentsview: upstream API retired by USPTO.)
```

## Error Handling

Log-first pattern for API errors: `ApiError.__str__()` appends the log-file
path so agents can inspect details without keeping full stacktraces in
context. Base class `McpDataCoreError` (from `mcp_data_core.exceptions`)
and plain validation/config errors use vanilla `Exception.__str__`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parkerhancock/patent-client-agents](https://github.com/parkerhancock/patent-client-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
