---
trigger: always_on
description: VS Code extension that analyzes local AI session logs and surfaces insights in a webview dashboard. Read-only, zero telemetry, all analysis runs on the user's machine.
---


# AGENTS.md

You are an experienced TypeScript engineer working on the **AI Engineer Coach** VS Code
extension. Your job is to keep analysis correct, the extension host responsive, and user data
private — this codebase has zero telemetry and never modifies user session logs.

If you're a human, [`README.md`](README.md) is the better starting point.

## Tech stack

- **Node** ≥ 20 (CI uses Node 22)
- **TypeScript** 6.0.3, strict mode
- **VS Code engine** `^1.120.0` (`@types/vscode` 1.120.0)
- **Bundler** esbuild 0.28.0 (`esbuild.mjs`, output → `dist/extension.js`)
- **Tests** vitest 4.1.7 (unit + inline rule tests), Playwright 1.60.0 (e2e webview)
- **Lint** eslint 10.4.0
- **Docs site** Hugo (sources in `docs/content/`, published to `microsoft.github.io/AI-Engineering-Coach/`)

## Repository map

```
AI-Engineering-Coach/
├── src/
│   ├── extension.ts            # VS Code activation entry point
│   ├── core/                   # Parsers, analyzers, the rule engine
│   │   ├── analyzer.ts          # Top-level coordinator across analyzer-*.ts
│   │   ├── parser.ts            # Reads session logs from disk
│   │   ├── parse-worker.ts      # Worker thread: logsDirs → progress + result/error
│   │   ├── warm-up-worker.ts    # Worker thread: sessions → antiPatterns + configHealth
│   │   ├── cache-write-worker.ts# Worker thread: persists cache payload
│   │   ├── metric-engine.ts     # DSL evaluator for rules and metrics
│   │   ├── rule-loader.ts       # Loads built-in + personal + project rule layers
│   │   ├── rule-trust.ts        # Trust gate (pending → review → approve → reload)
│   │   ├── rules/<id>.md        # 45+ built-in detection rules (markdown + DSL)
│   │   └── metrics/<id>.metric.md# Built-in metrics referenced by rules
│   ├── webview/                # Dashboard UI: app.ts plus page-*.ts per route
│   ├── chat/                   # VS Code Chat participant integration
│   ├── mcp/                    # Tools exposed to the chat participant / MCP
│   └── summary-export-vscode.ts# Markdown/JSON summary export
├── docs/
│   ├── content/                # Hugo source for https://microsoft.github.io/AI-Engineering-Coach/
│   ├── AUTHORING_RULES.md      # How to author a rule or metric (DSL + tests)
│   └── hugo.toml
├── scripts/                    # Packaging, smoke tests, data inventory tools
├── skills/                     # Reusable instructions for recurring agentic tasks
├── tests/e2e/                  # Playwright end-to-end tests
└── AGENTS.md                   # You are here
```

## Build, test, and ship

| Task | Command |
|---|---|
| Install dependencies | `npm ci` |
| Bundle the extension | `npm run build` |
| Watch-mode rebuild | `npm run watch` |
| Type-check | `npm run typecheck` |
| Lint | `npm run lint` |
| Spellcheck markdown + TS | `npm run spellcheck` |
| Unit tests (vitest) | `npm test` |
| All checks (CI gate) | `npm run check` |
| End-to-end (Playwright) | `npm run test:e2e` |
| Package the VSIX | `npm run package` (see [skills/package-extension.md](skills/package-extension.md)) |
| Bundle-size budget | `npm run check-size` |

CI runs `npm run check` (typecheck + lint + spellcheck + knip + test) plus the size check on
every PR. Run those locally before pushing.

## Skills

Repo-specific instructions for recurring tasks live in [`skills/`](skills/). They are symlinked
into [`.claude/skills/`](.claude/skills/) and [`.github/instructions/`](.github/instructions/)
so popular agent harnesses pick them up automatically. See
[`skills/README.md`](skills/README.md) for the authoring format.

Available today:

- [`skills/update-docs.md`](skills/update-docs.md) — author or update a Hugo doc page.
- [`skills/package-extension.md`](skills/package-extension.md) — produce an installable `.vsix`.

## Rule and metric authoring

Detection rules and metrics are the primary extensibility surface — markdown files with YAML
front matter and a small DSL, no code changes required.

- Built-in rules: [`src/core/rules/<id>.md`](src/core/rules/) (45+ today)
- Built-in metrics: [`src/core/metrics/<id>.metric.md`](src/core/metrics/)
- Authoring guide with annotated examples: [`docs/AUTHORING_RULES.md`](docs/AUTHORING_RULES.md)
- Trust layers (built-in / personal / project) gated through
  [`src/core/rule-trust.ts`](src/core/rule-trust.ts)

Rules ship with inline `# Tests` blocks that run as part of `npm test`.

## Workers

Heavy lifting happens off the extension host thread:

- [`src/core/parse-worker.ts`](src/core/parse-worker.ts) — `logsDirs` → `progress` + `result`/`error`.
- [`src/core/warm-up-worker.ts`](src/core/warm-up-worker.ts) — `sessions` → `antiPatterns` + `configHealth`.
- [`src/core/cache-write-worker.ts`](src/core/cache-write-worker.ts) — persists the cache payload.

## Local rule trust flow

Rules move pending → review → approve → reload; edits revoke trust. See
[`docs/content/improve/anti-patterns.md`](docs/content/improve/anti-patterns.md) and
[`docs/content/improve/rule-editor.md`](docs/content/improve/rule-editor.md).

## Documentation index

These pages are published at https://microsoft.github.io/AI-Engineering-Coach/. The links below

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/AI-Engineering-Coach](https://github.com/microsoft/AI-Engineering-Coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
