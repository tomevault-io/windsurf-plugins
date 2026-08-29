---
trigger: always_on
description: Guidance for working in this repository. [DESIGN.md](DESIGN.md) is the decision
---

# CLAUDE.md

Guidance for working in this repository. [DESIGN.md](DESIGN.md) is the decision
record — rationale, dates, and reversals live there; this file states only the
current rules. [docs/anaf-reference/](docs/anaf-reference/) is the compiled
local reference of ANAF's APIs. `docs/` is also the MkDocs source of the public
docs site (Read the Docs, `https://anafpy.readthedocs.io`).

## What this is

`anafpy` — typed async Python clients for Romania's **ANAF** tax-authority
services, plus a local stdio **MCP server** (`anafpy.mcp`, extra `anafpy[mcp]`)
exposing the same operations as Claude tools and skills. It is a **thin,
stateless transport client** — no persistence, no accounting logic; client
methods map 1:1 onto MCP tools. Python **3.12+** (dev pin 3.13), **httpx2**,
**Pydantic v2**.

The service strands:

- **e-Factura** (electronic invoicing). Outbound has two shapes (DESIGN.md §1):
  **XML pass-through is strongly recommended** when the caller runs invoicing
  software — bring its complete UBL XML; anafpy never re-composes an upstream
  document, and ANAF's SPV is *not* invoice storage (it purges filed messages
  after ~60 days, so the durable record lives upstream). **Structured
  authoring** (`efactura.authoring`) is the first-class path without one: a
  bidirectional `InvoiceDocument` covers invoice + credit note (`kind` picks
  the render target), totals/VAT computed with explicit overrides preserved, a
  hand-translated EN 16931 + CIUS-RO rule set (`validate()`, findings with
  official BR-* ids; ANAF stays authoritative), byte-stable
  render/read round-trips, `EFacturaClient.upload_invoice`. The same model
  backs the inbox — but **not the same contract**: the construction checks are
  authoring's, and nothing the caller did not author is judged by them
  (`_DERIVED_CONTEXT` — a document read off the wire, or a value `compute_*`
  derives), so the reader never rejects a document ANAF accepted and a read one
  always renders back. `DownloadedMessage.view` never raises
  (`None` when not representable — a missing mandatory element or off-list
  code; cause on `view_error` + a warning; raw bytes + full UBL model are the
  fallback tiers).
- **e-Transport** (goods transport) — **fully translated**: bidirectional flat
  models author a filing and view a parsed one, covering all four operations
  (declaration/correction, deletion, confirmation, vehicle change); XML input
  remains supported. **UIT presentation** (extra `anafpy[cards]`, DESIGN.md §13)
  renders a filed declaration into two PDFs — a phone-shaped driver card and an
  A4 detail document — locally, informative, never issued by ANAF.
- **Public no-auth services** (`anafpy.public`) — registry lookups, financial
  statements, and the stateless e-Factura `validare`/`transformare`.
- **SPV** (`anafpy.spv`) — read-only mailbox over a certificate cookie session.
- **Declarations** (`anafpy.declaratii`, DESIGN.md §12) — local authoring +
  validation via ANAF's DUKIntegrator (managed-installed from ANAF's update
  feed), official-PDF rendering, qualified signing (the raw op delegated to the
  OS token — no key material or PIN in-process), portal filing
  (production-only; opt-out `ANAFPY_DECLARATII_UPLOAD=off`), and no-auth
  StareD112 status/recipisa tracking.

Distribution is **free and as-is**: the MCP server is best-effort, and
configuring it — including provisioning the OAuth application on ANAF's portal —
is the user's responsibility (DESIGN.md §11).

## Commands

```bash
uv sync --all-extras                 # set up env with all dev dependency groups
uv run pytest -q                     # tests (respx-mocked, credential-free)
uv run pytest tests/test_auth.py     # one file
uv run ruff check . && uv run ruff format --check .
uv run mypy                          # strict
uv run mkdocs build --strict         # docs site (broken internal links fail); `serve` to preview
ANAFPY_LIVE=1 uv run pytest -m live  # opt-in live smoke: public services + authenticated TEST (needs .env + auth login)
```

Run the MCP server (host-side, where the `anafpy auth login` token store lives):

```bash
ANAFPY_CLIENT_ID=... ANAFPY_CLIENT_SECRET=... ANAFPY_CIF=... \
  uv run python -m anafpy.mcp        # stdio; or the `anafpy-mcp` console script
```

Server config is env-only — `ServerConfig()` **is** the environment (a
`BaseSettings`; kwargs override it, which is how the tests inject values), and an
invalid value raises `AnafConfigError`, not pydantic's `ValidationError`, so a
misconfiguration stays inside the `AnafError` hierarchy. The variable-by-variable
semantics live in [mcp/config.py](src/anafpy/mcp/config.py)'s docstrings and the
[docs/mcp/tools.md](docs/mcp/tools.md) table — don't retell them here. The
cross-cutting facts: credentials are optional (without them the public `anaf_*`
tools still serve); `ANAFPY_DOCS_DIR`'s wheel copy is a curated hatchling
force-include (a wheel-map tripwire in `test_mcp_server.py` catches a subtree
missing from the map); `ANAFPY_CURL` (both certificate bootstraps; the user's
override — SPV reference §1.1) and `ANAFPY_BUNDLED_CURL` (what the Windows
extension ships, set by the manifest, outranked by `ANAFPY_CURL`) are read at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robert-malai/anafpy](https://github.com/robert-malai/anafpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
