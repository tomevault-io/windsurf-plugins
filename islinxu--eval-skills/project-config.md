---
trigger: always_on
description: >-
---


# eval-skills

AI Agent Skill unit testing framework — a framework-agnostic toolkit for discovering, scaffolding, selecting, evaluating, and reporting on AI skills.

This skill fills the **L1 (Skill Unit Test)** gap that LangSmith / DeepEval leave open: while those platforms focus on agent-level and trajectory-level evaluation (L2-L3), eval-skills targets the individual skill level, ensuring each building block meets quality standards before it ever enters an agent pipeline.

## When to Use This Skill

- **Before deploying a new skill to production** — run `eval` to verify it meets your quality gate.
- **When choosing between multiple candidate skills** — run `select` to rank them on the same benchmark.
- **When a skill is upgraded** — run `report diff` to detect regressions.
- **In CI/CD** — use `--exit-on-fail` to block merges that degrade skill quality.
- **When bootstrapping a new skill** — run `create` to generate a ready-to-fill skeleton.

## Capabilities

### 1. Find Skills

Search for existing skills by keyword, tag, or adapter type.

```bash
eval-skills find \
  --query "web search" \
  --tag retrieval api \
  --adapter http \
  --min-completion 0.8 \
  --skills-dir ./skills \
  --limit 10
```

| Option | Description | Default |
|--------|-------------|---------|
| `-q, --query <string>` | Keyword search (matches name, description, tags) | — |
| `-t, --tag <tags...>` | Filter by tags (intersection: skill must have ALL specified tags) | — |
| `-a, --adapter <type>` | Filter by adapter type (`http`, `subprocess`, `mcp`) | — |
| `--min-completion <rate>` | Minimum historical completion rate (0.0 ~ 1.0) | — |
| `--skills-dir <dir>` | Directory to scan for `skill.json` files | `./skills` |
| `--limit <n>` | Maximum number of results | `20` |

Results are ranked by search relevance (when `--query` is provided) or by historical completion rate (descending).

### 2. Create Skills

Generate a skill skeleton from a template to bootstrap development.

```bash
eval-skills create \
  --name my_api_skill \
  --from-template http_request \
  --output-dir ./skills \
  --description "Fetches weather data from OpenWeather API"
```

| Option | Description | Default |
|--------|-------------|---------|
| `--name <name>` | **Required.** Skill name | — |
| `--from-template <tpl>` | Template type: `http_request`, `python_script`, `mcp_tool` | `http_request` |
| `--output-dir <dir>` | Output directory | `./skills` |
| `--description <text>` | Human-readable description embedded in `skill.json` | — |

Generated file structure:

```
skills/my_api_skill/
  skill.json            # Skill metadata (id, schemas, adapter config)
  adapter.config.json   # Adapter-specific configuration
  tests/
    basic.eval.json     # A starter benchmark with one sample task
  skill.py              # (python_script template only) JSON-RPC entrypoint
```

### 3. Evaluate Skills

Run benchmark evaluations against one or more skills. This is the core command.

```bash
eval-skills eval \
  --skills ./skills/calculator/skill.json ./skills/search/ \
  --benchmark coding-easy \
  --concurrency 4 \
  --timeout 30000 \
  --retries 2 \
  --runs 3 \
  --evaluator exact \
  --format json markdown html \
  --output-dir ./reports \
  --exit-on-fail --min-completion 0.8 \
  --store ./eval-skills.db
```

| Option | Description | Default |
|--------|-------------|---------|
| `--skills <paths...>` | **Required.** Skill file(s) or directory(ies) | — |
| `--benchmark <id\|path>` | Built-in benchmark ID or path to `benchmark.json` | `coding-easy` |
| `--tasks <file>` | Custom tasks JSON file (replaces benchmark) | — |
| `--concurrency <n>` | Number of parallel task executions | `4` |
| `--timeout <ms>` | Per-task timeout in milliseconds | `30000` |
| `--retries <n>` | Retry count on task failure (with incremental backoff) | `0` |
| `--runs <n>` | Repeat evaluation N times for consistency scoring | `1` |
| `--evaluator <type>` | Default scorer type (see Scorer Types below) | `exact` |
| `--format <formats...>` | Output formats: `json`, `markdown`, `html` | `json markdown` |
| `--output-dir <dir>` | Report output directory | `./reports` |
| `--exit-on-fail` | Exit with code 1 if any skill falls below threshold | disabled |
| `--min-completion <rate>` | Threshold for `--exit-on-fail` | `0.7` |
| `--dry-run` | Validate configuration only; do not execute tasks | disabled |
| `--benchmarks-dir <dir>` | Directory containing built-in benchmarks | `./benchmarks` |
| `--store <path>` | SQLite database path for persistent result storage | `./eval-skills.db` |
| `-c, --config <path>` | Path to `eval-skills.config.yaml` | auto-detected |

**Evaluation flow:**

1. Load skills from `--skills` paths (supports both single `skill.json` and directories)
2. Load benchmark tasks from `--benchmark` or `--tasks`
3. Build the cartesian product: `skills x tasks x runs`
4. Execute all task items concurrently (controlled by `--concurrency`, with timeout and retry)
5. Score each result using the appropriate scorer
6. Aggregate into `SkillCompletionReport` per skill
7. Write reports to `--output-dir`

### 4. Select Skills

Filter and rank skills based on evaluation reports using a multi-dimensional strategy.

```bash
eval-skills select \
  --from ./skills \
  --reports ./reports/eval-result.json \
  --strategy ./strategy.yaml \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isLinXu/eval-skills](https://github.com/isLinXu/eval-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
