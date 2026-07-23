---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **See also**: `AGENTS.md` for the universal agent entry point; `.github/copilot-instructions.md` for detailed API patterns and plugin examples.

## Project Overview

Zhin.js is a TypeScript chatbot framework — AI-driven, plugin-based, hot-reload, multi-platform. ESM-only (`"type": "module"`), targets Node.js ^20.19.0 || >=22.12.0.

## Commands

```bash
pnpm build              # Full build (Turborepo: basic/ → packages/ → plugins/)
pnpm test               # Run all Vitest tests
pnpm test:watch         # Watch mode
pnpm test:coverage      # Coverage report
pnpm lint               # ESLint (.ts,.tsx)
pnpm type-check         # tsc --noEmit -p tsconfig.typecheck.json
pnpm dev                # Start minimal-bot (examples/minimal-bot); use pnpm dev:test for test-bot
pnpm clean              # Clean all lib/ dist/ directories

# Bootstrap (first-time or after clean)
pnpm prepare:logger     # Build @zhin.js/logger + install CLI deps
pnpm prepare:cli        # Build scaffold-wizard + CLI + reinstall

# Single package
pnpm --filter @zhin.js/core build
pnpm --filter @zhin.js/core test

# Run a single test file
pnpm vitest run packages/im/core/tests/plugin.test.ts
```

Custom lint checks:
- `pnpm check:harness-paths` — 检测插件是否绕过 Adapter.sendMessage 直调 bot.$sendMessage
- `pnpm check:no-koa` — 检测插件是否直接 import koa（应使用 RouterContext）
- `pnpm check:prod` — 检查生产环境配置（无 console.log/debugger/TODO/FIXME）
- `pnpm check:plugin` — 检查插件是否符合标准规范（入口文件、测试、README）
- `pnpm check:use-plugin-top-level` — 检测 usePlugin() 是否在模块顶层调用
- `pnpm check:get-plugin-runtime` — 检测 getPlugin() 是否在运行时回调中调用
- `pnpm check:plugin-agent-publish` — 带 agent/ 的插件发布清单（files、prepublishOnly）
- `pnpm check:all` — 运行所有 harness 检查（含 type-check / lint / test）

## Architecture

Monorepo managed with **pnpm 9.0.2 workspaces** + **Turborepo** (`turbo.json`). Versioning via **Changesets**.

Build pipeline (`turbo.json`): `build` depends on `^build` (topological), outputs `lib/**` and `dist/**`. `test` depends on `build`. `type-check` depends on `^build`.

### Dependency layers (bottom → top)

```
basic/                      # @zhin.js/logger, schema, database, cli
  ↓
packages/im/kernel          # Runtime kernel (no IM concepts)
  ↓
packages/im/ai              # AI engine (providers, agents, memory, compaction)
  ↓
packages/im/core            # IM framework (Plugin, Adapter, Endpoint, Command, MessageDispatcher)
  ↓
packages/im/agent           # Agent orchestration (ZhinAgent, security policies, MCP client)
  ↓
packages/im/zhin            # Main entry — IM core (4.x); agent via optional peer + zhin.js/agent

packages/console/{contract,pagemanager,client}  # 控制台栈（平行，不经 IM 发送链）
packages/toolkit/{create-zhin,satori}         # 脚手架与渲染库
```

**禁止的导入**：kernel 不能导入 ai/core/agent/zhin；ai 不能导入 core/agent/zhin；core 不能导入 agent/zhin；插件不能直接导入 kernel（应通过 `@zhin.js/core`）。

### Key packages at a glance

| Package | Path | Role |
|---------|------|------|
| kernel | `packages/im/kernel/src/` | PluginBase, Feature, Cron, Scheduler, error hierarchy |
| ai | `packages/im/ai/src/` | Provider abstraction, Agent, ModelRegistry, Memory, Compaction, CostTracker |
| core | `packages/im/core/src/` | Plugin (AsyncLocalStorage), Adapter, Endpoint, Command, MessageDispatcher |
| agent | `packages/im/agent/src/` | ZhinAgent orchestrator, security (ExecPolicy, FilePolicy), MCP client |
| host-router | `packages/host/router/src/` | Koa 监听、Router、Bearer/CORS |
| host-api | `packages/host/api/src/` | Host 管理面 REST、Console 协议、entries |

### Outbound send chain (do not bypass)

`Message.$reply` / `Adapter.sendMessage` → `renderSendMessage` → root plugin `before.sendMessage` → platform `Endpoint`. No parallel `Plugin#sendMessage` bypass.

### Plugin system

Uses **AsyncLocalStorage** for context management. `usePlugin()` must be called at module top-level (not inside async functions). Plugins auto-mount on `start()`, unmount on `stop()`.

Context/DI: `provide()` registers, `inject()` / `useContext()` consumes. Return a cleanup function from `useContext` callback for lifecycle management.

### Build output

- Most packages: `src/` → `lib/` (via `tsc`)
- Client package: `src/` → `dist/`
- Dual exports: `./lib/index.js` (production) + `./src/index.ts` (development condition)
- Module resolution uses `conditions: ['development']` in tests to resolve `src/` directly

### TypeScript configuration

All packages extend `tsconfig.base.json`（根目录），只需设置 `outDir`、`rootDir`、`types`：

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "types": ["node"],
    "outDir": "./lib",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["lib", "node_modules"]
}
```

`pnpm type-check` uses a separate `tsconfig.typecheck.json` with explicit path aliases and limited scope (`basic/*/src` + `packages/im/*/src` only).

## Testing

**Vitest 4.x** with globals enabled (no need to import `describe`, `it`, `expect`).

Key config (`vitest.config.ts`):
- Environment: `node`
- Isolation: `false` locally, `true` in CI (avoids `vi.spyOn` leakage across files)
- Timeout: 10s
- Test pattern: `**/*.test.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhinjs/zhin](https://github.com/zhinjs/zhin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
