---
trigger: always_on
description: This directory automates creating **Architecture Adapters** for the `TransformerBridge` system. It is a **control plane** — it holds agent definitions, domain knowledge, launch scripts, and tooling. It lives at `devtools/adapter_builder/` inside the TransformerLens repo but is contributor tooling, not part of the shipped package. Agents work in **git worktrees of this repo** created outside the checkout; nothing runs against the main working tree.
---

# TL Adapter Builder

## Purpose

This directory automates creating **Architecture Adapters** for the `TransformerBridge` system. It is a **control plane** — it holds agent definitions, domain knowledge, launch scripts, and tooling. It lives at `devtools/adapter_builder/` inside the TransformerLens repo but is contributor tooling, not part of the shipped package. Agents work in **git worktrees of this repo** created outside the checkout; nothing runs against the main working tree.

For the manual (non-agent-team) path to the same outcome, see the repo's `/add-model-support` slash command — the adapter spec and registration checklist in the repo docs are canonical; this tool's docs cover orchestration only.

## Project Layout

```text
agents/                        Agent definitions and orchestration
  launch-agent-pair.sh         User-facing dispatcher — routes to launch.sh, launch-solo-pair.sh, or ops.sh
  launch.sh                    Launch flow — agent-teams mode (Max tier, orchestrator + subagents)
  launch-solo-pair.sh          Launch flow — solo mode (any tier, two independent sessions, file-based coordination)
  ops.sh                       Ops subcommands (status/logs/attach/send/stop/clean)
  watch-completion.sh          Background watcher — auto-sends /exit on genuine completion
  solo-coordinator.sh          Solo-mode signal router daemon (consumes signals, wakes panes)
  check-signals.sh             Non-blocking signal state snapshot (solo re-orientation)
  solo-programmer.md           Programmer prompt for solo mode (signal + end turn)
  solo-reviewer.md             Reviewer prompt for solo mode (message-driven)
  lib/common.sh                Shared bash helpers (log/ok/warn/err/require_cmd)
  orchestrator.md              Orchestration prompt template ({{PLACEHOLDER}} tokens)
  programmer.md                Programmer agent (3-step lifecycle)
  reviewer.md                  Reviewer agent (5-phase review, Opus model)
  signals.sh                   Signal protocol — single source of truth
  progress.sh                  Progress tracking for crash recovery
  overlord-request.sh          flock-based memory lock for heavy operations
  hooks/                       Claude Code hooks for auto-enforcement
    timeline-capture.sh          All events — structured JSONL logging
    guard-git.sh                 PreToolUse — blocks `git commit`, `git push`, `gh pr create`
    guard-review-rounds.sh       PreToolUse — blocks review files past round 3
    guard-verify-models.sh       PreToolUse — blocks verify_models on >7B or unregistered models
    gate-reviewer-writes-file.sh SubagentStop — blocks reviewer results that have no file artifact
    gate-lint-checks.sh          Stop — blocks exit until mypy+format pass
    notify-on-completion.sh      SessionEnd — fires Slack notification on success
docs/                          Domain knowledge for agents
  adapter-specification.md     What an adapter is and how to build one
  adapter-template.py          Skeleton adapter (Llama-style pattern)
  artifact-templates.md        File templates for all build artifacts (brief, plan, reviews, etc.)
  memory-lock.md               Memory lock protocol (flock-based, run subcommand only)
  hf-model-analysis-guide.md   How to analyze an HF model for adapter creation
  review-specification.md      Five-phase review methodology (source of truth)
scripts/                       Tooling
  analyze-hf-model.py          Analyze HF model config, generate scaffold adapters
  validate-architecture.py     Pre-flight check: is this arch real? (transformers + HF Hub)
  scan-hf-architecture.py      Exhaustive HF scan for all models of an arch class
  port-arch-models.py          Merge per-arch model list into supported_models.json
  validate-adapter.sh          Structural + deep validation of adapters
  validate-adapter-deep.py     Semantic validation against real models (meta device)
  compare-adapters.sh          Structured diff between two existing adapters
  format-timeline.py           Render timeline.jsonl entries for status/logs
  notify.sh                    Slack/iMessage/macOS notification on completion
  strip-adapter.py             Remove an adapter + registrations + registry entries (golden-master rebuild tests)
  dry-run-test.sh              Self-test suite for this project
```

## TransformerLens Repo

- **Path:** the repo containing this checkout, derived automatically (`git rev-parse --show-toplevel`). Override with `--target-repo` or `DEFAULT_TARGET_REPO` in `.env` to drive a different checkout.
- **Key paths:**
  - `transformer_lens/model_bridge/architecture_adapter.py` — base class
  - `transformer_lens/model_bridge/supported_architectures/` — all existing adapters
  - `transformer_lens/model_bridge/generalized_components/` — bridge components
  - `transformer_lens/factories/architecture_adapter_factory.py` — adapter registry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TransformerLensOrg/TransformerLens](https://github.com/TransformerLensOrg/TransformerLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
