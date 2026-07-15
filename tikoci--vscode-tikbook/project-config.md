---
trigger: always_on
description: Keep guidance brief and actionable. For full details, see [DEVELOPMENT.md](../DEVELOPMENT.md).
---

# Copilot instructions for TikBook

Keep guidance brief and actionable. For full details, see [DEVELOPMENT.md](../DEVELOPMENT.md).

Additional context-specific rules in `.github/instructions/`:

- `ai-editing-best-practices.md` - Safe code editing protocols (read first!)
- `vscode-extension.instructions.md` - Extension code standards
- `testing.instructions.md` - Test and experimental code
- `documentation.instructions.md` - Docs organization and linking
- `biome-rules.instructions.md` - Biome linting expectations
- `routeros-integration.instructions.md` - RouterOS REST API patterns

Pattern guides in `docs/`:

- `typescript-patterns.md` - TypeScript + RouterOS types (generics, type narrowing, extensible records)
- `routeros-patterns.md` - REST API patterns (version compatibility, error handling, credentials)
- `versioning-patterns.md` - VS Code version compatibility (API gates, runtime fallbacks)
- `testing-vscode-web-local.md` - Testing web extensions locally and on vscode.dev
- `copilot-setup.md` - Copilot configuration and troubleshooting
- `agentic-collaboration-patterns.md` - AI-assisted spec refinement workflows (iterative Q&A, research patterns)

Architectural reference: See [docs/architecture.md](../docs/architecture.md), [docs/conventions.md](../docs/conventions.md), and [docs/sarb/code-review-checklist.md](../docs/sarb/code-review-checklist.md).

**📋 Feature Development (start here each session):**

- **Roadmap (near-term themes):** [ROADMAP.md](../ROADMAP.md) — seeded tasks for agents
- **Quick tasks (1-3 hours):** [docs/llm-todos.md](../docs/llm-todos.md)
- **Larger features:** [docs/specs/README.md](../docs/specs/README.md) — only implement specs marked `ready-for-implementation`
- **Long-term vision:** [docs/future-features.md](../docs/future-features.md)

If `ROADMAP.md` conflicts with an older draft spec, TODO, or historical note, treat
`ROADMAP.md` as the near-term source of truth and update the stale doc as part of
the work.

`CHANGELOG.md` is past-tense only — do not record planned work there.

## Build, test, lint commands

Bundler is **Bun** (`bun build`), not webpack/esbuild. Two targets: `out/extension.js`
(node `main`) and `dist/extension.js` (browser `browser`).

| Command | What it does |
|---|---|
| `npm run compile` | clean + lint + typecheck + build node target (`out/extension.js`) |
| `npm run compile:web` | build browser target (`dist/extension.js`) |
| `npm run compile:test` | build `out/test/{unit,integration}/**/*.test.js` — **required before GUI Test Runner discovers tests** |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run lint` / `npm run lint:fix` | `biome check .` / `biome check --write .` |
| `npm test` | unit tests only (`out/test/unit/**`, via `.vscode-test-cli.mjs`) |
| `npm run test:web` | same unit tests in the web extension host (`--browser`) |
| `npm run format` | biome `--write` + markdown `--fix:all` |
| `npm run vsix:package` | build node + web `.vsix` |

**Run a single test** (mocha flags pass through `vscode-test`):

- By name: `npm test -- --grep "credential"` (or `-f` for fixed-string match)
- By file: `npm test -- --run out/test/unit/converters.test.js`

`npm test` runs `pretest` (compiles node + test) automatically. Integration tests in
`src/test/integration/` are opt-in (each file uses top-level `suite.skip`); default
`npm test` runs unit only.

## Architecture in one paragraph

TikBook is the VS Code companion to the `TIKOCI.lsp-routeros-ts` extension (shipped as
an `extensionPack`); language parsing/diagnostics live in that LSP, not here. Entry
`src/extension.ts` wires up: a **notebook kernel** (`notebook.ts`) supporting two
formats (`.tikbook`/`.md.rsc` and `.rscmd`/`.rsc.md`); two **virtual filesystems**
(`rscena://` read-only views in `virtualdocs.ts`; `rscfile://` read-write ScriptFS in
`scriptfs.ts`); a **REST client** for RouterOS (`routeros.ts` + `shared.ts`); a status
**watchdog** (`watchdog.ts`); `/app` YAML + schema tooling (`app-yaml.ts`,
`schema-mapper.ts`, `scriptfs-schema.ts`); and `converters.ts`/`commands.ts`/`menus.ts`/
`codelens.ts`. Parked CHR VM explorer work (`vm-explorer.ts`, `vm-commands.ts`,
`vm-providers/`) exists but is not currently activated. The same `src/extension.ts`
compiles to both node and browser targets, so **all** extension code must be
web-safe: gate desktop-only paths with `vscode.env.uiKind === UIKind.Desktop`, prefer
`vscode.workspace.fs`/`vscode.Uri` over `node:fs`/`path`, and use `SecretStorage` (via
`config.ts`) for credentials. `vscode-compat.ts` holds VS Code version-gating shims
(min engine `^1.78.2`).

## Core rules

- This is a VS Code extension. Avoid Node-only APIs in extension code, especially for web.
- Do not use console.log. Use the existing output logging helper (e.g., log.info()).
- RouterOS support targets 7.20.2+ (min 7.10). Validate commands against v7 schema.
- If a change belongs in the RouterOS LSP (not VS Code-specific), suggest that instead.
- Do not change package.json version unless the user asks.
- Prefer vscode.workspace.fs + vscode.Uri over Node fs/path for file IO.
- Gate desktop-only behavior with vscode.env.uiKind or vscode.env.appHost.
- Use SecretStorage for credentials; avoid settings for secrets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tikoci/vscode-tikbook](https://github.com/tikoci/vscode-tikbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
