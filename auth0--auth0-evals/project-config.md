---
trigger: always_on
description: `auth0-evals` measures how accurately LLM agents complete Auth0 SDK integration tasks across 5 configurations:
---

# AGENTS.md

## What this repo does

`auth0-evals` measures how accurately LLM agents complete Auth0 SDK integration tasks across 5 configurations:

| Configuration      | CLI flags                         | Grader levels |
| ------------------ | --------------------------------- | ------------- |
| `baseline`         | `--mode baseline`                 | L1-L3         |
| `agent`            | `--mode agent`                    | L1-L5         |
| `agent+skills`     | `--mode agent --tools skills`     | L1-L5         |
| `agent+mcp`        | `--mode agent --tools mcp`        | L1-L5         |
| `agent+mcp+skills` | `--mode agent --tools mcp,skills` | L1-L5         |

Each eval: `src/evals/<category>/<eval-dir>/PROMPT.md` + `graders.ts`. The `id` field in `PROMPT.md` frontmatter is used with `--eval`.

---

## Key commands

```bash
npm run build     # compile to dist/
npm test          # run Vitest
npm run lint
npm run format

# Run evals
npm run evals -- --eval react_quickstart --mode agent
npm run evals -- --eval react_quickstart --mode agent --tools mcp,skills
npm run evals -- --mode all --model all --workers 8
npm run evals -- --eval react_quickstart --mode agent --keep-workspace

# Generate HTML report
npm run report

# AXIS — run all evals across all agents
npm run axis
npm run axis -- --eval react_quickstart --agent claude-code --model claude-sonnet-5
```

Full CLI flags and AXIS flags: see [`packages/evals/README.md`](packages/evals/README.md).

### AXIS flags (`npm run axis`)

| Flag | Values | Default | Notes |
| ---- | ------ | ------- | ----- |
| `--eval <id>` | Any registered eval ID | all evals | Repeatable |
| `--agent <name>` | `claude-code`, `codex`, `gemini` | all agents | Repeatable |
| `--workers <n>` | number | AXIS default | Parallel job limit |
| `--model <model>` | Any model string | per-agent default | Override model for all configured agents |
| `--output <path>` | file path | `scores-axis.json` in app root | Where to write the scores file |
| `--debug` | flag | off | Capture raw adapter stdout as `.raw.ndjson` for debugging |

---

## Conventions

### ESM — `.js` extensions on every import

`package.json` sets `"type": "module"`. Every import needs a `.js` extension, even when importing `.ts` source files. Use `node:` prefix for builtins. Use `import type` for type-only imports.

```typescript
import { contains } from '@a0/evals-graders'; // ✓
import { readFileSync } from 'node:fs'; // ✓
import type { GraderDef } from '@a0/evals-graders'; // ✓
```

For dynamic imports of absolute paths, use `pathToFileURL(path).href` — bare absolute paths fail on macOS and Windows.

### Tools return tuples, never throw

```typescript
return ['path argument is required', false, false, true]; // ✓
throw new Error('path required'); // ✗ crashes the agent loop
```

Always resolve paths with `resolveInside(context.workspace, args.path)` — not `join()`.

---

## Grader levels

Two authoring rules: **grade the artifact, not the explanation** (verify generated code compiles and calls real SDK methods — never grade prose); **if every model passes, the eval is broken** (tighten graders when all models score >90%).

Every grader must have a `GraderLevel`. End every eval with one holistic `judge` with no level:

| Level | Enum value            | What it tests                                          | Runs in                |
| ----- | --------------------- | ------------------------------------------------------ | ---------------------- |
| L1    | `positive_presence`   | Required SDK symbols, imports, config keys are present | All configs            |
| L2    | `hallucination`       | Hallucinated packages / wrong SDK variants are absent  | All configs            |
| L3    | `security`            | No hardcoded credentials or tokens in insecure storage | All configs            |
| L4    | `structural`          | Code is correctly wired — right components, lifecycle  | Agent configs only     |
| L5    | `version_correctness` | Uses current API, not deprecated patterns              | Agent configs (with or without MCP) |

Use `notContainsInSource` (not `notContains`) when a value is allowed in config files but must not appear in source code.

### Grader primitives

| Primitive                                        | What it does                                                                                                                                   |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `contains(needle)`                               | Substring present in any workspace file. Add `{ source: 'response' }` or `{ source: 'both' }` to also search the agent's final reply text.    |
| `notContains(needle)`                            | Substring must NOT appear in any workspace file. Same `source` option as `contains`. Add `{ ignoreComments: true }` to strip comments before matching (string literals kept), so a token mentioned only in a `//`/`#`/block comment doesn't trip an L2 check — also available on `contains` and `notContainsInSource`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/auth0-evals](https://github.com/auth0/auth0-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
