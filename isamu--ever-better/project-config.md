---
trigger: always_on
description: Working notes for AI agents in this repo. What the tool is and how to use it lives in
---

# CLAUDE.md — ever-better

Working notes for AI agents in this repo. What the tool is and how to use it lives in
**README.md**; the design reasoning lives in [`plans/mvp-p0-p2.md`](plans/mvp-p0-p2.md); where the
work stands and what to do next lives in [`docs/HANDOFF.md`](docs/HANDOFF.md). Read those first.

## Stack

TypeScript, ESM, Node 20.11+. **Zero runtime dependencies** — that is a feature, not an accident.
A tool that installs other people's dev dependencies has no business dragging in its own. Before
adding one, check whether `node:util`, `node:fs` or a subprocess already covers it.

Package manager here is **yarn**.

## Run after changes

```
yarn format      # prettier
yarn lint        # eslint (this repo runs every tier it generates)
yarn typecheck   # tsc --noEmit over src AND test
yarn build       # tsc -p tsconfig.build.json -> dist/
yarn test        # node:test over test/*.ts
```

Never judge these through a pipe — `yarn lint | tail` exits with `tail`'s status and reports a
failing lint as success. Check the exit code, or use `set -o pipefail`.

## Imports use `.ts` extensions

`import { x } from "./foo.ts"`, not `"./foo.js"`. Tests run the TypeScript sources directly under
Node's native type stripping, and `rewriteRelativeImportExtensions` turns them into `.js` on emit.
`erasableSyntaxOnly` is on, so enums, namespaces and parameter properties are compile errors — they
would build fine and then crash the test runner.

## The one architectural rule

**Deterministic work goes in the CLI. Judgment goes in a skill.**

| CLI (`src/`) | Skill (`skills/`) |
| --- | --- |
| detect, install, count, render, gate | is this warning a real bug |
| the same answer every run | how to split this function |
| no LLM in the loop | DRY or coincidence |

If you are about to write a heuristic in a SKILL.md that produces a number, it belongs in the CLI.
If you are about to encode taste in `src/`, it belongs in a skill.

## Pure decisions, one impure gatherer

`gatherFacts(cwd)` in `src/facts.ts` is the **only** place detection touches the filesystem. It
returns a `RepoFacts` value; `diagnose(facts)` and everything under `src/detect/` are pure
functions over it. That is why the diagnosis has real tests and no fixtures on disk.

Keep it that way. A new detector takes facts and returns a verdict — it does not read a file.

## Do not reimplement the ratchet

`eslint --suppress-all` / `--prune-suppressions` **is** the ratchet, maintained by the ESLint team.
We invoke it. We never parse or write `eslint-suppressions.json` — `src/suppressionsFile.ts` only
sums it, to report how much a prune reclaimed.

Two behaviours here were bugs, and both would come back if the reasoning is lost:

- **`runRuleCounts` must pass `--pass-on-unpruned-suppressions`.** Without it ESLint exits fatally
  the moment a suppressed violation is *fixed*, because the leftover suppression is now unused —
  turning every act of draining into a red build.
- **`freeze` refuses to run twice.** A second freeze pins whatever exists at that moment, quietly
  legalising everything added since. `prune` is the only path down; `--force` is the documented
  escape and belongs in a PR description.

## Counting violations goes through the formatter, not `--format json`

`formatters/rule-counts.js` is an ESLint formatter that emits per-rule totals. `--format json`
grows with the number of violations, and the first run on an untouched repository is exactly when
that is largest.

It is **plain JavaScript and never compiled** — it must load identically whether the CLI runs from
`dist/` or from source. `formatterPath()` resolves it relative to the package root.

## Ask the tool, do not read the config — and count with colour off

`src/probe/` exists because a config file does not tell you what is enforced. Presets, a
framework's own config and later blocks override each other silently, and a rule that ends up off
reports nothing to notice. `eslint --print-config <file>` and `tsc --showConfig` are the only
honest answers, and both are asked per-file / after `extends` for that reason.

Two traps, both of which produced a wrong answer here before being fixed:

- **`tsc` and `eslint` colour their output, so `grep -c "error TS"` matches nothing** and reports
  a clean run. Three strictness flags were measured at "zero cost" that way and one of them had
  six errors. Always pass `--pretty false` when counting.
- **Measure a flag by enabling it in the tsconfig, not by passing it on the command line.** The
  CLI form silently disagreed with the project form here.

Type errors have no suppression mechanism — `--suppress-all` cannot touch them — so a strictness
flag must be measured before it is switched on. Enabling one that costs 500 errors hands the owner
a repo whose `typecheck` script fails, and nothing can grandfather that.

## Windows is not a formality

Both Windows failures here passed on Linux and macOS first, and neither is reproducible locally on
a Mac. The three-platform matrix is the only thing that sees them.

- **Never spawn a `node_modules/.bin` shim.** On Windows that directory holds an extensionless
  shell script *and* a `.cmd`; Node cannot spawn the first, and refuses the second without a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isamu/ever-better](https://github.com/isamu/ever-better) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
