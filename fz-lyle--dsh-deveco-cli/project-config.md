---
trigger: always_on
description: Guidance for AI coding assistants working in this repo.
---

# Agents.md

Guidance for AI coding assistants working in this repo.

## What this repo is

`deveco-cli` wraps the DevEco Studio toolchain (`ohpm`, `hvigor`, `hdc`, `emulator`, `hilog`, bundled `node` + JBR + SDK) plus a HarmonyOS skills installer and a project-scaffolding template engine — all behind a single `devecocli` binary. Distribution is one minified ESM bundle `dist/cli.js` (bin: `devecocli`).

Commands shipped: `build`, `run`, `update`, `device`, `emulator`, `skills`, `log`, `create`, `init`, `serve`, `docs`, `ui`, `check`, `auth`, `signature`.

User-facing invocation guide for AI agents lives in `SKILL.md` — update it whenever a command or flag changes.

## Commands

```bash
npm run build      # tsup → dist/cli.js (prepublishOnly runs lint → license → build)
npm run dev        # tsx watch mode
npm start          # tsx one-shot, no build
npm run lint       # eslint src
npm run lint:fix   # eslint --fix
npm run format     # prettier --write src
npm run license    # regenerate THIRD-PARTY-LICENSES
npm run build:index # regenerate index.zip for `devecocli docs` search
```

There is **no test framework** (no vitest/jest, no `*.test.ts`). Verification = `npm run lint` + manual smoke. `prepublishOnly` is the closest thing to CI locally.

## Conventions that bite

- **Modules**: ESM, `"type": "module"`. Internal imports **must** use `.js` even when the source is `.ts`.
- **Output bundle**: edits to `src/` are not visible until `npm run build` (or `npm run dev` / `npm start`).
- **Husky**: pre-commit runs `npm run lint`; `prepare-commit-msg` auto-appends `Signed-off-by:` (DCO).

## Env vars

- `DEVECO_CLI_DEBUG=1` — log raw `node` / `ohpm` / `hvigor` / `hdc` / `emulator` invocations via `utils/logger.ts → debugLog`. Also prints stack traces on error from `cli.ts`.
- `DEVECO_CLI_SKIP_VERSION_CHECK=1` — bypass the DevEco Studio 6.1.0+ version check on startup. (`update` is the only command exempt by default; see `TOOLCHAIN_FREE_COMMANDS` in `src/cli.ts`.)
- `DEVECO_CLI_DATA_DIR` — override user data root (default `~/.local/share/deveco-cli`). Derives `docs/.index/search.db`, `logs/doc-init.log`, `TraceLogData/upload-state.json`, etc.
- `DEVECO_CLI_CPP_ENABLED` — set to `false` or `0` to disable C++ LSP in the MCP server (skips `compileNative` + `clangd`; C++ tools return "disabled"). Default: enabled.
- `HTTP_PROXY` / `HTTPS_PROXY` — honoured by `global-agent` bootstrapped in `src/cli.ts`.

## Architecture

```
src/
├── cli.ts                    # Commander entry; global-agent bootstrap; preAction version check
├── commands/                 # One file per subcommand. Keep it thin: CLI shape + spinner + render.
├── compat/                   # compat command: SDK API compatibility scanning (single-file module — see "Compat module" below)
├── auth/                     # auth command: login, logout, status, team (login flow + encrypted token storage)
├── skills/                   # HarmonyOS skills marketplace client (api + installer + agents + mcp-installer)
├── service/                  # Domain helpers (device, emulator, doc)
├── utils/                    # Adapters (hdc, hilog, ohpm, hvigor) + shared validators
├── config/                   # constants, network, skills, mcp
├── data/                     # Bundled data files (e.g. emulator-privacy-bundled.ts)
├── types/                    # Shared type defs
├── toolchain/                # DevEco Studio toolchain resolution (ToolProvider: node/ohpm/hvigor/java/hdc/emulator/sdk)
└── internal/                 # Internal entry points (e.g. doc-init-background.ts, telemetry-upload-background.ts — all tsup entries)

mcp/src-server/               # Bundled stdio MCP server (ArkTS/C++ syntax checking via LSP)
templates/application/        # Project scaffold copied by `devecocli create`
scripts/                      # postinstall + docs index bootstrap
index/                        # Source for `npm run build:index` (regenerates index.zip)
```

**Real entrypoints**: `src/cli.ts` is the user-facing bin. `src/internal/doc-init-background.ts` is a tsup entry spawned after install to populate the docs search index. `src/internal/telemetry-upload-background.ts` is a tsup entry spawned (detached) by the CLI when `isUploadDue` is true, to flush telemetry events for non-MCP commands. `mcp/src-server/index.ts → createMcpServer` is the MCP orchestrator started by `devecocli serve mcp`.
**Toolchain resolution**: `toolchain/tool-provider.ts` finds DevEco Studio (Win: registry → `C:\Program Files\Huawei\DevEco Studio`; macOS: `~/Applications` + `/Applications` for `*DevEco*.app`; **Linux unsupported**). It resolves `nodePath` / `ohpmJsPath` / `hvigorJsPath` / `javaPath` / `hdcPath` / `emulatorPath` / `sdkPath`. hilog is not a separate binary — it runs through `hdc shell hilog`.
**Build pipeline**: `commands/build.ts` runs `ohpm install --all → hvigor --sync → hvigor assemble*`. The artifact path resolver lives in `utils/project.ts → findArtifactPath`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fz-lyle/dsh-deveco-cli](https://github.com/fz-lyle/dsh-deveco-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
