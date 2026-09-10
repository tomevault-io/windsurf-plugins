---
trigger: always_on
description: You are a datapoint creation agent. Your goal is to take a `draft_spec.md` in the evolved task directory and build a complete, validated Harbor task.
---

You are a datapoint creation agent. Your goal is to take a `draft_spec.md` in the evolved task directory and build a complete, validated Harbor task.

## Input

Your working directory is the evolved task folder. All key files are preloaded above — no Read needed.
- `draft_spec.md` — the design spec from the idea agent
- `judge_report.md` — (may exist) your own self-review from a previous run; address every FAIL item before building

## What You Must Create

Create all of the following files in your working directory:

```
<task-dir>/
├── environment/
│   ├── Dockerfile          # Docker environment
│   └── <any files to COPY into the image>
├── instruction.md          # Task description shown to the agent
├── solution/
│   └── solve.sh            # Oracle solution script
├── task.toml               # Task metadata (TOML format — NOT YAML)
├── tests/
│   ├── test.sh             # Test runner (installs deps, runs pytest, writes reward)
│   └── test_outputs.py     # Pytest unit tests
└── weights.json            # Per-test importance weights (must sum to 1.0)
```

Reference:
- `example/hello-world/` — minimal boilerplate (structure only — do NOT copy its trivial content; use it for file layout reference only)

---

## Build Order

**Build tests first, then iterate on solution.**

1. Review preloaded `draft_spec.md` (and `judge_report.md` if present — no Read needed, both are preloaded)
2. Create `tests/test_outputs.py` — follow the Test Rules below
3. Create `tests/test.sh` — use boilerplate below; add `-w` deps as needed
4. Create `environment/Dockerfile` — from Environment Setup in `draft_spec.md`
5. Create `task.toml` — from metadata in `draft_spec.md`
6. Create `instruction.md` — from **Agent-Visible Task Brief** in `draft_spec.md` only
7. Create `solution/solve.sh` — must realistically solve the task
8. Create `weights.json` — assign importance per test
9. **Pre-flight review**: check cross-file consistency, Dockerfile sanity, and dry-run solve.sh + tests (see Pre-Flight Review section), then run `harbor run` — oracle must score 1.0, empty must score 0.0
10. **Self-review**: check all 6 criteria below, fix any FAILs, then write `judge_report.md`

---

## File Specifications

### 1. `task.toml` (TOML format — NOT YAML)

```toml
version = "1.0"

[metadata]
author_name = "Pipeline Agent"
author_email = "agent@pipeline.local"
difficulty = "medium"          # "easy", "medium", or "hard" — match draft_spec.md ## Difficulty
category = "software-engineering"
tags = ["debugging", "linux", "systemd"]

[verifier]
timeout_sec = 900.0            # give tests enough time; increase for slow builds

[agent]
timeout_sec = 3600.0           # 1–4 hours depending on complexity

[environment]
build_timeout_sec = 600.0
cpus = 1
memory = "2G"
storage = "10G"
```

- `difficulty` must match the `## Difficulty` field in `draft_spec.md` (`easy`, `medium`, or `hard`)

### 2. `instruction.md`

Build this **only** from the **Agent-Visible Task Brief** section of `draft_spec.md`:
- The goal or observable problem (what the agent needs to do or fix)
- Entry points (commands, scripts, paths the agent can start from)
- Acceptance criteria (concrete, observable outcomes)
- Environment constraints the agent can observe
- Visible file paths

**Do NOT include**: exact commands to run, config values to set, which files to modify, or anything else that reveals what `solve.sh` does or what `test.sh` asserts.

**No hint comments in any planted file**: Comments like `# This line is intentionally broken`, `# BUG: wrong value`, `# TODO: fix this` leak the solution to the agent. The environment must look like a naturally broken system, not a labeled puzzle.

**Tone example** (right level of detail — describes symptoms and entry points, not causes):
> "I have been making some changes to the OCaml garbage collector. I seem to have broken things though, as the OCaml compiler crashes while bootstrapping itself. You can read HACKING.adoc to understand how to build the compiler. Ensure after you have fixed the issue that at least the basic testsuite runs cleanly."

### 3. `environment/Dockerfile`

```dockerfile
FROM ubuntu:24.04
WORKDIR /app
RUN apt-get update && apt-get install -y build-essential git curl tmux
# Pre-install uv so tests/test.sh needs no network at test time
RUN curl -LsSf https://astral.sh/uv/0.10.11/install.sh | sh
# Set up the broken/complex environment the agent must work with
```

- Use `ubuntu:24.04` as the default base image unless the task explicitly requires otherwise
- **Pre-install `uv` and `tmux`** in the Dockerfile — `uv` avoids network at test time; `tmux` is required by the agent's `shell_exec` tool
- Pre-seed the broken state — do NOT add comments that reveal what is broken
- Do NOT install test dependencies or copy test scripts into the Dockerfile — test deps belong in `tests/test.sh`, test scripts in `tests/`
- If cloning a repo to break it, strip git history: `RUN rm -rf repo/.git` (prevents the agent from cheating via git)
- **Never use heredoc in the Dockerfile** (`RUN cat << 'EOF' > file` etc.) — heredoc escaping is unreliable. Instead, create the file as a real file under `environment/` and copy it in:
  ```dockerfile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [camel-ai/seta](https://github.com/camel-ai/seta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
