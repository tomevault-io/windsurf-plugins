---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Data Wiki turns AWS Glue databases into portable knowledge bundles and serves them to AI agents over MCP. An LLM reads a Glue catalog, authors markdown docs describing each dataset (tables, joins, metrics, known issues), keeps them in sync as the catalog changes, and exposes them to agents over an MCP server.

Bundles are **Open Knowledge Format (OKF)** bundles — a directory of markdown files with YAML frontmatter. The `okf`/`OKF_` prefix on identifiers, resource names, and env vars refers to the *format*, not the product. OKF's model maps onto AWS as `data domain → dataset (Glue database) → table`.

## Read these first

The repo is documentation-heavy and the docs are authoritative. Before making changes, read the relevant one:

- **`docs/CONVENTIONS.md`** — the contract between services: S3 bundle layout, DynamoDB item shapes, the harvest lease, the harvest invocation payload, and every `OKF_*` env var. A mismatch here is an integration bug that ripples across services. **Changes to these shapes affect every component** — keep them intact.
- **`docs/ARCHITECTURE.md`** — how the seven components fit and the non-obvious reasoning behind them.
- **`docs/API_REFERENCE.md`** — the exact third-party API shapes (deepagents, AgentCore, S3 Vectors, Titan/Glue/Athena, Terraform `aws ~> 6.0`, react-oidc) the code was written against. These are the details that are easy to get wrong.
- **`docs/SNAPSHOT_EVIDENCE.md`** — the two deterministic snapshot-time passes (column profiles, relationship evidence sheets): nomination heuristics, size-gate/sampling statistics, verdicts, caching. Read before touching `profile.py`, `relationships.py`, or `probes.py`.
- **`docs/ATTESTED_COMPUTATIONS.md`** — the frozen-parameterized-SQL feature (`references/computations/`): the trust model (content hash + human verification via the off-mount overlay + runtime fold-in; VERIFIED docs are frozen to agents in the in-place modes — guard-refused, unlocked only by a human Unverify; a full harvest is deliberately destructive and re-authors them), the parameter contract-vs-evidence split, and the guard/lint/executor surfaces. Read before touching `okf_core/computations.py`, `okf_aws/computation_run.py`, or `harvest/verification.py`.

## Architecture

Seven Python services under `services/`, two Terraform stacks under `infra/`, a React SPA in `ui/`, and a Claude Code plugin in `okf-mcp/`.

**Shared libraries (import from these, don't re-implement):**
- `okf_core/` — pure-Python OKF primitives, no AWS or agent deps. Owns the source-of-truth invariants: `paths.py` (concept id ↔ S3 key), `embedding.py` (vector key + embed text/metadata builders), `document.py`, `link_graph.py`, `guard.py`, `session.py` (`runtime_session_id`), `index_gen.py`, `hive_types.py`.
- `okf_aws/` — shared boto3 helpers: Titan embed, S3 Vectors, S3 keys.

**Runtime services:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-okf-llm-wiki](https://github.com/aws-samples/sample-okf-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
