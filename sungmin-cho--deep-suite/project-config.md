---
trigger: always_on
description: Registry and integration layer for the Deep Suite plugin family: marketplace manifests, suite-side schemas, integration guides, analysis docs, CI tooling. **No plugin source code** — each plugin lives in its own repo at `github.com/Sungmin-Cho/deep-{name}`. Treat this repo as a registry, not a monorepo.
---

# Deep Suite — Agent Guide

Registry and integration layer for the Deep Suite plugin family: marketplace manifests, suite-side schemas, integration guides, analysis docs, CI tooling. **No plugin source code** — each plugin lives in its own repo at `github.com/Sungmin-Cho/deep-{name}`. Treat this repo as a registry, not a monorepo.

The repositories were renamed from `claude-deep-*` to runtime-neutral `deep-*` on 2026-08-19; GitHub serves the old names as permanent redirects, so never re-create a `claude-deep-*` repo — doing so kills the redirect. The marketplace **name** stays `claude-deep-suite`: it is what plugin keys (`deep-work@claude-deep-suite`) and the `~/.claude/plugins/cache/<marketplace>/` path derive from, and renaming it would break every existing install. Codex parity comes from the mirror manifest, not a rename.

## Plugins

<!-- deep-suite:auto-generated:plugin-table-agents:start -->

| Plugin | Version | Description |
|---|---|---|
| deep-work | 7.3.0 | Evidence-Driven Development Protocol |
| deep-wiki | 1.10.1 | Exact worker contracts with bounded timeout fallback and journaled wiki mutation |
| deep-evolve | 3.6.3 | Autonomous Experimentation Protocol |
| deep-review | 2.10.0 | Independent Evaluator for AI coding agents |
| deep-docs | 1.7.0 | Document gardening + authoring |
| deep-dashboard | 1.5.1 | Cross-plugin harness diagnostics + suite telemetry |
| deep-memory | 1.0.6 | Cross-project semantic memory |
| deep-goal | 1.2.1 | Goal condition compiler |
| deep-loop | 1.22.1 | Loop Engineering control plane over the deep-suite |
| deep-model-router | 1.14.0 | Deterministic model/effort/review router for Claude Code, Codex, and Grok |

<!-- deep-suite:auto-generated:plugin-table-agents:end -->

> Auto-generated from the marketplace manifest + each plugin's pinned `plugin.json.version`. Everything outside the markers is hand-curated.

## Quick Start

```bash
npm install                              # ajv + ajv-formats, devDeps only
npm test                                 # node:test — unit + spawnSync CLI
npm run validate                         # sidecar: schema + referential integrity
npm run docs:write                       # regenerate marker regions
npm run docs:sync                        # 8 doc-sync checkers
npm run preflight                        # the gate: validate + docs:check + docs:sync + fixtures + test
npm run release:bump -- <plugin> <sha40> # pin → docs:write → preflight
```

Node 20+, ESM. A `prepare`-installed pre-push hook runs `preflight` before every push (bypass: `SKIP_PREFLIGHT=1 git push`).

## Project Structure

```
.claude-plugin/
  marketplace.json          — Claude manifest; pins every plugin to a commit SHA
  suite-extensions.json     — suite sidecar (M1); all cross-plugin metadata
.agents/plugins/
  marketplace.json          — Codex mirror; same pins behind extra policy fields
schemas/                    — sidecar + M3 artifact-envelope + payload-registry/<producer>/<kind>/
scripts/                    — validators, marker generator, 8 check-*.js gates, release-bump
tests/                      — node:test suite covering every script above
guides/                     — 7-plugin integrated workflow, hook patterns, context management
examples/                   — installable hook configs + a handoff walkthrough
docs/
  memory-hierarchy.md       — cross-plugin memory hierarchy contract
  test-catalog.md           — where each cross-plugin test lives and what it owns
  envelope-migration.md     — M3 Phase 2 migration guide for plugin maintainers
```

The rest of `docs/` is gitignored working notes. `.claude/`, `.deep-review/`, `.deep-loop/`, `.deep-suite-cache/`, `node_modules/` are runtime artifacts — never commit them.

## Conventions

### Version policy — plugin SemVer, marketplace SHA pinning

- `marketplace.json` plugin entries carry **no `version` field**. `plugin.json.version` at the pinned SHA is the single source of truth, and each plugin's cache key (official priority: `plugin.json.version` → marketplace `entry.version` → commit SHA → unknown).
- `source.sha` is source pinning — which commit to fetch.
- The two manifests must move together: `tests/codex-marketplace-contract.test.js` deep-compares `source` and `description`, so bumping one side alone is red.

### Suite sidecar (M1)

- All cross-plugin metadata goes into `.claude-plugin/suite-extensions.json` only. **Never modify `marketplace.json`** to carry it — that file stays conformant to the official schema.
- Sidecar schema is locked at `1.0`. Forward-compatible additions go through `x-*` patternProperties only; a breaking change requires a new file (`*-v2.schema.json`). See `schemas/README.md`.
- Sidecar `artifacts.writes` / `reads` must match the **pinned** source — only advertise a path the plugin actually emits at that SHA (`check-pinned-plugin-paths.js` greps the pinned tree for it).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sungmin-Cho/deep-suite](https://github.com/Sungmin-Cho/deep-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
