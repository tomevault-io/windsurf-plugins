---
trigger: always_on
description: CLI for evaluating [Agent Skills](https://agentskills.io/home) - a format for extending AI agent capabilities. Runs standalone or as a GitHub Action.
---

# CLAUDE.md

CLI for evaluating [Agent Skills](https://agentskills.io/home) - a format for extending AI agent capabilities. Runs standalone or as a GitHub Action.

## Key Files

- `src/cli.ts` - CLI entry point (run, score, report, validate, create-task, import, export, cache)
- `src/types.ts` - TypeScript interfaces
- `src/config.ts` - Centralized config (file + env + CLI precedence)
- `src/task/schema.ts` - Task-package frontmatter schema (checks, verifier, metadata)
- `src/task/load.ts` - Task-package loader/validator (task.md frontmatter + prompt body)
- `src/task/scaffold.ts` - `create-task` scaffolding (task.md, verifier, oracle stubs)
- `src/task/import-skillsbench.ts` + `src/task/export-skillsbench.ts` - SkillsBench/BenchFlow interop (tolerant import, native-package export)
- `src/pipeline.ts` - Full pipeline orchestrator (load → workspace → run → verify → score → report)
- `src/run/workspace.ts` - Per-trial throwaway workspaces (seed files + skills mount)
- `src/score/verifier.ts` - Cross-platform verifier/oracle executor (reward contract; routes to docker when `--sandbox docker`)
- `src/sandbox/docker.ts` - Docker verifier sandbox (verifier-only containerization; SkillsBench reward.txt convention)
- `src/runner/claude-sdk-runner.ts` - Claude Agent SDK runner
- `src/runner/claude-code-runner.ts` - Claude Code CLI runner (stream-json subprocess)
- `src/runner/codex-runner.ts` - Codex CLI runner (`codex exec --json`, e2e-verified)
- `src/runner/gemini-runner.ts` - EXPERIMENTAL CLI runner (docs-derived, synthetic-fixture tested; will be REPLACED by an Antigravity CLI runner — Gemini CLI is deprecated for consumer tiers, see issue #126)
- `src/runner/opencode-runner.ts` - OpenCode CLI runner (e2e-verified against opencode 1.17.13; env-based per-trial isolation)
- `src/harness/subprocess.ts` - Shared CLI spawn/JSONL/process-tree-kill plumbing
- `src/runner/base-runner.ts` - Shared runner base class (timeout wrapper, skillsMountPath)
- `src/runner/runner-factory.ts` - Runner selection factory
- `src/runner/security.ts` - PreToolUse write restrictions
- `src/scorer/scorer.ts` - Score orchestrator (deterministic reward + opt-in judge diagnostics)
- `src/scorer/deterministic.ts` - Activation/marker/tool-call/contains/regex/js/file-exists checks
- `src/scorer/judge.ts` - LLM-as-judge diagnostics (SkillJudge)
- `src/score/metrics.ts` - Pure metrics: resolution rate, pass@k, skill lift, invocation rate, binomial CI, grouping
- `src/results/types.ts` + `src/results/summary.ts` - RunSummary contract + summary.json builder
- `src/cache/response-cache.ts` - Content-addressed cache of TaskResult by execution inputs
- `src/utils/concurrency.ts` - Bounded-concurrency helper used by runner + judge
- `src/session/session-logger.ts` - Event capture and session logging
- `src/report/report.ts` - Markdown + JSON report generation
- `src/report/html-report.ts` - Interactive static HTML report
- `src/report/github-summary.ts` - Condensed GitHub Actions summary
- `src/index.ts` - Public API exports
- `action/action.yml` + `action/index.ts` - GitHub Action entry point

## Commands

```bash
npm run build           # Compile TypeScript to dist/
npm run bundle:action   # Build + bundle GitHub Action (action/dist/index.cjs)
npm run dev             # Run CLI in dev mode (tsx)
npm run typecheck       # Type check without emitting
npm run start           # Run compiled CLI
```

**Important:** When changing scorer, task loader, types, or pipeline code, run `npm run bundle:action` before committing to keep the GitHub Action bundle in sync.

## Architecture

```
Task packages → Config → Per-trial workspace → Runner (with-skill + baseline conditions) → Verifier → Deterministic reward (+ optional judge diagnostics) → summary.json → Report
```

## Task packages

A task is a directory containing `task.md` (YAML frontmatter + markdown prompt body). `skilljack-evals run <path>` accepts a single task-package dir or a suite dir of task packages. Frontmatter: `id` (defaults to dir name), `difficulty`/`category`/`tags`, `expected_skill`, `expect_skill_invocation` (false = anti-trigger test), `timeout_ms`, `verifier: { timeout_ms, command }`, `checks:` (lite checks: `contains`, `not_contains`, `regex`, `marker`, `tool_calls`, `no_tool_calls`, `files_exist`, `javascript`), `assertions:` (judge-graded checklist), plus interop keys `requires_docker` and `x_skillsbench` (written by `import`). Optional dirs: `environment/skills/` (task-level skills; falls back to suite-level `<suite>/skills/`), `environment/workspace/` (seed files), `verifier/verify.*`, `oracle/solve.*`. `--skills-dir` overrides skills for all tasks (candidate injection). `validate <path>` runs schema checks plus the oracle gate (oracle → verifier must yield reward 1.0; skip with `--no-oracle`); it warns when a task's only signal is skill invocation (baseline would trivially pass → lift meaningless).

## SkillsBench/BenchFlow interop


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olaservo/skilljack-evals](https://github.com/olaservo/skilljack-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
