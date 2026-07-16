---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the single source of truth for all **public Anypoint Platform API specifications**. Each service has its own directory containing OpenAPI 3.0 specs validated against AI-agent-friendly best practices.

## Common Commands

### Validation

```bash
# Validate all APIs with governance rules (recommended before PRs)
make validate-all-governed

# Validate specific API
make validate-api API=api-manager

# List all discoverable APIs
make list-apis

# Generate comprehensive validation report
make report

# Clean validation reports
make clean
```

### Validation Tools

The repository uses **Anypoint CLI v4** for validation:
- Basic OAS validation: structural correctness, schema consistency
- Governed validation: AI-agent best practices from `./.claude/skills/api-spec-validator/scripts/ruleset.yaml`

### Python Validators

```bash
# Validate x-origin annotations
python3 scripts/build/validate_xorigin.py

# Validate Jobs To Be Done (JTBD) format
python3 scripts/build/validate_jtbd.py <path-to-job-file> .

# Validate all mcps/<name>/server.json files against the MCP registry schema
python3 scripts/build/validate_mcp_server.py

# Validate a deployed portal's agentic endpoints (C1-C5: core URLs, registry, hrefs, OAS, SKILL.md)
python3 scripts/build/validate_portal.py --url <URL> [--header "X-MS-Developer: true"]
```

### Portal Generator Tests

```bash
# Run all portal generator tests via Makefile
make test-portal

# Or run directly with pytest (from scripts/ directory)
cd scripts && python3 -m pytest tests/ -v

# Run individual test files
cd scripts && python3 -m pytest tests/test_units.py -v
cd scripts && python3 -m pytest tests/test_oas_parser.py -v
cd scripts && python3 -m pytest tests/test_smoke.py -v

# Install test dependencies (first time only)
pip3 install -r scripts/requirements.txt
```

The test suite covers:
- **Unit tests** (`test_units.py`): pure functions across utils, tree builder, Jinja2 filters, generator helpers, skill parser, discovery stats, and the x-origin operation lookup builder
- **OAS parser tests** (`test_oas_parser.py`): `$ref` resolution (internal, external, fragment pointers), `allOf` merging, schema property extraction with constraints, operation extraction, example loading
- **Smoke tests** (`test_smoke.py`): full end-to-end generation against minimal fixtures, output file validation, HTML structure checks with BeautifulSoup

## Architecture

### Directory Structure

All API specifications are organized under the `apis/` directory:
```
apis/
└── <service-name>/
    ├── api.yaml           # OpenAPI 3.0 specification
    ├── exchange.json      # Exchange metadata (groupId, assetId, version)
    ├── schemas/           # Optional: reusable schema definitions
    └── examples/          # Optional: request/response examples
```

Skills (JTBD files) live in a top-level `skills/` directory:
```
skills/
└── <skill-name>/
    └── SKILL.md       # Job-to-be-done written in agent-skill format
```

Skills are automatically associated with APIs by parsing `urn:api:` references in their YAML step blocks. A skill that references multiple APIs appears on each of those API's portal pages. Skills can also reference MCP servers via `urn:mcp:<name>`.

MCP servers live under a top-level `mcps/` directory:
```
mcps/
└── <server-name>/
    ├── exchange.json  # Exchange metadata (name, version, visibility, ...)
    ├── server.json    # MCP registry descriptor (title / description / version / remotes)
    └── mcp.yaml       # MCP metadata: transport, capabilities, tools, prompts, resources
```

The Makefile auto-discovers APIs by finding `exchange.json` files in the `apis/` directory, and MCP servers by finding `mcp.yaml` files under `mcps/`.

Terraform provider docs live under a top-level `terraform/` directory, **versioned per provider**:
```
terraform/
└── <provider-name>/
    └── <version>/                # semver directory name (e.g. 1.0.0, v0.9.0)
        ├── provider.json         # install snippet metadata for this version
        ├── resources/
        │   └── *.md
        ├── data-sources/
        │   └── *.md
        └── guides/               # optional
            └── *.md
```

Each `(provider, version)` is rendered as a standalone HTML page at `terraform/<provider>/<version>.html`. The portal serves the latest version by default via `terraform/<provider>/index.html` (a static meta-refresh stub) and preserves backward-compatible deep links from the legacy unversioned URL `terraform/<provider>.html`.

Version directories must match `^v?\d+\.\d+\.\d+(-[A-Za-z0-9.-]+)?$`; anything else is skipped at discovery with a warning.

### Special Extensions

#### x-origin Annotations

`x-origin` documents dynamic enum sources - which API provides the values, which operation to call, and how to extract both identifiers (`values`) and display names (`labels`) from the response. Key rules:

- Always an array of source objects (even for single source)
- Must reference valid `operationId` in target API spec
- `values` is required (JSONPath expression), `labels` is optional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mulesoft/mulesoft-dx](https://github.com/mulesoft/mulesoft-dx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
