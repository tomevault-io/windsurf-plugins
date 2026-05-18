---
trigger: always_on
description: Best-practice instructions for AI coding agents working in this repository. Treat this as the repo-specific companion to `README.md`: humans get the product story there; agents get the operating contract here.
---

# AGENTS.md

Best-practice instructions for AI coding agents working in this repository. Treat this as the repo-specific companion to `README.md`: humans get the product story there; agents get the operating contract here.

## Project overview

M2M2 is a rewrite of Math-To-Manim: short educational prompts become typed planning artifacts, generated Manim code, optional renders, review outputs, and a reproducible run bundle.

Core promise: story before symbols, geometry before algebra, artifacts before side effects.

Primary package: `math_to_manim`.
Primary CLI entry points: `m2m2` and `math-to-manim`.
Primary runtime path: `math_to_manim/pipeline/runner.py`.
Architecture reference: `docs/ARCHITECTURE.md`.
Human-facing landing page: `README.md`.

## Agent operating principles

Follow these Karpathy-inspired rules in every change:

1. Think before coding.
   - Do not silently assume requirements, architecture, file ownership, or command behavior.
   - Surface ambiguity when it changes implementation choices.
   - Ask for clarification only when genuinely blocked; otherwise choose the smallest safe interpretation and state the assumption.
   - Present tradeoffs when a request has meaningful complexity, safety, or product implications.

2. Simplicity first.
   - Prefer the smallest maintainable change that satisfies the request.
   - Do not add speculative abstractions, broad configurability, background services, new frameworks, or “future-proofing” unless asked.
   - If a solution grows large, stop and look for a smaller cut before continuing.

3. Surgical changes.
   - Touch only files and lines required for the task.
   - Do not opportunistically rewrite comments, formatting, docs, or adjacent code.
   - Match existing style in the file you are editing.
   - Remove imports/functions/files only when your change made them unused, or when the user explicitly asked for cleanup.
   - Mention unrelated dead code in your final notes instead of deleting it.

4. Goal-driven execution.
   - Define success criteria before editing.
   - For bugs, reproduce the failure or add a failing test first when practical.
   - For features, add or update tests around the changed behavior when practical.
   - Verify with exact commands before final response.

## Repository layout

- `math_to_manim/agents/` — stage adapters for intent, graph, curriculum, math, storyboard, scene spec, codegen, static review, render, video review, and publishing.
- `math_to_manim/schemas/` — Pydantic artifact contracts. Treat these as public pipeline interfaces.
- `math_to_manim/pipeline/` — orchestration, tracing, state, and repair loop behavior.
- `math_to_manim/tools/` — deterministic helpers for graph work, AST/static validation, scene discovery, and artifact storage.
- `math_to_manim/rendering/` — Manim, FFmpeg, and render command wrappers.
- `math_to_manim/providers/` — provider-specific integrations such as the Codex CLI bridge.
- `math_to_manim/app/` — optional API/UI surfaces.
- `tests/unit/` — current automated test suite.
- `docs/` — architecture, docs index, showcase, and visual documentation assets.
- `docs/showcase/assets/` — intentionally tracked legacy showcase GIFs used as art-direction targets.
- `scripts/` — operational helper scripts such as render dependency bootstrap.
- `runs/` — generated run bundles; ignored and normally not committed.

## Setup commands

Use the existing local virtual environment when present:

```bash
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

Fresh checkout on macOS/Linux/WSL:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

Fresh checkout on Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -U pip
python -m pip install -e ".[dev]"
```

Install render extras only when the task requires real Manim rendering:

```bash
python -m pip install -e ".[dev,render]"
./scripts/bootstrap-render.sh  # Debian/Ubuntu/WSL system deps: FFmpeg, LaTeX, etc.
```

## Verification commands

Run the fastest relevant checks before finishing. Prefer the venv-qualified form so results are independent of shell activation:

```bash
./.venv/bin/python -m pytest
./.venv/bin/python -m math_to_manim.cli --help
./.venv/bin/python -m math_to_manim.cli generate --help
./.venv/bin/python -m math_to_manim.cli generate "Explain why derivatives are slopes" --deterministic --no-render --runs-dir /tmp/m2m2-smoke
```

If the CLI entry points are installed in the active environment, these equivalents should also work:

```bash
m2m2 generate "Explain why derivatives are slopes" --deterministic --no-render
math-to-manim generate "Explain why derivatives are slopes" --deterministic --no-render
```

For codegen-provider work, verify Codex separately before blaming M2M2:

```bash
codex --version
codex exec "Say ready from inside this repo"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarleyCoops/Math-To-Manim](https://github.com/HarleyCoops/Math-To-Manim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
