---
trigger: always_on
description: 本文件是 dsh 插件（`dsh-web-startup-auth`）的入场指南。**先读「速览」，再看「通用 dsh 插件知识」，最后读本插件专属部分。** 前两部分通用，可复用到任何 dsh 插件开发；第三部分只针对本项目。
---

# AGENTS.md

本文件是 dsh 插件（`dsh-web-startup-auth`）的入场指南。**先读「速览」，再看「通用 dsh 插件知识」，最后读本插件专属部分。** 前两部分通用，可复用到任何 dsh 插件开发；第三部分只针对本项目。

---

## 速览

- 这是一个 **dsh 插件包（bundle）**：**替换** dsh 原生的 Web 启动器 + 加一层登录认证，让 `dsh web --host 0.0.0.0` 可以安全地暴露到局域网/非回环接口。
- 由**三个插件入口**组成（`package.json` 的 `exports` 子路径分别暴露）：
  - `dsh-web-startup-auth/startup` → 插件 id `remote-web-startup`（`src/startup.ts`）：与原版 `@deepseek-ai/dsh-web-app/startup` 唯一区别是**不拒绝 `--host 0.0.0.0`**，提供同名 `webStartup` 服务。
  - `dsh-web-startup-auth/auth` → 插件 id `web-auth`（`src/auth.ts`）：登录/注册页、会话 cookie、`/api` 路由保护、`webAuth` 服务。
  - `dsh-web-startup-auth/client` → **前端插件**（`src/client/index.tsx`，打包产物 `lib/client.js`）：向 DSH 设置面板的 `settings.section` slot 注册「认证」标签页（退出登录 + 修改密码）。
- 插件 id：`remote-web-startup` / `web-auth`；npm 包名：`dsh-web-startup-auth`；目录：`/home/pax/coding/dsh-web-startup-auth`。
- 构建流水线：`src/*.ts` → `tsc` → `lib/*.js`，前端插件额外 `tsdown` → `lib/client.js`（**必须 `npm run build` 后插件才能加载**，`exports` 指向 `lib/`）。
- 测试：`npm test`（vitest），凭据文件用临时目录隔离（见「如何测试」）。
- 本 fork 的母体（原版 web-app bundle）在 `/home/pax/coding/research/deepseek-harness/packages/bundle/web-app/`，涉及对比/移植时先对照它。

---

## 第一部分：dsh 插件开发通用知识

### dsh 是什么

dsh 是 DeepSeek Harness（`/home/pax/coding/research/deepseek-harness`）的 CLI 入口，一个**基于 cordis 的插件式 agent 框架——「一切皆插件」**。用户用 `dsh web` 启动 web 应用；应用由一组插件 bundle 按层叠加组合而成。

- **cordis**（`@deepseek-ai/cordis`）是框架内核：插件在 `Context` 上注册服务、监听事件、注入依赖。关键 API：`ctx.provide(key, value)`、`ctx.get(key)`、`ctx.effect(fn, label?)`（挂副作用）、`ctx.logger`。
- **profile** 是一个可启动的插件组合，位于 `$DSH_HOME/profiles/<name>/`（默认 `~/.dsh/profiles/`）。常见 profile：`web`、`headless`、`tui`。
- **bundle** = 一个 npm 包 + 一张 patch 配置层（`package.json` 里 `dsh.bundle.patch` 指向 `cordis.patch.yml`）。安装进 profile 后 patch **自动应用**，无需手动编辑 profile 配置。

### 插件包的基本形态

一个 dsh 插件就是一个 npm 包（ESM，`"type": "module"`）。入口模块导出：

```ts
export const name = 'remote-web-startup'   // 插件 id（全局唯一，patch/配置用它定位）
export const inject = ['cmdlineArgs']      // 声明注入的能力名（缺失时插件不启动）
export function apply(ctx: Context, config) { /* 挂载逻辑 */ }
```

关键约定：

- **服务提供**：插件用 `ctx.provide('webStartup', values)` 提供服务，下游行（如 webserver、connection）通过 `inject` 或 `ctx.get('webStartup')` 消费。**服务名是契约**——替换插件必须提供同名同型服务，下游才能无感切换。
- **路由注册**：Web 类插件通过 `ctx.webServer.register(route)` 注册 HTTP 路由（route 有 `kind: 'exact' | 'prefix'`、`path`、`handler(req, res)`）；`webServer.tapIndex(fn)` 可改写 SPA 的 `index.html`。
- **生命周期**：`ctx.effect(fn, label)` 里的 fn 在插件激活后执行；`apply` 里抛错会中断整个 profile 启动。
- **命令行解析**：用 `@deepseek-ai/dsh-cmdline` 的 `parseCmdline(ctx, commanderProgram)`，把 commander 命令接到 dsh 的 `cmdlineArgs` 服务上。

### 前端插件（browser half）——给 DSH 界面注入 UI

DSH 的浏览器界面（SPA）**本身就是一组前端插件**：后端 `ClientModuleRegistry`（`packages/client/modules/src/index.ts`）扫描所有 loader entry 的 `package.json` 的 `dsh.client` 声明，组合成 `window.__DSH_BOOT__` 注入 `index.html`，浏览器端 loader 按图加载每个包的 `lib/client.js` 并执行其 `apply`。**想给 DSH 界面加东西（设置面板标签页、菜单、按钮等），走的不是 `webServer.register`，而是这个前端插件机制**——一个 npm 包可以同时有 node half（`exports["."]`）和 browser half（`exports["./client"]`）。

要点与**踩过的坑**：

- 声明：`package.json` 加 `dsh.client: { platform: "web", inject: [依赖的前端插件包名] }` 和 `exports["./client"]`（`dsh.client` 与 `dsh.bundle` 的 patch 层互不排斥，可并存）。
- 打包：`lib/client.js` 由 `tsdown` 打包，格式为 `window.__ModuleLoader__.load({ id: 包名, factory: (require) => {…} })`；`react` / `react/jsx-runtime` / `@deepseek-ai/cordis` / `dsh-client-ui-slots` 保持 external（loader 模块表提供），其余依赖内联。
- **关键坑（客户端包必须插"包根行"）**：`ClientModuleRegistry` 用 loader entry 的 `name` 字段当**包名**去 resolve `package.json` 读 `dsh.client`。因此 `cordis.patch.yml` 里必须**插入一条 `name` 为纯包名（包根，如 `name: dsh-web-startup-auth`）的 entry**——只插子路径（`name: xxx/startup`）时该包永远不被识别为 client 包，`dsh.client` 声明形同虚设（本插件踩过，见「设置面板标签页」）。包根入口（`lib/index.js`）需导出 `apply()`（可为空，模仿 `@deepseek-ai/dsh-client-ui-settings` 的 node half），loader 才能激活该行。
- 设置面板是 **slot 贡献点机制**：`ui-settings` 声明 `settings.section` 契约（`packages/client/ui-settings/src/client/contract/slots.ts`），前端插件用 `ctx.slots.inject('settings.section', …)` 注册标签页（参考 `ui-settings-models/src/client/index.ts:118`）。

### bundle patch 机制

dsh 的 profile 配置由多层 patch 叠加合成，`cordis.patch.yml` 就是插件的 patch 文件。顶层是 **YAML 数组**，每项一个 patch 条目（本项目 `cordis.patch.yml` 的四种写法全覆盖）：

```yaml
- id: web-startup          # 1. 按 id 禁用原插件
  disabled: true

- id: connection           # 2. 给现有行追加注入依赖
  inject: [webServer, webRuntime, webAuth]

- insert:                  # 3. 插入自己的插件
    - id: remote-web-startup
      name: dsh-web-startup-auth/startup
```

- `{ id, disabled: true }`：禁用某个插件。
- `{ id, inject: [...] }`：给某个已有行追加注入的能力名。
- `{ insert: [{ id, name }] }`：插入插件（`name` 是 npm 包名 + `/子路径`；**前端插件包必须插纯包名「包根行」**，见「前端插件（browser half）」）。
- patch 里允许 `!!js` 表达式（仅限 config 值和 disabled 字段），其他元数据保持字面量。

### profile 组成与插件安装/卸载

一个 profile 目录（如 `~/.dsh/profiles/web/`）里：

| 文件 | 作用 |
|---|---|
| `cordis.yml` | profile 根，通常是空数组；**不要直接编辑** |
| `cordis.patch.yml` | 用户 patch 层（组合顺序在所有 bundle 之后） |
| `package.json` | `dsh.profile.bundles` 数组列出该 profile 启用的 bundle；`dependencies` 里是插件包本体（本地路径用 `link:/abs/path`） |
| `node_modules/` | pnpm 安装的依赖（按 profile 各自安装） |
| `pnpm-lock.yaml` / `pnpm-workspace.yaml` | 安装锁 |

**CLI（唯一子命令 `plugin`，转发给 profile 目录里的 pnpm，`--profile` 必填）：**

```sh
cd /path/to/plugin-package

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GDWhisper/dsh-web-startup-auth](https://github.com/GDWhisper/dsh-web-startup-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
