---
trigger: always_on
description: Design Studio AI exposes one shared document contract through REST, network MCP, browser WebMCP, and `dsa`. The CLI package is maintained under [packages/cli](../packages/cli/README.md). Its executable bundles the shared validators, theme/template/block catalog, targeted operations, and safe static renderers. It does not require a running local project checkout after installation.
---

# Agent access and CLI

Design Studio AI exposes one shared document contract through REST, network MCP, browser WebMCP, and `dsa`. The CLI package is maintained under [packages/cli](../packages/cli/README.md). Its executable bundles the shared validators, theme/template/block catalog, targeted operations, and safe static renderers. It does not require a running local project checkout after installation.

## Install and connect

Follow the [CLI installation instructions](../README.md#agent-access) for the released tarball. The package is not published to the npm registry.

To build from source, install dependencies with `npm ci` and `npm ci --prefix packages/cli`, then run `npm run build --prefix packages/cli`. From `packages/cli`, run `npm pack`; install the resulting tarball with `npm install -g <path-to-tarball>`. The build also generates `dist/document.schema.json` and `dist/operations.schema.json`.

Set `DESIGN_STUDIO_URL=https://studio.agentkit.best` and inject `DESIGN_STUDIO_API_KEY` from workspace Settings. `dsa` does not save a configuration file, keychain record, or login session. `--url` and `--api-key` override these values for one invocation; use environment injection to avoid shell history. HTTP is accepted for localhost development only.

Install the [companion skill](../skills/design-studio-ai/SKILL.md) by copying its directory into the installed skills directory of your agent runtime. The repository layout is also suitable for a skill installer that accepts a repository and skill path. Copy the complete directory, including `references/`. The skill routes each design kind to composition and review guidance, alongside brief capture, catalog discovery, targeted edits, and authorized exports/publishing. Start with its [shared layout and quality reference](../skills/design-studio-ai/references/layout-and-quality.md); the [skill index](../skills/design-studio-ai/SKILL.md#choose-the-design-kind-guidance) links the kind-specific references.

Connect a coding agent directly to the network MCP server with `dsa mcp install <agent>`; it prints the server config snippet by default and only touches the agent's config file when `--write` is passed (with a `.bak` backup and no-clobber merge).

## CLI command surface

This reference follows the current [CLI source](../packages/cli/src/dsa.ts) and [library commands](../packages/cli/src/design-system-commands.ts). The linked release can lag these capabilities; inspect installed command help and build from source when a needed command is absent.

| Commands | Behavior |
| --- | --- |
| `health`, `config` | Public health and configuration; no credential persistence |
| `schema [--operations]` | JSON Schema from the shared validators; semantic checks still run on writes |
| `catalog`, `themes list/get`, `templates list/get/instantiate`, `blocks list/get` | Bundled design resources; instantiated IDs are unique |
| `projects list/get/create/rename/delete/clone` | Persisted project management; clone copies owned asset bytes |
| `projects paint ID --file command.json` | Server-rendered stroke/fill using observed revision, painting generation and exact retry ID |
| `projects document get/put/patch` | Canonical document reads and atomic expected-revision writes |
| `projects document merge/changes` | Three-way merge using the exact earlier base, and revision polling |
| `brief get/put/interview/approve` | Persisted interactive questions, answers, scope and explicit version-bound approval |
| `observability summary/events/trace` | Owner-scoped activity, provider usage, and correlated spans; global reads require configured operator authorization |
| `projects check` | Read-only preflight hints with exact layer IDs; inspect the actual preview too |
| `projects inspect ID`, `projects overview` | Private saved-page, project contact-sheet, and workspace-cover PNGs with revision and pagination metadata |
| `projects import/export`, `render` | Canonical JSON import; authenticated cloud export; offline JSON/HTML/SVG rendering |
| `assets list/upload/download` | Authenticated asset storage; node placement is a separate document edit |
| `generate` | Real provider document proposal; no implicit save |
| `fonts --query`, `providers models PROVIDER --query` | Search catalog metadata with explicit live/cache/fallback provenance |
| `design-systems schema/list/get/versions/create/update/apply/insert/remove/import/export` | Shared reusable libraries, immutable versions and conflict-checked project writes; `import`/`export` round-trip portable `DESIGN.md` + `tokens.css` + `manifest.json` folders |
| `providers list/set/remove` | Masked configuration; provider secret from environment/stdin |
| `tokens list/create/revoke` | Token metadata and lifecycle; new token returned once |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bestagentkits/design-studio-ai](https://github.com/bestagentkits/design-studio-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
