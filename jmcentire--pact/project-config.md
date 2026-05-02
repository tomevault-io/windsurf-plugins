---
trigger: always_on
description: Contract-first multi-agent software engineering. Decomposition produces contracts and tests, not code. Black-box implementations verified by functional tests at boundaries. Recursive composition.
---

# CLAUDE.md -- Pact

Contract-first multi-agent software engineering. Decomposition produces contracts and tests, not code. Black-box implementations verified by functional tests at boundaries. Recursive composition.

## Quick Reference

```bash
cd ~/Code/pact
python3 -m pytest tests/ -v        # Run all tests
pact init <project-dir>            # Initialize project
pact status <project-dir>          # Show state
pact components <project-dir>      # List components
pact build <project-dir> <id>      # Build specific component
pact run <project-dir>             # Execute pipeline
pact tasks <project-dir>           # Generate/display task list
pact analyze <project-dir>         # Cross-artifact analysis
pact checklist <project-dir>       # Requirements quality checklist
pact assess <directory>            # Architectural assessment (any codebase)
pact export-tasks <project-dir>    # Export TASKS.md
pact handoff <project-dir> <id>    # Render/validate handoff brief
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
2. **Shape** -- (Optional) Produce a Shape Up pitch: appetite, breadboard, rabbit holes, no-gos
3. **Decompose** -- Task -> DecompositionNode tree (2-7 components), guided by shaping context
3. **Contract** -- For each component (leaves first), generate ComponentContract
4. **Test** -- For each contract, generate ContractTestSuite with executable tests + hidden Goodhart tests
5. **Validate** -- Mechanical gate: all refs resolve, no cycles, test code parses
6. **Preflight** -- Establish red lines and contingencies before implementation (Claude Code backend only). Queries Kindex for lessons from previous runs. Stores PreflightPlan per component in `.pact/preflight/`. Skipped for direct API backends.
7. **Implement** -- Each component independently by code_author agent, verified by contract tests
8. **Integrate** -- Parent components: glue code wiring children, parent-level tests
9. **Retrospective** -- Post-run analysis: cost, failure patterns, lessons learned (mechanical, no LLM)
10. **Diagnose** -- On failure: I/O tracing, systematic error recovery

### Execution Modes

Two independent levers:
- `parallel_components: true` -- Independent leaves implement concurrently (semaphore-limited)
- `competitive_implementations: true` -- N agents implement same component, best wins
- `plan_only: true` -- Stop after contracts, use `pact build` to target specific nodes
- `max_concurrent_agents: 4` -- Concurrency limit for parallel modes

### Build Modes

Three build modes control decomposition behavior:

```yaml
# pact.yaml
build_mode: auto    # unary | auto | hierarchy
```

- **unary**: Single agent session. Skips LLM decomposition, creates one component with the full task. Still produces contract+tests for verification.
- **auto** (default): LLM decides whether to decompose or implement directly. Improved prompts genuinely encourage `is_trivial=true` for straightforward tasks.
- **hierarchy**: Always decompose into multiple components (previous default behavior).

Project config overrides global config. Set at runtime via directive:
```bash
pact directive ./my-proj '{"type": "set_mode", "mode": "unary"}'
```

### Global Standards

After decomposition, pact automatically collects shared conventions from contracts:
- Shared types (appearing in 2+ contracts)
- Common validator patterns
- Package requirements
- Coding conventions from SOPs

Standards are injected into every agent's handoff brief and persisted at `standards.json` in the project root.

### Processing Register

Research (Papers 35-39) established that LLM representations follow a hierarchy: register (processing mode) > domain > structural shape. Register is the hub that domain anchors to. Pact makes this explicit:

- **Establishment**: Interview phase determines the processing register before any domain analysis (e.g., "rigorous-analytical", "exploratory-generative", "systematic-verification")
- **Propagation**: Register is a first-class field on `ComponentContract` and flows through the handoff protocol: reset → prime register → prime domain
- **Override**: Set `processing_register` in `pact.yaml` to skip LLM establishment
- **Monitoring**: Health system tracks register drift — agents departing from their established processing mode — as an early indicator of coordination failure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmcentire/pact](https://github.com/jmcentire/pact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
