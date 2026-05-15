---
trigger: always_on
description: npm run build          # TypeScript compile (must pass before committing)
---

# CLAUDE.md — Project Guide for AI Agents

## Quick Reference

```bash
npm run build          # TypeScript compile (must pass before committing)
npm test               # Unit tests (221 tests, ~1s, no external deps) via vitest
npm run test:integration  # Integration tests against fixture vault (need OPENAI_API_KEY)
npm run coverage       # Unit tests with v8 coverage (≥40% lines required)
npm run knip           # Dead code / unused exports check (0 issues required)
npm run lint           # ESLint (0 errors required; warnings on `any` are ok)
npm run format         # Prettier write (run before committing)
npm run format:check   # Prettier check (used in CI)
```

**Pre-commit hooks run automatically** via husky + lint-staged (format + lint on staged files).

---

## Agent Verification Checklist

Run this sequence after any code change to get full feedback before committing:

```bash
npm run format && npm run build && npm test && npm run lint && npm run knip
```

**After modifying `searcher.ts`:**

- `npm test` catches BFS depth scoring regressions (`test/searcher.test.ts`)
- `npm test` catches tag/scope filter regressions (`test/searcher.test.ts`)
- `npm test` catches result shape contract regressions (`test/contract.test.ts`)
- Run eval before and after to measure ranking quality impact (see below)

**After modifying `db.ts`:**

- `npm test` catches NFD path storage, model change wipe, link integrity (`test/db.test.ts`)
- If you changed the DB **schema** (new columns, new tables, altered FTS structure): delete the eval
  fixture DB and regenerate the baseline so `test/eval/regression.test.ts` reflects the new state:
  ```bash
  rm -f fixtures/obsidian-help/en/.obsidian-hybrid-search.db
  npm run eval -- --vault fixtures/obsidian-help/en --output eval/results/baseline-no-rerank.json
  ```
  Then commit the updated `baseline-no-rerank.json`. Do NOT update the thresholds in
  `test/eval/regression.test.ts` unless the new metrics are genuinely better — the thresholds
  are a quality floor, not a mirror of the latest run.

**After modifying `server.ts` (MCP schema):**

- `npm run build` — TypeScript enforces that MCP parameter names map to valid `SearchOptions` fields
- If you add a new MCP parameter: update `SearchOptions` in `searcher.ts`, `server.ts`, and CLI flags in `cli.ts`
- `npm test` catches result shape changes (`test/contract.test.ts`)

**MCP description principles** — when editing any `description` field in `server.ts`, each parameter (especially mode-type enums) must answer four questions:

1. What does it do / return?
2. When should the agent choose this over alternatives?
3. What does it NOT do (that the agent might expect)?
4. What are the input constraints?

For overlapping modes, include explicit routing: "Use X when Y. Do NOT use X when Z — use W instead."
Omitting "when not to use" for overlapping modes causes agents to misinterpret correct behavior as bugs (precedent: S-66 — agent expected alias-match to be rank #1 in hybrid mode, but hybrid ranks by content depth, not note identity).

**After adding new `SearchOptions` fields:**
Update all three places: `SearchOptions` interface in `searcher.ts`, MCP tool schema in `server.ts`, and CLI flags in `cli.ts`.

**After any change that affects ranking quality** (`searcher.ts`, `embedder.ts`, `chunker.ts`, indexing logic):

Run eval before and after the change, then compare:

```bash
# Before your change — save baseline
npm run eval -- \
  --vault fixtures/obsidian-help/en \
  --output eval/results/before-<feature>.json

# Make your change, then run again
npm run eval -- \
  --vault fixtures/obsidian-help/en \
  --output eval/results/after-<feature>.json

# Compare
npm run eval:compare -- \
  eval/results/before-<feature>.json \
  eval/results/after-<feature>.json
```

Files in `eval/results/` are **working artifacts** — generate as many as you need and delete
freely. The regression test (`test/eval/regression.test.ts`) does NOT read them; it only reads
`eval/results/baseline-no-rerank.json` (the committed reference) and checks absolute thresholds.

Current committed baseline: **nDCG@5 = 0.727** (hybrid, local model, no rerank, 58 queries).
See `eval/README.md` for full benchmark table and model configuration options.

**Updating regression test thresholds** — only when a change genuinely improves ranking:

1. Run eval and confirm the new metrics are higher than the current thresholds
2. Update `eval/results/baseline-no-rerank.json` with the new run
3. Raise (never lower) the `FLOOR` values in `test/eval/regression.test.ts`
4. Update the "Measured baseline" comment in that file to match

**Coverage gates** (enforced in CI via `npm run coverage`):

- Lines ≥ 60%, Functions ≥ 65%, Branches ≥ 47%
- `embedder.ts`, `server.ts`, `cli.ts` are intentionally low coverage (require API key or OS I/O)
- Do not lower these thresholds — raise them as new testable code is added

---

## Architecture

```
CLI (cli.ts) ──┐
               ├──▶ search() in searcher.ts ──▶ db.ts (SQLite)
MCP (server.ts)┘                           └──▶ embedder.ts (OpenAI/local)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flowing-abyss/obsidian-hybrid-search](https://github.com/flowing-abyss/obsidian-hybrid-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
