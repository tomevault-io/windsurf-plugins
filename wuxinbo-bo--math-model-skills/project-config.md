---
trigger: always_on
description: Math modeling contest skill using functional role-based architecture.
---

# Meta-model-skills-max -- Multi-Agent Policy

## Project Overview

Math modeling contest skill using functional role-based architecture.
The main agent is the sole LLM and orchestrator.
All reasoning happens inside the agent; Python scripts are pure tooling (file I/O, DOCX, figures, state).

## Subagent Authorization

The main agent is authorized to spawn subagents via inline role-play.
Subagents are temporary inference units that directly write their own output files.
No external agent configuration files are used.

### Parallel Spawn (spawn multiple, wait for all)

| Scenario                         | Agents                              | When                                          |
|----------------------------------|-------------------------------------|-----------------------------------------------|
| M001 Problem Consensus Meeting   | planner + analyst                   | Stage 1: parallel perspective gathering       |
| M002 Model Debate                | proposer (accuracy) + proposer (robust) | Stage 4-5: parallel model proposals        |
| Candidate model scoring          | proposer + builder                  | Stage 4: independent scoring                  |
| Multi-perspective paper review   | critic + reviewer                   | Stage 12: independent review                  |

### Sequential Spawn (spawn one, wait, then next)

| Scenario                         | Agents                              | When                                          |
|----------------------------------|-------------------------------------|-----------------------------------------------|
| M001 review                      | reviewer (after planner+analyst finish) | Stage 1: synthesis review                  |
| M002 decision                    | reviewer (after proposer finish)    | Stage 5: model selection                      |
| M004 Red-Team Review             | critic -> reviewer -> writer        | Stage 12-13: adversarial review chain         |
| Model verify                     | builder -> reviewer                 | Stage 6: code + review                        |

### Do NOT Spawn Subagents For

- Final modeling decision (main agent decides)
- Tasks requiring concurrent edits to the same file
- Vague tasks like "solve the whole problem"
- Any task where the main agent can switch roles sequentially
- Stage transitions and state updates
- Simple file operations

## File Ownership Rules

- Each subagent must be told which files it may read and which it may write
- **Sub-agents directly write their own output files** (no main-agent relay)
- No two subagents should write to the same file concurrently
- Main agent owns: `state/`, cross-subquestion merge files (`unified_framework.md`, `cross_validation_report.md`), `summary.md` aggregation
- Subagent outputs go to agent-specific files (e.g., `planner_view.md`, `analyst_view.md`, `outputs/q{N}/summary.md`)
- Parallel sub-question isolation: each sub-agent writes to `outputs/q{N}/` independently

## Python Scripts as Pure Tooling

All scripts in `scripts/` are invoked via `python scripts/<name>.py <args>`.
They perform file operations, state management, DOCX generation, figure plotting, etc.
**None of them call any LLM API.** All reasoning is done by the main agent.

### Stage Execution Protocol (v4.0)

**CRITICAL**: SKILL.md is a compact routing table. Detailed stage instructions are in `stages/S{N}.md`.

**v4.0 Execution Chain (2026-06-17)**:
- 14-stage DAG dependency: S0->S1->S2->S3->S4->S5->S5.5->S6->S7->S7.5->S8->S9->S9.5->S10
- 32 gate contracts, all implemented in gate_contracts.py
- Fast mode total calls <=20, Standard approx 40-50
- Execution protocol: after each stage completes, must run gate_check; on failure, rework until pass

```bash
# First-time: create workspace directory structure
python scripts/workspace_init.py --contest CUMCM --subquestions 3

# Initialize pipeline (with mode selection; auto-creates workspace if missing)
python scripts/stage_executor.py init --contest CUMCM --subquestions 3 --mode championship

# For each stage transition:
python scripts/stage_executor.py current              # Show current stage
python scripts/stage_executor.py begin S3             # Start stage
# Read stages/S3.md and execute all steps
python scripts/stage_executor.py validate S3          # Verify artifacts
python scripts/stage_executor.py gate_check S3        # Run gate checks (v10.0 with decision log)
python scripts/stage_executor.py complete S3          # Mark complete
python scripts/stage_executor.py rework S3 --reason "..." # Rework if gate failed (v10.0)
python scripts/stage_executor.py checklist            # View progress
```

### Script Reference

| Need                             | Command                                               |
|----------------------------------|-------------------------------------------------------|
| **Workspace initialization**     | `python scripts/workspace_init.py [--check] [--contest CUMCM] [--subquestions N]` |
| **Stage execution**              | `python scripts/stage_executor.py <command>`           |
| State machine                    | `python scripts/pipeline_manager.py <command>`         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WuXinbo-bo/Math-model-skills](https://github.com/WuXinbo-bo/Math-model-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
