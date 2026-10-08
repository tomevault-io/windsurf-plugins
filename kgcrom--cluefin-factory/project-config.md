---
trigger: always_on
description: Cluefin Factory is an agent configuration for Korean-market investment research —
---

# AGENTS.md

Cluefin Factory is an agent configuration for Korean-market investment research —
a "factory" that keeps turning out company analyses and stock picks.
It is mostly Claude Code configuration (Markdown) plus docs. Run `claude` in-repo;
analyst skills live in `.claude/skills/`, the orchestrator in
`.claude/agents/market-review.md`. The agent reaches market data by shelling out to
the external cluefin CLI (`uv run cluefin-openapi-cli`) via bash — no MCP server, no
build step, no TypeScript.

It is **no longer source-free.** Deterministic steps that skills used to spell out in
prose now belong in `scripts/` (see *Code vs. skill* below). Everything else is still
Markdown.

The Pi runtime (`.pi/`, `@earendil-works/pi-coding-agent`) was removed; keeping two
runtimes in sync with a moving CLI cost more than it returned. npm is biome (formats
the JSON/config files) and vitest, which now has `scripts/` to cover.

## Commands

```bash
claude                # the actual entrypoint — no build, no install needed
npm test              # vitest run (tests/**/*.test.mjs)
npm run test:coverage # vitest with v8 coverage
npm run lint          # biome check .
npm run lint:fix      # biome check --write .
npm run format        # biome format --write .
```

There is nothing to build. npm is the lint/test toolchain; `scripts/` runs on Node
directly, with `js-yaml` and `ajv` as its only dependencies.

## Layout

```
.claude/
├── agents/market-review.md   # market-review subagent (the orchestrator)
├── agents/blind-judge.md     # blind backward judge: Read only, guarded to one case file
└── skills/                   # 13 analyst skills: bull/bear-analyst, fundamental-,
                              # technical-, macro-, news-analysis, scenario-planner,
                              # final-decision, data-sanity-check, portfolio-fit,
                              # risk-position-sizing, investment-journal,
                              # decision-scorecard
docs/                         # TODO.md + assets/ (GitHub Pages source)
schemas/                      # final-decision frontmatter + blind case JSON Schemas
scripts/                      # deterministic steps lifted out of the skills
├── scorecard.mjs             # lint / score [--write] / aggregate
├── blind.mjs                 # blind run: universe / register / build / judge / restore
├── blind/                    # case masking, seal, sampling, restore, judge read-guard
├── pit.mjs                   # PIT store: migrate / rebuild / stats / fill
└── pit/                      # node:sqlite store, raw→fact parsers, as-of queries
tests/                        # vitest specs + fixtures
```

Investments data is per-user and git-ignored: `.claude/investments/` (the PIT store is
`.claude/investments/pit/pit.sqlite`, blind runs are `.claude/investments/blind/<run>/`).

Each skill pins a model in its frontmatter: `sonnet` for rule-application, arithmetic
and recording (data-sanity-check, technical-analysis, risk-position-sizing,
investment-journal, decision-scorecard), `opus` for everything that weighs multiple
sources or argues a side.

## Code vs. skill

A step belongs in `scripts/` when the same input must always produce the same output:
schema validation, date arithmetic, predicate evaluation, return/benchmark/volatility
maths, aggregation tables, and writing computed fields back into journal frontmatter.
Writing that logic ad hoc each session costs tokens, drifts between runs, and cannot be
tested. The `invalidation` predicates (`metric`/`op`/`value`) were designed to be
machine-read — a skill should not be reading them by eye.

A step stays in a skill when it weighs evidence or writes prose: forming a thesis,
designing invalidation conditions, arguing bull vs. bear, resolving `checkable: false`
conditions with the user, and interpreting a scorecard. Skills also stay in charge of
changing the rules themselves.

**Rules live in the code that enforces them.** When a rule moves to `scripts/`, the
skill keeps how to invoke it and how to read the result, not a second copy of the rule —
two copies drift and the Markdown one wins by accident. Scripts are Node ESM (`.mjs`)
because biome and vitest already cover JS; they are not Python, even though the cluefin
CLI is. Tests live in `tests/**/*.test.mjs`.

New scripts ship with vitest coverage — that is what stops `npm test` being
decorative.

## Data sources

Market data tools shell out to the external **cluefin** CLI (`uv run
cluefin-openapi-cli`). Required keys in `.env` (see `.env.example`):
`DART_AUTH_KEY`, `KIS_APP_KEY`/`KIS_SECRET_KEY`/`KIS_ENV`.

## Gotchas

- **`blind-judge` runs as a separate headless process** (`claude -p --agent blind-judge`,
  driven by `node scripts/blind.mjs judge`), never as a subagent of a session that has
  read seals. Agent definitions load at session start, so a new or edited agent file is
  invisible to the Agent tool until restart — the headless process reads it fresh.
- **PIT store needs Node ≥ 22.5** for the built-in `node:sqlite` (still experimental —
  `scripts/pit/db.mjs` is the only module that imports it). A row dated after `as_of`
  reaching the blind case builder is a bug upstream; the builder throws instead of
  filtering it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kgcrom/cluefin-factory](https://github.com/kgcrom/cluefin-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
