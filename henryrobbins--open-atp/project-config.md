---
trigger: always_on
description: Developer guide for **open-atp** (Open Automated Theorem Proving). Read this
---

# AGENTS.md

Developer guide for **open-atp** (Open Automated Theorem Proving). Read this
before making changes. The user-facing overview lives in [README.md](README.md);
this file is the engineering reference.

## What this project does

Upload one or more Lean files containing `sorry`, run them through proof-synthesis
backends, and get back **verified** completed proofs with metadata (verification
status, cost, duration). Every prover — including the hosted Aristotle — funnels its
output through one **shared verifier** that compiles the candidate in a Lean+Mathlib
sandbox and checks it compiles, is sorry-free, and is axiom-clean.

### Two primitives + thin generators

1. **`ComputeBackend`** (`backends/`) — run a command over a working directory inside a
   Lean+Mathlib sandbox. Two impls: `DockerBackend`, `ModalBackend`.
2. **`Verifier`** (`verify.py`) — compile a candidate project in a backend and
   report `verified` / `sorry_free` / `axioms`.

```
ComputeBackend (docker | modal)         ← the sandbox primitive
        │
        ├── Verifier  ──────────────────← shared final check (ALL provers)
        │
AutomatedProver (provers/base.py, base)
 ├── AgentProver      coding-agent harness (claude/codex/opencode/axproverbase/vibe) + lean-lsp-mcp
 ├── NuminaProver     configured AgentProver: claude + vendored Numina assets + round loop
 └── AristotleProver  remote `aristotle submit --project-dir --wait` (no local generation sandbox)
```

### Input contract

Submit a **full lake project** (carries `lean-toolchain` + `lake-manifest.json`). The
verifier **rejects** projects whose toolchain doesn't match the sandbox image's pin
(`ToolchainMismatch`) instead of failing deep in a build. The CLI can also take bare
`.lean` files and stage them into the pinned skeleton. One Mathlib image to start
(pinned Lean/Mathlib **v4.28.0**); `image` is a config field so more can be added.

## Project structure (high-level)

```
src/open_atp/
  config.py         standard_prover + STANDARD_PROVERS registry (build provers by name)
  __main__.py       `open-atp prove | benchmark | download | build-docker-image | build-modal-image` CLI
  images/           image name + toolchain pins (DEFAULT_IMAGE, DEFAULT_TOOLCHAIN)
  lean.py           LeanProject, ProofTask, create_project (the Lean input contract)
  verify.py         VerificationReport, Verifier (the shared final check)
  benchmark.py      run_benchmark: run named provers x named tasks, tabulate results
  backends/         base.py  docker.py  modal.py            (ComputeBackend impls)
  provers/          base.py  agent_prover.py  numina.py  numina_tracker.py  aristotle.py
  harness/          coding-agent CLIs staged into the sandbox:
                      base.py  claude_code.py  codex.py  opencode.py
                      axproverbase.py  vibe.py  cost.py  _catalog.py  _numina.py  _paths.py
                    assets/  scripts/*.sh  configs/mcp.json  vibe/lean-standin.toml

images/             Dockerfile (Mathlib base image) + lean/ skeleton (toolchain, lakefile)
vendor/             vendored third-party assets, tracked to upstream SHAs (see VENDOR.md in each)
  numina/             Numina skills + prompts (round-loop prover)
  leanprover-skills/  host-agnostic Lean skills
  lean4-skills/       Claude `lean4` plugin
tests/              pytest suite (+ tests/.runs/ integration artifacts, gitignored)
docs/               Sphinx docs (user_guide/, provers/, agent_harness/, api/)
refs/               read-only symlinks to reference projects (NEVER modify or commit)
```

The README's `Layout` section predates the `harness/` split — trust the tree above.

### Vendored code

`vendor/*` is upstream third-party code pinned to a SHA (each has a `VENDOR.md`).
Ruff is configured with `extend-exclude = ["vendor"]` — **do not reformat or lint
vendored code**, and keep its upstream style. It ships in the wheel via
`force-include` and is resolved at runtime by `harness/_paths.py` (wheel:
`open_atp/vendor/<name>`; checkout: repo-root `vendor/<name>`).

## Provers

Names accepted by the `prove` positional `prover`, the `benchmark --provers` flag,
and the `STANDARD_PROVERS` registry (`config.py`):

| Name | Backing tool | Notes |
| --- | --- | --- |
| `aristotle` | Harmonic Aristotle (hosted) | remote API via `aristotlelib`, no local gen sandbox |
| `claude` | Claude Code (`claude_code` harness) | default; coding agent + lean-lsp-mcp |
| `codex` | OpenAI Codex CLI | model `gpt-5.5` |
| `opencode` | opencode | |
| `axproverbase` | ax-prover (LangGraph) | proposer→builder→reviewer loop; default model `claude-opus-4-8`, effort `high` |
| `numina` | Numina skills/prompts on Claude Code | round-continuation loop |
| `leanstral` | Mistral Vibe `lean` scaffold | hosted model (default `magistral-medium-latest`), no GPU; `--model` configurable |

Agentic harnesses share **lean-lsp-mcp** as their LSP server. The shared `Verifier`
does the final compile/sorry/axiom check regardless of which tool generated the proof.

## Tooling

- **Python ≥ 3.12**, packaged with **hatchling**, deps managed by **uv** (`uv.lock`).
- **ruff** — lint (`E,F,I,UP`) + format, line length 88, excludes `vendor`.
- **mypy** — `strict`, `files = ["src/open_atp"]`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henryrobbins/open-atp](https://github.com/henryrobbins/open-atp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
