---
trigger: always_on
description: LLM-driven pipeline that takes a customer's domain brief (use-cases in natural language)
---

# MCP-Anything

## What this project does
LLM-driven pipeline that takes a customer's domain brief (use-cases in natural language)
and a data source (OpenAPI / gRPC / DB schema / SDK), and produces a fully-implemented,
optimized MCP server plus skill bundle and validation artifacts.

Two output backends: **Python/FastMCP** and **TypeScript/mcp-use**.

Legacy path: `mcp-anything generate <path>` (codebase scanner, backwards-compatible).
Domain path: `mcp-anything build --brief <brief.yaml>` (new, recommended).

## Development
- Install: `pip install -e ".[dev,llm]"`
- Run tests: `pytest tests/ -v`
- Run CLI: `mcp-anything --help`

## Architecture

### Domain Pipeline (new, primary)
5 phases: DOMAIN_MODELING → TOOL_DESIGN → EMIT → SKILL_BUNDLE → VALIDATION_HARNESS
- Phase 1 (`domain_modeling.py`): LLM reads brief + data source → `domain_model.json`
- Phase 2 (`tool_design.py`): LLM shapes tools per 2026 rules → `tool_spec.yaml`
- Phase 3 (`emit/python_fastmcp/` or `emit/typescript_mcp_use/`): code generation
- Phase 4 (`skill_bundle.py`): LLM generates `SKILL.md` + `quick_queries.json`
- Phase 5 (`validation_harness.py`): LLM generates `eval_cases.json`, optional live eval
- Output contract: `CONTRACT.md` — 29 testable items (C-01..C-29) both emitters must satisfy
- Conformance suite: `src/mcp_anything/conformance/` with parity assertion and CI reporter

### Legacy Pipeline (preserved, unchanged)
5 phases: ANALYZE → DESIGN → IMPLEMENT → DOCUMENT → PACKAGE
- Static detectors + optional LLM analysis
- Jinja2 templates in `src/mcp_anything/codegen/templates/`
- `mcp-anything generate <path>` — unchanged

### 2026 Stack Defaults (wired into domain pipeline)
- Group CRUD: ≥3 CRUD ops on same resource → single `manage_X(operation=...)` tool
- Composed tools: multi-step workflows promoted to single callable
- Progressive disclosure: `disclosure_level` on every tool, verbose-only tools hidden by default
- Compact responses: every tool has `verbose` flag (C-10)
- Discovery endpoint: `GET /.well-known/mcp` (C-01..C-03)
- Telemetry: anonymized per-call logging, `MCP_TELEMETRY_ENDPOINT` env var (C-19..C-20)
- Docker: `Dockerfile` in every generated server (C-17)
- SKILL.md: required sections Overview, Tools, Usage Patterns, Gotchas, Recipes, Anti-patterns (C-22)

## Key Files
- `src/mcp_anything/cli.py` — CLI entry point (subcommands: model, build, validate, generate, serve)
- `src/mcp_anything/pipeline/engine.py` — phase orchestration; ALL_PHASES (legacy) + DOMAIN_PHASES
- `src/mcp_anything/pipeline/domain_modeling.py` — Phase 1: domain brief → DomainModel
- `src/mcp_anything/pipeline/tool_design.py` — Phase 2: DomainModel → ServerDesign (2026 rules)
- `src/mcp_anything/pipeline/llm_client.py` — shared LLM call utility with JSON retry
- `src/mcp_anything/emit/python_fastmcp/phase.py` — Phase 3 (Python): ServerDesign → FastMCP server
- `src/mcp_anything/emit/typescript_mcp_use/phase.py` — Phase 3 (TS): ServerDesign → mcp-use server
- `src/mcp_anything/emit/base.py` — EmitPhase ABC + structural CONTRACT.md checker
- `src/mcp_anything/pipeline/skill_bundle.py` — Phase 4: SKILL.md + quick_queries.json
- `src/mcp_anything/pipeline/validation_harness.py` — Phase 5: eval_cases + conformance_report
- `src/mcp_anything/conformance/` — EvalRunner, ConformanceParity, ConformanceReporter
- `src/mcp_anything/models/domain.py` — DomainBrief, DomainModel, UseCase, GlossaryTerm
- `src/mcp_anything/models/validation.py` — EvalCase, EvalResult, ConformanceReport
- `src/mcp_anything/models/design.py` — ServerDesign + ToolGroup, ComposedTool (extended)
- `src/mcp_anything/models/manifest.py` — GenerationManifest v0.2.0 (dual pipeline_mode)
- `CONTRACT.md` — 29 testable output contract items for both emitters
- `src/mcp_anything/pipeline/scope.py` — scope filtering (--include/--exclude/--review/--scope-file)
- `src/mcp_anything/codegen/emitter.py` — legacy Jinja2 template rendering
- `src/mcp_anything/url_fetcher.py` — URL spec fetching and type detection

## Detectors (17 total)
Located in `src/mcp_anything/analyzers/`:
- Python: CLI (argparse/click/typer), AST (functions/classes), Flask/FastAPI, Django DRF
- Java: Spring Boot, Spring MVC, JAX-RS/Quarkus, Micronaut
- JS/TS: Express.js
- Go: Gin, Echo, Chi, net/http, gorilla/mux
- Ruby: Rails
- Rust: Actix, Axum, Rocket, Warp
- Specs: OpenAPI 3.x/Swagger 2.x, GraphQL SDL, gRPC/Protobuf
- Other: Socket, File I/O, WebSocket, --help text parser

---

## Current Status (verified 2026-03-23)

### What's WORKING (tested end-to-end with real generation)

All of these produce valid Python and install correctly.
All have integration tests + were validated against real open-source repositories.

| Source Type | Detection | Tools | Real Repo Tested |
|---|---|---|---|
| Python CLI (argparse) | 0.90 confidence | Correct | httpie/httpie |
| Flask REST | 0.95 | Routes → tools | synthetic fixture |
| FastAPI + Pydantic | 0.95 | Routes + params | synthetic fixture |
| Express.js | 0.95 | Routes → tools | synthetic fixture |
| Spring Boot | 0.95 | Annotations → tools | synthetic fixture |
| Go Gin | 0.95 | Routes → tools | synthetic fixture |
| Go Echo | 0.95 | Routes → tools | labstack/echo |
| Go Chi | 0.90 | Routes → tools | go-chi/chi |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Type-MCP/mcp-anything](https://github.com/Type-MCP/mcp-anything) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
