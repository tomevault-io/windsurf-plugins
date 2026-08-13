---
trigger: always_on
description: This is the **authoritative, highest-bar governance document** for AI agents across the entire `sbom-graph` monorepo. Sub-project `AGENTS.md` files inherit these standards and may only add project-specific technical context — they must never weaken the rules defined here.
---

# AGENTS.md — Monorepo Governance for AI Agents

This is the **authoritative, highest-bar governance document** for AI agents across the entire `sbom-graph` monorepo. Sub-project `AGENTS.md` files inherit these standards and may only add project-specific technical context — they must never weaken the rules defined here.

## Project Overview

`sbom-graph` is a monorepo for building, enriching, and visualising software dependency graphs from SBOM files. It stores data in FalkorDB and provides reports, visualisations, policy enforcement, and supply-chain trust scoring.

### Sub-Projects

| Sub-Project | Description | AGENTS.md |
|-------------|-------------|-----------|
| `sbom-graph-model` | Python library for parsing CycloneDX/SPDX SBOMs and persisting them to FalkorDB | [`sbom-graph-model/AGENTS.md`](sbom-graph-model/AGENTS.md) |
| `sbom-graph-api` | Flask application for REST API, reports, visualisations, and interactive documentation | [`sbom-graph-api/AGENTS.md`](sbom-graph-api/AGENTS.md) |
| `sbom-graph-enrichment` | Celery-based enrichment pipeline querying OSV, ClearlyDefined, OpenSSF Scorecard, Sonatype OSS Index, and deps.dev | [`sbom-graph-enrichment/AGENTS.md`](sbom-graph-enrichment/AGENTS.md) |
| `sonatype-lifecycle-release-listener` | Flask microservice receiving SCA webhook events and ingesting SBOMs | [`sonatype-lifecycle-release-listener/AGENTS.md`](sonatype-lifecycle-release-listener/AGENTS.md) |
| `sbom-graph-cli` | CLI for ingestion, querying, policy annotation, and report export | [`sbom-graph-cli/AGENTS.md`](sbom-graph-cli/AGENTS.md) |

### Cross-Project Dependencies

```
sbom-graph-api ──────────────► sbom-graph-model
       │                              ▲
       │ (optional)                   │
       ▼                              │
sbom-graph-enrichment ────────────────┘
       ▲
       │ (enqueue onto `ingest` queue only -- no direct model/graph dependency)
       │
sonatype-lifecycle-release-listener

sbom-graph-cli ──────────────► sbom-graph-api (HTTP client)

sbom-graph-api, sbom-graph-enrichment ──► FalkorDB (shared graph database)
sonatype-lifecycle-release-listener ──► FalkorDB's Redis instance (Celery broker/result DBs only, no graph access)
```

- `sbom-graph-api` depends on `sbom-graph-model` and optionally `sbom-graph-enrichment`
- `sbom-graph-enrichment` depends on `sbom-graph-model`
- `sonatype-lifecycle-release-listener` does **not** depend on `sbom-graph-model` -- it fetches SBOM/VEX documents from SonaType and enqueues them onto the `ingest` Celery queue for `sbom-graph-enrichment`'s worker pool to parse and persist; it holds no direct FalkorDB graph-write capability
- `sbom-graph-cli` communicates with `sbom-graph-api` via HTTP (no direct model dependency)
- `sbom-graph-cli` is a standalone CLI that calls the sbom-graph API (no direct FalkorDB dependency)
- `sbom-graph-api` and `sbom-graph-enrichment` share FalkorDB as the backing store; `sonatype-lifecycle-release-listener` shares only the broker/result-backend Redis DBs on the same instance

---

## Working Agreements (Mandatory)

These agreements apply to **all** sub-projects without exception.

1. All agents must operate in Privacy mode and use only approved models. **Never use fast/cheap models (e.g. `fast`) for code-generating, testing, or security subagents.** Fast models may only be used for trivial file searches.
2. Each code-generating agent must use a different model and focus area.
3. **Each code-generating agent must generate a complete design to be threat modelled before implementation, correct design flaws, and then implement the solution to be evaluated against the others.** This is non-negotiable for all new features and architectural changes.
4. All code must be well-architected, elegant, maintainable, and thoroughly documented.
5. Cognitive complexity must be minimised; rationale for complex logic must be documented inline.
6. All public APIs and methods must have docstrings/comments and be reflected in documentation.
7. **No hardcoded secrets, credentials, or API keys** in any code, configuration, or test fixture. Secrets must come from environment variables or a secret manager.
8. **All user input must be validated and sanitised** before use.
9. **Parameterised queries only** — no string concatenation for database queries (Cypher, SQL, or otherwise).
10. **Never include exception details in HTTP responses** (CWE-209: Generation of Error Message Containing Sensitive Information, CWE-497: Exposure of Sensitive System Information to an Unauthorized Control Sphere). Return a static, descriptive error message to the client instead. Exception details in log messages are acceptable at debug level only.
11. All agents must communicate findings in Markdown, using clear section headers and evidence appendices.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mimecast/sbom-graph](https://github.com/mimecast/sbom-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
