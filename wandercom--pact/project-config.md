---
trigger: always_on
description: Contract-first multi-agent software engineering. Decomposition produces contracts and tests, not code. Black-box implementations verified by functional tests at boundaries. Recursive composition.
---

# CLAUDE.md -- Pact

Contract-first multi-agent software engineering. Decomposition produces contracts and tests, not code. Black-box implementations verified by functional tests at boundaries. Recursive composition.

## Quick Reference

```bash
cd ~/WanderRepos/repos/pact
python3 -m pytest tests/ -v        # Run all tests
pact init <project-dir>            # Initialize project
pact init <project-dir> --spec <file> # Initialize from AI-authored JSON/YAML build spec
pact spec apply <project-dir> <file> # Apply AI-authored JSON/YAML build spec
pact status <project-dir>          # Show state
pact components <project-dir>      # List components
pact build <project-dir> <id>      # Build specific component
pact run <project-dir>             # Execute pipeline
pact tasks <project-dir>           # Generate/display task list
pact analyze <project-dir>         # Cross-artifact analysis
pact checklist <project-dir>       # Requirements quality checklist
pact production init <project-dir> # Scaffold optional production-readiness pack
pact production fingerprint <project-dir> # Print the source fingerprint for evidence
pact production validate <project-dir> # Validate production-readiness gate for a Pact-managed project
pact assess <directory>            # Architectural assessment (any codebase)
pact export-tasks <project-dir>    # Export TASKS.md
pact handoff <project-dir> <id>    # Render/validate handoff brief
pact review <target> --claim <text> # Advocate + Simulacrum review
pact agent spec-author [--apply]    # Run one constrained spec-author agent
pact agent repair --source-root <dir> [--apply] # Run one constrained repair agent
pact directive <project-dir> <json> # Send structured directive to daemon
pact mcp-server [--project-dir <dir>] # Run MCP server (stdio)
pact-mcp                              # MCP server entry point
```

**Entry point**: `pact = "pact.cli:main"`, `pact-mcp = "pact.mcp_server:main"` (pyproject.toml)

**Python**: >=3.12 | **Dependencies**: pydantic>=2.0, pyyaml>=6.0 | **Optional**: anthropic>=0.40, mcp>=1.0

## Architecture Overview

### Research-First Agent Protocol

Every agent follows 3 phases: Research -> Plan+Evaluate -> Execute. Research and plan outputs are persisted alongside work products.

### Core Workflow

1. **Interview** -- Establishes processing register (cognitive mode), then identifies risks/ambiguities, asks user clarifying questions
   and confirms the readiness profile for operational maturity, security,
   privacy, compliance, gating, testing, and monitoring.
2. **Shape** -- (Optional) Produce a Shape Up pitch: appetite, breadboard, rabbit holes, no-gos
3. **Decompose** -- Task -> DecompositionNode tree (2-7 components), guided by shaping context
3. **Contract** -- For each component (leaves first), generate ComponentContract
4. **Test** -- For each contract, generate ContractTestSuite with executable tests + hidden Goodhart tests
5. **Validate** -- Mechanical gate: all refs resolve, no cycles, test code parses
6. **Preflight** -- Establish red lines and contingencies before implementation (iterative coding-shell backends). Queries Kindex for lessons from previous runs. Stores PreflightPlan per component in `.pact/preflight/`. Skipped for direct API backends.
7. **Implement** -- Each component independently by code_author agent, verified by contract tests
8. **Integrate** -- Parent components: glue code wiring children, parent-level tests
9. **Retrospective** -- Post-run analysis: cost, failure patterns, lessons learned (mechanical, no LLM)
10. **Diagnose** -- On failure: I/O tracing, systematic error recovery

### Production-Readiness Pack

`pact production` is an explicit opt-in layer for higher-bar builds. It does
not alter the default planner or implementation flow. Instead it scaffolds
file-backed artifacts under `production/` and validates them against Pact's
existing outputs:

- `trust_policy.yaml` for machine-checkable trust assertions
- `control_matrix.yaml` for control-to-evidence mapping
- `threat_model.yaml` for threat and mitigation coverage
- `architecture_laws.yaml` for hard implementation invariants
- `preflight.yaml` for red lines and fallback
- `live_validation.yaml` for running-instance evidence
- `done_gate.yaml` plus `na_register.yaml` for falsifiable readiness status

Derived evidence comes from project config, Constrain bundle, contracts,
contract tests, Goodhart tests, analysis, checklist, review, certification, and
stub scan. It also runs deterministic static checks inspired by webprobe's
mechanical audit model: likely hard-coded secrets, dependency manifest, SBOM,
and OpenAPI validity/auth/error/rate-limit shape. Static checks are never live
browser, network, or LLM probes; missing optional artifacts are reported as
`not_detected`, not as false failures. External evidence must be explicit. N/A
across the pack requires a matching structured record and review reference. Set
`production_artifact_dir` when the tracked pack should live somewhere other
than `production/`; an unset `constrain_dir` follows that configured directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wandercom/pact](https://github.com/wandercom/pact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
