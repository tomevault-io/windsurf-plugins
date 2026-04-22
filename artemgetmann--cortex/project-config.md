---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Cortex?

Cortex is an AI agent project focused on persistent memory across sessions. The active track in this branch is the CLI Memory V2 lab (`tracks/cli_sqlite/`), where the agent learns from failures and recovers faster over repeated runs.

The original FL Studio computer-use track is preserved as historical context under `docs/archive/fl-studio-legacy/`.

**Hackathon project:** Anthropic "Built with Opus 4.6" (Feb 10-16, 2026).

## Project North Star (Important)

The goal is **not** to build the best FL Studio autopilot.

The goal is to test whether lessons improve agent performance over repeated attempts and whether those gains generalize across domains/tasks. FL Studio is one stress-test environment for computer-use, not the product end-state.

When evaluating progress, prioritize:
- Does the agent avoid repeating the same mistakes after failures?
- Do post-task lessons/patches get applied and produce measurable lift?
- Does learning transfer beyond one UI/task shape (especially to CLI Memory V2 domains)?

If FL benchmarks are unstable due to environment issues (lock screen, hidden window, capture failures), treat those runs as infra-invalid and do not use them as evidence against learning quality.

## Commands

```bash
# Install dependencies (Python 3.11+, macOS only)
pip install -r requirements.txt

# Configure
cp .env.example .env   # then set OPENAI_API_KEY

# Run an agent session
python3 scripts/run_agent.py \
  --task "Create a 4-on-the-floor kick drum pattern" \
  --session 2201 --max-steps 80 --verbose

# Run without skills (baseline comparison)
python3 scripts/run_agent.py --task "..." --session 1 --no-skills

# Override model
python3 scripts/run_agent.py --task "..." --model claude-haiku-4-5

# Use subscription-backed Claude CLI transport (no API key needed for executor loop)
python3 scripts/run_agent.py --task "..." --llm-backend claude_print
```

There is no formal lint/build pipeline yet. Verify runtime behavior with agent sessions and check `sessions/session-NNN/` artifacts (`events.jsonl`, `metrics.json`, `step-NNN.png`).

Test suites:

```bash
# Root FL/runtime smoke tests
python3 -m pytest tests -q

# Active Memory V2 CLI track tests
python3 -m pytest tracks/cli_sqlite/tests -q
```

## API Limit Policy (Execution Priority)

- Default benchmark/runtime path is `--llm-backend openai` (API) for speed and cost control.
- If API quota/rate limits are hit during active collaboration, stop and notify the user immediately.
- Do not silently switch to `claude_print`; it is typically much slower for benchmark iteration.
- Only use `claude_print` when the user explicitly asks for it, or for unattended overnight runs.

## Architecture

```
agent.py          ← FL Studio loop orchestrator
computer_use.py   ← macOS Quartz CGEvent wrapper (key, click, screenshot, coordinate mapping)
config.py         ← Env-based config loader (CortexConfig dataclass)
memory.py         ← Session path management + JSONL/metrics I/O
consolidate.py    ← Post-session skill generation from logs (stub, not yet implemented)
claude_print_runtime.py ← Shared Claude Print backend helpers (JSON parsing, model/effort/env resolution)
claude_print_client.py  ← Anthropic-compatible client shim over `claude -p`

scripts/
  run_agent.py    ← CLI entry point (argparse → run_agent())

skills/fl-studio/ ← Markdown skill docs loaded into context
  basics/SKILL.md
  drum-pattern/SKILL.md
  drum-pattern/CONTRACT.json

sessions/         ← Per-session output (gitignored)
  session-NNN/    ← events.jsonl + metrics.json + step-NNN.png screenshots

docs/
  README.md                         ← Canonical docs index (active vs archive)
  MEMORY-V2-EXECUTION-PLAN.md      ← Living execution plan
  MEMORY-V2-AGNOSTIC-PLAN.md       ← Memory V2 requirements + status
  MEMORY-V2-BENCHMARKS.md          ← Benchmark runbook
  MEMORY-V2-CURRENT-FLOW.html      ← Current runtime diagram
  archive/                          ← Historical docs only (not source of truth)
```

### Data flow

1. `run_agent()` builds system prompt + loads skills from `skills/fl-studio/` into context
2. Sends task to OpenAI API (`llm-backend=openai`) or Anthropic API/Claude CLI when explicitly selected
3. Model returns `tool_use` blocks (screenshot, key, click, etc.)
4. `ComputerTool.run()` executes via macOS Quartz CGEvent APIs, returns screenshot
5. Loop continues until model stops requesting tools or hits `max_steps`
6. Events logged to JSONL, metrics written to JSON, screenshots saved as PNGs

### Key design decisions

- **No database/vector store.** Opus 4.6 has 1M token context — skills loaded directly into prompt.
- **Prompt caching** on system blocks + recent user turns (~80% cost reduction on repeated context).
- **Quartz CGEvent APIs** (not pyautogui) for reliable macOS input delivery.
- **Bundle ID matching** (`com.image-line.flstudio`) to find FL Studio, not window title.
- **Coordinate mapping:** API operates in 1024x768 space, mapped to FL Studio window bounds at runtime.
- **UI settle detection:** Post-action screenshot polling with image similarity threshold prevents race conditions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artemgetmann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
