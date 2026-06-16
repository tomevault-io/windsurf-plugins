---
trigger: always_on
description: Guidance for AI agents working on this repo. Keep this file under ~200 lines.
---

# CLAUDE.md

Guidance for AI agents working on this repo. Keep this file under ~200 lines.

## What coresmith is

AI-orchestrated ASIC pipeline. Three LangGraph state machines run in sequence:

1. **Architecture** (`orchestrator/architecture/`, `orchestrator/langgraph/architecture_graph.py`) — PRD → block diagram → SAD/FRD/ERS → constraint check → final-review gate.
2. **Frontend pipeline** (`orchestrator/langgraph/pipeline_graph.py`) — per-block loop: generate uArch spec → generate RTL → Verilator lint → generate cocotb TB → simulate → synthesize → diagnose/retry on failure. After each tier: uArch integration review. After all tiers: integration check (chip_top.v generation + lint) → integration DV → validation DV.
3. **Backend** (`orchestrator/langgraph/backend_graph.py`) — PnR, DRC, LVS, signoff. Driven via MCP, not headless.

The pipeline is **LLM-driven**, not deterministic. Every code-gen + diagnose step calls an LLM (Claude or Codex). The LLM has shell/file-edit tools and writes RTL/TB directly to disk.

## Architecture: coresmithd daemon + outer agent

coresmith is a **daemon**, not a CLI. There is no `run_pipeline.py` auto-approver any more. The flow is:

1. **`orchestrator/daemon/server.py`** — FastAPI service, one process per `CORESMITH_PROJECT_ROOT`. Wraps the LangGraph via `orchestrator/graph_lifecycle.py`. Writes `<project_root>/.coresmith/daemon.json` ({port, pid}) on startup so clients can discover it. Endpoints: `POST /run/start`, `GET /run/state`, `POST /run/resume`, `POST /run/pause`, `GET /healthz`.
2. **`bin/coresmith`** — CLI client. Auto-discovers the daemon via daemon.json and shells to its HTTP endpoints. Subcommands: `daemon start|stop|status`, `run start|pause`, `state`, `resume`, `logs`.
3. **Outer agent (Claude on cron)** — the **autochecker**. The pipeline parks on every interrupt — it never auto-approves. A scheduled Claude invocation runs `coresmith state`, inspects the pending interrupt's payload, decides, and runs `coresmith resume --action ...`. If the daemon is dead or the run is stuck, the agent restarts it.

```
┌────────────────┐     HTTP      ┌─────────────────────┐     LangGraph
│  bin/coresmith │ ◀──────────▶ │    coresmithd       │ ◀──────────────▶ pipeline_graph
│  CLI           │              │  (FastAPI daemon)   │                  + SQLite checkpoint
└────────────────┘              └─────────────────────┘
        ▲                                ▲
        │ exec                           │ daemon.json (port, pid)
┌───────┴──────────────────────────┐    │
│ Claude on cron (10-min cadence)  │────┘
│  - reads escalation state         │
│  - decides + resumes              │
│  - fixes/restarts on failure      │
└───────────────────────────────────┘
```

## How to run

```bash
# Pick / create the run directory
RUN_DIR=/home/ubuntu/coresmith-runs/<task>-<flavor>-$(date +%Y%m%d-%H%M%S)
mkdir -p $RUN_DIR/inputs && cp <spec>.md $RUN_DIR/inputs/requirements.md

# Export the project root + LLM provider env vars (Codex shown; Claude also works)
export CORESMITH_PROJECT_ROOT=$RUN_DIR
export CORESMITH_LLM_PROVIDER=codex
export CORESMITH_CODEX_MODEL=gpt-5.5
export CORESMITH_MODEL=gpt-5.5
export CORESMITH_BLOCK_MODEL=gpt-5.5
export CORESMITH_CODEX_SANDBOX=danger-full-access
export CORESMITH_SKIP_SYNTH=1            # unless Sky130 PDK is local
export CORESMITH_ENABLE_MEMORY_MAP=0
export CORESMITH_ENABLE_CLOCK_TREE=0
export CORESMITH_ENABLE_REGISTER_SPEC=0

# Start the daemon
bin/coresmith daemon start --project-root $RUN_DIR

# (1) Architecture phase — generates PRD, ERS, FRD, block_diagram.json
#     and the initial arch/uarch_specs/<block>.md files from a
#     natural-language requirements doc. Parks at PRD review, block
#     diagram review, constraint check, etc.; outer agent resumes
#     each via `coresmith resume`.
bin/coresmith architecture start --project-root $RUN_DIR \
    --requirements $RUN_DIR/inputs/requirements.md

# (2) Frontend pipeline — runs against the architecture artifacts from (1)
#     OR against a hand-written examples/<design>/blocks.yaml if you are
#     intentionally skipping the architecture phase. Skipping is supported
#     but the daemon will print a warning at /run/start because
#     `integration_review` then can't verify cross-block data_width and
#     `validation_dv` soft-aborts on missing ERS. Set
#     CORESMITH_SKIP_ARCH_WARN=1 to silence the warning for rapid
#     iteration / PPABench-style runs where you only care about the
#     per-block frontend loop.
bin/coresmith run start --project-root $RUN_DIR \
    --blocks-file /home/ubuntu/coresmith/examples/<design>/blocks.yaml

# Inspect / drive
bin/coresmith state --project-root $RUN_DIR
bin/coresmith resume --project-root $RUN_DIR --action approve
bin/coresmith logs  --project-root $RUN_DIR -n 100
```

`model_reasoning_effort = "high"` is set globally in `~/.codex/config.toml`. `/home/ubuntu` is already trusted there, so subdirs need no extra entry.

**Always use an isolated run dir** — never start the daemon against the repo root. Convention:

```
/home/ubuntu/coresmith-runs/<task>-<flavor>-<YYYYMMDD-HHMMSS>/
  inputs/requirements.md     # copy of the spec
  blocks.yaml                # design-specific blocks override (optional)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookexperimental/coresmith](https://github.com/facebookexperimental/coresmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
