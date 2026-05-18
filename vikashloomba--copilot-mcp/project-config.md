---
trigger: always_on
description: Guidance for coding agents working in this repository. This file focuses on structure and code maps so feature work can be done quickly and safely.
---

# AGENTS.md

Guidance for coding agents working in this repository. This file focuses on structure and code maps so feature work can be done quickly and safely.

## Engineering Standard
- All changes (workflows, code, tests, docs, and release automation) must follow durable best practices.
- Do not ship temporary hacks, stopgap fixes, or "good for now" implementations.
- Prefer maintainable architecture, deterministic behavior, and clear validation over quick patches.

## Project Shape
- VS Code extension backend: `src/`
- React webview frontend: `web/src/`
- Extension/webview RPC contracts: `src/shared/types/rpcTypes.ts`
- Chat participant (`@mcp`): `src/McpAgent.ts`
- Secondary Side Bar webview + Activity Bar launcher provider/RPC handlers: `src/panels/ExtensionPanel.ts`

## Build + Run
- Install deps (root + web): `npm run install:all`
- Build extension + webview: `npm run build:all`
- Watch mode (extension + web): `npm run watch`
- Lint: `npm run lint`
- Tests: `npm run test`
- Webview unit tests (Vitest): `npm run test:webview`
- Extension UI smoke/E2E harness: `npm run test:ui`
- Combined UI test pass: `npm run test:all-ui`
- Package VSIX: `npm run package-extension`
- CI test workflow: `.github/workflows/tests-on-main.yml` (runs on push/PR to `main`)

Important: the extension webview loads static assets from `web/dist/assets/index.js` and `web/dist/assets/index.css` (see `src/panels/ExtensionPanel.ts`). If you change frontend code, rebuild web assets.
Important (dev Play flow): `.vscode/tasks.json` default `watch` includes `watch:esbuild`, `watch:tsc`, and `watch:webview`. `watch:webview` runs `npm --prefix web run build -- --watch` so `web/dist/assets/*` is regenerated during development.

## UI Testing Requirements
- If you change or add webview UI (`web/src/**`), add or update tests under `web/src/**/*.test.tsx`.
- If you change extension-side UI wiring or view contributions (`src/extension.ts`, `src/panels/**`, related contribution metadata), add or update tests under `src/test/ui/**`.
- Minimum verification for UI-related changes:
  1. `npm run test:webview`
  2. `npm run test:ui`
- Prefer focused smoke tests that verify behavior/contracts over brittle snapshots.
- For regressions, add a test that fails before the fix and passes after it.
- UI copy must describe user value and actions, not internal implementation state.
  - Avoid phrasing like "because X input is empty" or other system-internal explanations.
  - Prefer direct labels/instructions such as "Installed Skills", "Manage...", and "Search above to install...".

## Entry Points
- Extension activation + registrations: `src/extension.ts`
  - Registers webviews: `copilotMcpView` (real UI) and `copilotMcpLauncherView` (launcher)
  - Registers chat participant id: `copilot.mcp-agent`
  - Configures logging + telemetry
- Webview frontend bootstrap: `web/src/main.tsx` -> `web/src/App.tsx` -> `web/src/components/MCPServers.tsx`

## View Placement (Current Contract)
- Real extension UI container: `viewsContainers.secondarySidebar` id `copilotMcpSidebar` in `package.json`.
- Activity Bar icon is a launcher container: `viewsContainers.activitybar` id `copilotMcpLauncher`.
- Launcher view id/type: `copilotMcpLauncherView`; real view id/type: `copilotMcpView`.
- Launcher behavior lives in `src/panels/ExtensionPanel.ts` and should immediately redirect/focus to the secondary container.

## Directory Map
- `src/extension.ts`: extension activation lifecycle.
- `src/panels/ExtensionPanel.ts`: webview backend; most RPC request/notification handling.
- `src/McpAgent.ts`: chat participant command handling (`/search`, `/install`) and README-to-install extraction.
- `src/utilities/repoSearch.ts`: GitHub repository search + README retrieval.
- `src/skills-client.ts`: skills search/list/install orchestration.
- `src/skills.ts`: SKILL.md discovery/parsing/filtering.
- `src/installer.ts`: skill install logic (symlink/copy, canonical paths, global/project scope).
- `src/agents.ts`: supported agent registry + install path detection.
- `src/source-parser.ts`: source string parsing (GitHub/GitLab/local/direct URL/well-known/git).
- `src/shared/types/rpcTypes.ts`: authoritative RPC message contracts.
- `src/telemetry/*` and `src/utilities/outputLogger.ts`: telemetry + output channel logging.
- `web/src/components/*`: sidebar UI and install/search flows.
- `web/src/utils/registryInstall.ts`: Official MCP Registry payload construction for VS Code/Claude/Codex installs.

## Feature Code Maps

### 1) GitHub MCP search (webview tab)
- UI:
  - `web/src/components/SearchMCPServers.tsx`
  - `web/src/components/SearchGitHubServers.tsx`
  - `web/src/components/RepoCard.tsx`
- RPC:
  - request type `searchServersType` in `src/shared/types/rpcTypes.ts`
- Backend:
  - request handler in `src/panels/ExtensionPanel.ts`
  - GitHub API query in `src/utilities/repoSearch.ts` (`searchMcpServers2`)

### 2) AI-assisted install from GitHub README
- UI trigger:
  - Install button in `web/src/components/RepoCard.tsx`
- RPC:
  - request type `aiAssistedSetupType` in `src/shared/types/rpcTypes.ts`
- Backend flow:
  - handler in `src/panels/ExtensionPanel.ts` (`vscodeLMResponse`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VikashLoomba/copilot-mcp](https://github.com/VikashLoomba/copilot-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
