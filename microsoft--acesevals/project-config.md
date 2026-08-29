---
trigger: always_on
description: > **Naming:** The external name for this project is **ACES** (Agent Capability Evaluation Suite). **SABER** (Security Agent Benchmarking and Evaluation Research) is the internal Microsoft codename. The Python package, CLI commands, and code all use the name `saber`. Both names refer to the same system.
---

```instructions
# ACES / SABER Repository – Copilot Instructions

> **Naming:** The external name for this project is **ACES** (Agent Capability Evaluation Suite). **SABER** (Security Agent Benchmarking and Evaluation Research) is the internal Microsoft codename. The Python package, CLI commands, and code all use the name `saber`. Both names refer to the same system.
>
> **Repositories (GitHub is the permanent home):**
> - **GitHub (permanent home):** [ACESEvals](https://github.com/microsoft/ACESEvals) (benchmarks) + [ACES](https://github.com/microsoft/ACES) (library)
> - **Azure DevOps (deprecated — read-only after 2026-09-30):** [oss_saber](https://dev.azure.com/MSECAIModels/Benchmarking/_git/oss_saber) + [SABER](https://dev.azure.com/MSECAIModels/Benchmarking/_git/SABER). Being retired; do new work on GitHub.

**SABER** is a distributed system for benchmarking agentic workflows in cybersecurity domains using **inspect_ai** integration with **Model Context Protocol (MCP)**.

## Repository Structure

| Path | Purpose |
|------|---------|
| `src/saber/server/` | FastAPI + FastMCP server – REST API, session management, Docker sandbox execution |
| `src/saber/client/` | Client-side – session management, REST client, MCP tool access |
| `src/saber/inspect_ai/` | Inspect AI integration – SABERSandboxEnvironment, agent registry, solvers |
| `src/saber/domain/` | Domain orchestration – manifest loading, Docker compose management |
| `src/saber/models/` | Shared Pydantic models – REST, MCP, evaluation, benchmark task models |
| `domains/` | Benchmark domains – task configurations, prompts, Docker environments |
| `notebooks/` | Analysis notebooks – model comparison, agent architecture analysis, cross-domain aggregation |
| `docs/` | Architecture documentation and diagrams |
| `tests/` | Test suite organized by component |

## Architecture Overview

```
Inspect AI → SABERSandboxEnvironment → ClientSessionManager → REST/MCP Server
                      ↓                        ↓                    ↓
               Agent Solver              MCP Client           SessionManager
                      ↓                        ↓                    ↓
               Tool Execution          Tool Discovery      Docker Sandbox Execution
```

**Key Components**:
- `SABERSandboxEnvironment`: Inspect AI sandbox lifecycle management (task_init, sample_init, sample_cleanup)
- `SessionManager`: Central server orchestrator – sessions, episodes, execution, evaluation
- `ClientSessionManager`: Client-side session and episode management via REST
- `ExecutionManager`: Docker sandbox command execution with security validation
- `EvaluationManager`: Client-side evaluation result storage and validation
- `BenchmarkManager`: YAML-based task and benchmark configuration loading

**Key APIs**:
- REST: `/sessions`, `/episodes`, `/benchmarks`, `/policy`, `/evaluation`
- MCP: Tool discovery and execution via FastMCP server

## Python Environment

- **Use `uv`** – the package manager for this project
- Run commands: `uv run pytest`, `uv run pre-commit`
- Install deps: `uv sync --all-extras` (workspace root)
- **Do NOT** use system Python or create new virtual environments
- Python 3.11-3.12 required (managed by `.python-version`)

## Dependency Management — saber Library

The `saber` library (from the ACES/SABER repo) is installed via git in `pyproject.toml`
using `branch = "main"`, but `uv.lock` pins the **exact commit hash**. This means:

- `uv sync` installs whatever commit is locked, **not** the latest on `main`.
- After changes are pushed to the ACES/SABER library repo, the lock here must be
  explicitly updated:
  ```bash
  uv lock --upgrade-package saber   # resolves latest commit on main
  uv sync --all-extras               # installs it
  ```
  Then commit the updated `uv.lock`.
- **If evals produce unexpected errors** (missing features, broken scoring), the first
  thing to check is whether `uv.lock` is pointing at a stale saber commit.
- Use `uv pip show saber` to see the currently installed commit hash.

> **`inspect-ai` is a REQUIRED fork — do not replace with upstream.** `pyproject.toml`
> pins `inspect-ai` to the GitHub `ACESEvals` branch `inspect-ai/dev/aces_integration`.
> This fork powers the agent harnesses (`-T agent=copilot`, `-T agent=claude_code`) and
> tool_call limits, which are **not** in upstream inspect_ai. Switching to upstream breaks
> the harness / agent-architecture evals (see `notebooks/*agent_architecture_analysis.ipynb`).
> This branch must keep being maintained on GitHub; it is not retired by the ADO deprecation.

> **Agent harnesses run their CLIs inside the sandbox, not on the host.** `-T agent=copilot`
> and `-T agent=claude_code` use inspect_ai's `sandbox_agent_bridge()` to run the GitHub
> Copilot / Claude Code CLIs **inside the Docker sandbox** — the base `saber/sandbox` image
> ships Node 22 + the `claude` CLI + the Copilot SDK — with model calls proxied back to
> inspect_ai's `--model` (BYOK). The **host** only needs the Python SDKs (`uv sync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/ACESEvals](https://github.com/microsoft/ACESEvals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
