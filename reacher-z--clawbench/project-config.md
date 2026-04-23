---
trigger: always_on
description: This file is for coding agents (Claude Code, Cursor, Copilot, etc.) to understand and operate on the ClawBench project. If you are a human, see the [README](README.md).
---

# ClawBench -- Agent Context

This file is for coding agents (Claude Code, Cursor, Copilot, etc.) to understand and operate on the ClawBench project. If you are a human, see the [README](README.md).

## What This Is

ClawBench is a benchmarking framework for evaluating AI web agents on 153 real-world online tasks spanning 144 live websites and 15 life categories. Each task runs in an isolated Docker container with Chromium, a recording Chrome extension, and an AI agent harness (`openclaw`, `opencode`, `claude-code`, `codex`, or `browser-use`, selectable via `--harness`). The framework captures five layers of data: session replay (MP4), action screenshots, HTTP traffic, browser actions, and agent messages.

## Project Structure

```
ClawBench/
  run.sh                          # Entry point -- launches interactive TUI
  Dockerfile.base                 # Base image (Chromium + extension + Node + uv, harness-agnostic)
  Dockerfile.openclaw             # Layer that adds the openclaw CLI on top of base
  Dockerfile.opencode             # Layer that adds opencode + @playwright/mcp on top of base
  Dockerfile.claude-code          # Layer that adds Claude Code + @playwright/mcp + LiteLLM on top of base
  Dockerfile.codex                # Layer that adds @openai/codex@0.120 + @playwright/mcp + LiteLLM on top of base
  Dockerfile.browser-use          # Layer that adds browser-use Python framework + LiteLLM on top of base
  entrypoint.sh                   # Shared infra (Xvfb, Chrome, noVNC, human mode); execs /run-harness.sh in agent mode
  setup-openclaw.sh               # Generates ~/.openclaw/openclaw.json from env vars (called from run-openclaw.sh)
  setup-opencode.sh               # Generates ~/.config/opencode/opencode.json from env vars (called from run-opencode.sh)
  setup-claude-code.sh            # Configures API keys + LiteLLM proxy from env vars (called from run-claude-code.sh)
  setup-codex.sh                  # Generates ~/.codex/config.toml (wire_api=responses + playwright MCP) + /tmp/litellm-config.yaml from env vars
  setup-browser-use.sh            # Generates /tmp/browser-use-env.sh + /tmp/litellm-config.yaml (routes via LiteLLM)
  run-openclaw.sh                 # Per-harness agent runner; copied into clawbench-openclaw image as /run-harness.sh
  run-opencode.sh                 # Per-harness agent runner; copied into clawbench-opencode image as /run-harness.sh
  run-claude-code.sh              # Per-harness agent runner; copied into clawbench-claude-code image as /run-harness.sh
  run-codex.sh                    # Per-harness agent runner; copied into clawbench-codex image as /run-harness.sh
  run-browser-use.sh              # Per-harness agent runner; copied into clawbench-browser-use image as /run-harness.sh
  run-browser-use-agent.py        # Python entry script invoked by run-browser-use.sh — always uses ChatOpenAI pointed at LiteLLM
  .env.example                    # Template for PurelyMail credentials
  models/
    models.yaml                   # Model API configs (gitignored -- copy from example)
    models.example.yaml           # Template with placeholder keys
    model.schema.json             # JSON schema for model entries
  test-cases/                     # 153 task directories
    task.schema.json              # JSON schema for task.json
    001-daily-life-food-uber-eats/
      task.json                   # Task instruction, eval schema, time limit
    ...
  test-driver/
    tui.py                        # Interactive TUI (called by run.sh)
    run.py                        # Single test-case runner
    batch.py                      # Batch runner (model x case cross-product)
    generate_resume_pdf.py        # Resume PDF generator
    README.md                     # Full test driver documentation
  extension-server/
    server.py                     # FastAPI data collection server
    README.md                     # Server documentation
  chrome-extension/               # Recording extension (content.js, background.js)
  shared/
    alex_green_personal_info.json # Synthetic user profile template
  eval/
    README.md                     # Evaluation guide + Claude Code prompt template
    agentic_eval.md               # Evaluator rubric for judging agent success
```

## Setup

**Prerequisites:** Python 3.11+, [uv](https://docs.astral.sh/uv/), Docker or Podman.

```bash
# 1. Configure PurelyMail credentials (disposable email for each run)
cp .env.example .env
# Edit .env: set PURELY_MAIL_API_KEY and PURELY_MAIL_DOMAIN

# 2. Configure at least one model
cp models/models.example.yaml models/models.yaml
# Edit models.yaml: add API key and base URL for your model(s)

# 3. Launch
./run.sh
```

## Common Commands

```bash
# Interactive TUI (recommended):
./run.sh

# Single run:
uv run --project test-driver test-driver/run.py test-cases/<case-dir> <model-name> --harness openclaw

# Single run with Claude Code harness:
uv run --project test-driver test-driver/run.py test-cases/<case-dir> <model-name> --harness claude-code

# Batch run (model x case cross-product):
uv run --project test-driver test-driver/batch.py \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reacher-z/ClawBench](https://github.com/reacher-z/ClawBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
