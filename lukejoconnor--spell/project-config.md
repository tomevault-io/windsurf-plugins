---
trigger: always_on
description: This file is a public orientation guide for agents, readers, and contributors working through the Spell source tree. Start with `README.md` for the human-facing project overview, CLI usage, and core language ideas, then use this file for installation checks, source-map lookup, tests, and implementation orientation.
---

# Spell Source Guide

This file is a public orientation guide for agents, readers, and contributors working through the Spell source tree. Start with `README.md` for the human-facing project overview, CLI usage, and core language ideas, then use this file for installation checks, source-map lookup, tests, and implementation orientation.

Current release state: `v0.2.0` is unreleased. The `v0.2.0-dev` branch is the active public-release preparation branch, with the public Clojure API and configuration surface documented in `docs/api.md`.

Spell is a Lisp dialect for LLM self-orchestration. A Spell completion is itself a program: the evaluator runs the program, and the program can call back into an LLM, spawn sub-agents, manage context, and use configured namespaces such as `io`, `web`, `agents`, `globals`, and `patterns`.

## Repo Skills

This repo includes Spell-specific skills under `.agents/skills/`. Use them as the first stop for task-oriented work:

- `spell-setup`: install/check prerequisites, configure provider credentials, and run the first smoke tests or examples.
- `spell-agent-config`: create or modify model profiles and agent profiles; use `docs/api.md` as the canonical API/config reference.
- `spell-developer`: navigate and modify the Spell source code, choose relevant files, and run focused checks.

## Terminology

- Edit marker: a source form, such as `prune`, `rethink`, or `persist`, that affects how `apply-edits` rewrites a completion for a later turn.
- Edit time: the phase when `apply-edits` applies edit markers to a completion before it is used as a model prefix.

## Top-Level Layout

| Path | Purpose |
| --- | --- |
| `bin/spell` | Shell wrapper around the Clojure CLI entry point. |
| `deps.edn` | Clojure dependencies and aliases, including `:run`, `:test-fast`, and `:test-slow`. |
| `src/spell/` | Main Spell implementation. |
| `config/` | Runtime agent, provider, prompt, web, and Spell library configuration. |
| `examples/` | Runnable `.spl` examples plus short writeups for selected examples. |
| `test/` | Unit and integration tests. |
| `data/pricing.edn` | Model pricing table used for usage and cost reporting. |
| `docs/` | Public documentation for the release. |
| `docs/CHANGELOG.md` | Release notes; `v0.2.0` is currently unreleased. |
| `LICENSE` | MIT license text. |

## Agent Quick Start

Prerequisites:

- Java 11+
- Clojure CLI (`clj`)
- At least one LLM provider credential or local provider

On macOS with Homebrew:

```bash
brew install clojure/tools/clojure
```

Provider setup:

- Codex tool-call provider, the CLI default: install the OpenAI Codex CLI and run `codex` once so `~/.codex/auth.json` exists.
- Anthropic API: set `ANTHROPIC_API_KEY`.
- OpenAI API: set `OPENAI_API_KEY`.
- Fireworks API: set `FIREWORKS_API_KEY`.
- Ollama: run a local Ollama server and pass an `ollama:<model>` model spec.

Install from a fresh checkout:

```bash
git clone https://github.com/lukejoconnor/spell.git
cd spell
```

No build step is required for normal CLI use. The `bin/spell` wrapper runs the Clojure CLI entry point with `clj -M:run`.

Smoke test without making an LLM API call:

```bash
bin/spell -h
bin/spell -t "Return a short greeting"
```

The `-t` flag uses the test provider and is useful for checking Java, Clojure, dependencies, and CLI wiring.

## Core Runtime Files

| Path | Purpose |
| --- | --- |
| `src/spell/cli.clj` | CLI argument parsing, model/provider selection, trace/log options, and execution dispatch. |
| `src/spell/eval.clj` | Main evaluator, special forms, environment threading, futures, self-calls, context-management forms, and namespace lookup. |
| `src/spell/parse.clj` | Reader/parser entry points for Spell forms. |
| `src/spell/grammar.clj` | Parenthesis and delimiter grammar checks. |
| `src/spell/format.clj` | Formatting helpers for Spell forms. |
| `src/spell/macros.clj` | Macro registry and macro expansion. |
| `src/spell/runtime.clj` | Agent boxes, registry, `spawn`, `ask`, `send`, notifier flow, and completion coordination. |
| `src/spell/llm.clj` | LLM request construction, prompt prefix handling, suffix cleanup, and inbox pipeline. |
| `src/spell/provider.clj` | Provider implementations for Anthropic, OpenAI, Codex CLI, Fireworks, Ollama, user, and test modes. |
| `src/spell/agent.clj` | Agent definition loading, inheritance, namespace resolution, and provider default wiring. |
| `src/spell/prompt.clj` | System prompt composition from namespace metadata. |
| `src/spell/recovery.clj` | Recovery prompts and retry helpers for malformed completions. |
| `src/spell/trace.clj` | Execution trace recording. |
| `src/spell/trace_tool.clj` | Developer tooling for inspecting recorded traces. |
| `src/spell/api.clj` | Programmatic entry point used by library callers; API details are documented separately. |

The public API/configuration reference is `docs/api.md`. In `v0.2.0-dev`, `spell.api/run` requires `:model-profile` and `:agent-profile`, and rejects old public run keys such as `:provider`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukejoconnor/spell](https://github.com/lukejoconnor/spell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
