---
trigger: always_on
description: 本文档为 ClawUI 项目开发提供基本的规范指南。
---

# ClawUI 开发基本指南
本文档为 ClawUI 项目开发提供基本的规范指南。

## 项目概述
ClawUI 是一个为 OpenClaw 开发的跨平台桌面应用，基于 Electron、React、TypeScript、Antd、AgentScope Spark Design 和 Vite 构建。

## 技术栈

| 分类 | 技术 | 版本 |
|------|------|------|
| 桌面运行时 | Electron | 40.x |
| 前端框架 | React | 19.x |
| 开发语言 | TypeScript | 5.9.x |
| 构建工具 | Vite | 7.x |
| 包管理器 | pnpm | 10.30.x |
| UI 组件 | @agentscope-ai/chat, @agentscope-ai/design, antd | - |
| WebSocket | ws | 8.x |

## 编码规范

### 代码格式化 (Prettier)

```yaml
printWidth: 100         # 行宽 100
tabWidth: 3             # 缩进宽度 3 空格
```

### TypeScript 配置

- 目标：ESNext
- 模块：ESNext (bundler 解析)
- 严格模式：启用
- JSX：react-jsx (自动转换)

### 资产放置
- 所需要的资产都放到：src/renderer/assets/目录下

## 注意事项

1. **错误处理**：所有异步操作需要 try-catch 包裹
2. **日志格式**：使用 `[ModuleName]` 前缀，如 `[GatewayClient]`
3. **中文支持**：用户界面文本使用中文
4. **类型安全**：避免使用 `any`，优先使用 `unknown` + 类型守卫
5. **清理监听器**：组件卸载时清理事件监听器，避免内存泄漏
6. **不准修改 OpenClaw 代码**：ClawUI 是 OpenClaw 的下游应用，所有功能变更都应在 ClawUI 端实现。如果 OpenClaw 行为不符合预期，优先通过纯 ClawUI 端方案解决（如 hook、包装层、事件拦截等），而不是直接修改 OpenClaw 源码

## 最佳实践

### 替换第三方包中的远程图标

`@agentscope-ai/chat` 的 `Attachments` 组件内部使用 alicdn 远程 URL 作为文件类型图标。Electron 离线环境下这些 URL 不可用，需要替换为本地 SVG。

**实现方式：** 通过 Vite 插件 `src/renderer/plugins/replace-file-icons.ts` 在构建时完成替换：

- **本地图标位置**：`src/renderer/assets/icons/` 目录下的 SVG 文件
- **dev 模式**：通过 `optimizeDeps.esbuildOptions.plugins` 注入 esbuild 插件，在依赖预打包阶段将 alicdn URL 替换为内联 `data:image/svg+xml` data URL
- **production 构建**：通过 Rollup `transform` 钩子做同样替换

**注意事项：**
- 不要使用 `optimizeDeps.exclude` 排除 `@agentscope-ai/chat`，其内部依赖（如 `rc-util`）有 CJS/ESM 兼容问题，会导致白屏
- 新增文件类型图标时，需同时在 `ICON_MAP` 中添加 alicdn URL 映射和对应的本地 SVG 文件
- 如果上游包更新了图标 URL，插件会在控制台输出警告，需同步更新 `ICON_MAP`

## 参考文档

- 如果想知道 OpenClaw Gateway 协议怎么定义的，参考：../openclaw/src/gateway/protocol（或 OpenClaw 仓库的 src/gateway/protocol 目录）
- 如果想知道怎么连接到 OpenClaw gateway 的，参考 OpenClaw UI 的实现：../openclaw/ui（或 OpenClaw 仓库的 ui 目录）
- 如果要了解 agentscope-ai 的使用，先参考内容大纲：docs/agentscope-ai/index，然后再参考详细内容：docs/agentscope-ai/all
- 如果想参考 agentscope-ai 怎么实现的，参考源码：../agentscope-spark-design（需本地克隆该仓库）
- OpenClaw 项目 github ：https://github.com/openclaw/openclaw

## 内置 OpenClaw Gateway 打包指南

ClawUI 内置了 OpenClaw Gateway 子进程，打包时需要将 OpenClaw 的构建产物嵌入 Electron 应用。本节记录完整的打包方案和已知问题的解决方式。

### 架构概览

```
OpenClaw 源码 (../openclaw)
  ├── openclaw.mjs          # CLI 入口
  ├── dist/                  # tsdown 构建产物（~500 个 ESM chunk）
  ├── assets/                # 静态资源
  └── package.json           # 含 ~50+ npm 依赖

    ↓  scripts/prepare-openclaw.ts

resources/openclaw/          # 打包后的产物（无 node_modules）
  ├── openclaw.mjs
  ├── dist/                  # esbuild 二次打包后的 chunk（依赖已内联）
  ├── assets/
  ├── docs/reference/templates/  # Gateway workspace 模板（AGENTS.md 等，运行时必需）
  └── package.json           # 仅含 {"type":"module"}

    ↓  electron-builder extraResources

<App>/Contents/Resources/openclaw/   # 最终在 .app 内的位置
```

### 核心问题：为什么需要 esbuild 二次打包

electron-builder 的 `extraResources` 功能**硬编码排除 `node_modules/` 目录**，无法通过配置绕过。而 OpenClaw 的 dist/ 文件中包含大量裸模块导入（tslog, ws, zod, express, chalk 等 50+ 个包），运行时如果找不到 node_modules 就会报 `ERR_MODULE_NOT_FOUND`。

**解决方案**：在 `scripts/prepare-openclaw.ts` 中用 esbuild 对 OpenClaw 的 dist/ 做二次打包，将所有 npm 依赖内联到 bundle 中，然后删除 node_modules。

**已尝试但失败的方案（不要重试）：**
- `.npmignore` 空文件方案：无法阻止 electron-builder 的硬编码排除
- `node_vendor` 重命名方案：electron-builder 的排除逻辑不仅针对目录名

### 打包流程 (`scripts/prepare-openclaw.ts`)

1. **复制产物**：从 OpenClaw 源码目录复制 `openclaw.mjs`、`dist/`、`assets/`、`docs/reference/templates/` 到 `resources/openclaw/`
2. **安装依赖**：复制精简版 `package.json`（仅 dependencies），运行 `npm install --production` 安装到临时 `node_modules`（供 esbuild 解析用）
3. **esbuild 二次打包**：
   - 入口：`dist/entry.js`
   - 配置：`bundle: true`, `platform: 'node'`, `format: 'esm'`, `splitting: true`
   - 产出写入 `dist-bundled/`，然后替换原 `dist/`
4. **清理**：删除 `node_modules`，将 `package.json` 替换为 `{"type":"module"}`

### esbuild 配置中的关键设计决策

#### 1. EXTERNAL_PACKAGES（外部化列表）

以下包不会被内联，因为是原生模块或 Gateway 不需要的超大包（OpenClaw 内部会优雅降级）：

```
sharp, @img/*, koffi, @lydell/node-pty*, @napi-rs/canvas*,
node-llama-cpp, @node-llama-cpp/*, sqlite-vec,
@matrix-org/matrix-sdk-crypto-nodejs, playwright-core, pdfjs-dist,
@microsoft/*, @lancedb/lancedb,
@aws-sdk/*,
@larksuiteoapi/node-sdk, @buape/carbon, @slack/*,
grammy, @grammyjs/*,
silk-wasm, mpg123-decoder
```

**维护规则**：如果 OpenClaw 新增了原生模块依赖，需要添加到此列表。判断标准：包含 `.node` 二进制文件的包必须外部化。不在 OpenClaw `dependencies` 中的可选集成包（通道、云服务 SDK、媒体处理等）也需要外部化，否则 esbuild 无法解析。

#### 2. createRequire Banner 注入

```javascript
banner: {
   js: "import { createRequire as __cr } from 'node:module'; const require = __cr(import.meta.url);",
}
```

**原因**：esbuild 将 CJS 模块转换为 ESM 输出时，CJS 中的 `require()` 调用会变成 esbuild 的 `__require` shim，无法处理 `require("node:assert")` 等 Node.js 内置模块。注入 `createRequire` 后提供真正的 `require` 函数。

**症状**：如果缺少此 banner，运行时会报 `Dynamic require of "node:assert" is not supported`。

#### 3. tsdown require 修复插件 (`createTsdownRequireFixPlugin`)

tsdown 的 CJS 互操作会生成两种 esbuild 无法识别的 require 模式：

**模式 1：`__require("xxx")`**
tsdown 自己的 CJS shim，定义为 `var __require = createRequire(import.meta.url)`。

**模式 2：`require$N("xxx")`**（如 `require$1("ajv")`）
tsdown 通过 `const require$1 = createRequire(import.meta.url)` 创建的带编号 require 变量，用于内联 CJS 模块到 ESM 输出中。

esbuild 只能静态分析标准的 `require("xxx")` 调用。这两种非标准模式会被原样保留，打包后 node_modules 被删除就会报 `Cannot find module 'xxx'`。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawangcoding/ClawUI](https://github.com/dawangcoding/ClawUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
