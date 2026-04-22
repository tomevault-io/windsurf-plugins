---
trigger: always_on
description: Skill test harness with autonomous prompt optimization. Two modes: `xvivo run` (fast pass/fail) and `xvivo optimize` (Karpathy loop for iterative skill improvement).
---

# xvivo

Skill test harness with autonomous prompt optimization. Two modes: `xvivo run` (fast pass/fail) and `xvivo optimize` (Karpathy loop for iterative skill improvement).

## Commands

```bash
npm run dev          # run with tsx (ESM, no compile step)
npm run build        # tsc → dist/
npm run test         # vitest run
npm run test:watch   # vitest watch
npm run lint         # biome check + tsc --noEmit
```

## Tech stack

- **TypeScript** (strict, ESM-only) — `"type": "module"` in package.json
- **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`) — agent invocation via `query()`
- **Ink** + **Pastel** — React-based TUI rendering, CLI command routing
- **Vitest** — test runner (ESM-native, no Jest)
- **execa** — subprocess execution for Python scripts
- **Zod** — runtime schema validation for test specs and configs
- **simple-git** — git operations for the optimization loop's keep/revert cycle
- **PyTorch** + **sentence-transformers** — ML-based evaluators (Class C), runs as Python subprocess
- **Biome** — linting + formatting (replaces ESLint + Prettier)

## Architecture

- `src/cli/` — Ink components and Pastel commands (`run`, `optimize`, `train`, `models`)
- `src/runner/` — test discovery, orchestration, result collection
- `src/evaluators/` — Class A (deterministic), B (LLM-judge), C (ML-model) evaluators
- `src/optimizer/` — Karpathy loop: mutation operators, keep/revert, results logging
- `src/sandbox/` — temp directory lifecycle, script copying, fixture writing, output collection
- `src/parsers/` — markdown section parser, test spec YAML loader
- `src/types/` — shared Zod schemas and TypeScript types
- `scripts/ml/` — Python ML evaluator scripts (embedding similarity, classifier, anomaly, calibrator)
- `models/` — trained ML model checkpoints (gitignored except metadata)
- `.claude/references/` — detailed best practices per technology (read on demand, not in every session)

## Conventions

- Named exports only, no default exports
- Async/await over raw promises — never `.then()` chains
- All Python interaction goes through `execa` with JSON on stdin/stdout — never import Python from Node
- Evaluators return `EvalResult` (see `src/types/eval.ts`) — always `{ passed, score, details }`
- Test specs are YAML with Zod validation — malformed specs fail at load time, not runtime
- The optimization loop NEVER modifies evaluation code — strict separation of concerns
- Git operations use simple-git, not shell `git` commands
- All exported symbols (functions, schemas, types, classes, interfaces) MUST have JSDoc comments — see `.claude/references/jsdoc-conventions.md`
- All new features and bug fixes MUST use Test-Driven Development (vertical slices, not horizontal) — invoke the `/tdd` skill and see `.claude/references/tdd-workflow.md`

## IMPORTANT

- YOU MUST use `.js` extensions in all TypeScript import paths (ESM resolution requires it)
- YOU MUST keep Ink components in `.tsx` files, all other TypeScript in `.ts` files
- NEVER use `require()` — this is an ESM-only project
- NEVER use `localStorage` or browser APIs — this is a Node.js CLI application
- NEVER put ML model files (`.pt`, `.pkl`) in git — only `metadata.json` is committed
- When adding a new evaluator, add it to all three places: the type union in `src/types/eval.ts`, the registry in `src/evaluators/index.ts`, and the Zod schema in `src/types/spec.ts`

## Reference docs

Read these BEFORE working on the relevant subsystem. They contain antipatterns that will waste your time.

- `.claude/references/ink-and-pastel.md` — Ink component patterns, Static for streaming output, TSX gotchas
- `.claude/references/agent-sdk.md` — query() streaming, message types, hooks, sessions, tool permissions
- `.claude/references/esm-typescript.md` — ESM module resolution, .js extensions, tsx runner, vitest config
- `.claude/references/python-subprocess.md` — execa patterns, JSON stdin/stdout protocol, venv management
- `.claude/references/ml-evaluators.md` — PyTorch model lifecycle, embedding caching, graceful degradation
- `.claude/references/optimization-loop.md` — Karpathy loop mechanics, mutation operators, results logging, git safety
- `.claude/references/jsdoc-conventions.md` — JSDoc requirements for exported symbols, patterns for schemas vs functions vs types
- `.claude/references/tdd-workflow.md` — TDD vertical slice workflow, mocking boundaries, test quality checklist, invoke `/tdd` skill

<!-- GSD:project-start source:PROJECT.md -->
## Project

**xvivo**

A test harness and autonomous prompt optimizer for AI instruction files (skills, commands, agents, prompts). It parses instruction files into testable units — YAML frontmatter, XML blocks, markdown sections — then runs those units through a three-class evaluation pipeline (deterministic, LLM-judge, ML-model) to produce fast pass/fail results. In optimize mode, it iteratively mutates individual sections using a Karpathy-style hill-climbing loop, keeping changes that improve the evaluation score and reverting those that don't. Intended for open-source release.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddunnock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
