---
trigger: always_on
description: 面向 AI 协作者的开发约定与注意事项。本文件是给编码代理（Codex、opencode、Cursor 等）读取的，用于在改代码前快速了解本项目的构建方式、目录约定与常见坑，减少到处搜索。
---

# AGENTS.md

面向 AI 协作者的开发约定与注意事项。本文件是给编码代理（Codex、opencode、Cursor 等）读取的，用于在改代码前快速了解本项目的构建方式、目录约定与常见坑，减少到处搜索。

## 项目是什么

Ethereal 是一款基于 Astro 构建的 **Halo CMS 主题**。它先用 Astro 编写组件与模板，构建后输出为 Halo 使用的 **Thymeleaf 模板**，再由 Halo（Spring Boot + Thymeleaf）在服务端渲染最终页面。

核心心智模型：**你在 `.astro` 文件里写的 `th:xxx` 属性不是前端语法，而是给 Thymeleaf 模板引擎用的指令**，构建后原样保留在 `templates/*.html` 里。

技术栈：**Astro**（页面/路由）+ **Svelte 5**（交互组件，如 Search、LightDarkSwitch）+ **Tailwind CSS 4** + **TypeScript** + **Swup**（页面过渡动画）+ **Iconify**（图标）。

## 开发环境

- 需要 **Node.js >= 22.12.0**（推荐 24.x，见 `.nvmrc`）与 **pnpm**。
- `pnpm dev`：监听 `src/` 文件变更自动重建（不会打包 zip）。

## 构建与校验命令

| 命令               | 作用                                                       |
| ------------------ | ---------------------------------------------------------- |
| `pnpm build:only`  | 仅执行 `astro build`，输出到 `templates/`（开发调试常用）  |
| `pnpm build`       | `astro build` + `pnpm package`（打成发布 zip）             |
| `pnpm astro check` | 类型检查，务必在改动后运行确认 0 error                     |
| `pnpm format`      | prettier 格式化全项目，随后自动刷新 README-Halo.md         |
| `pnpm readme:halo` | 单独触发 README→README-Halo 转换（见「README-Halo 转换」） |

**重要：改完代码后运行的校验是 `pnpm astro check` 和 `pnpm build:only`。** 两类产物的位置不同：`astro build` 的 HTML 模板输出到 `templates/`；`pnpm package` 打出的发布 zip 输出到 `dist/`。两者都是构建生成、勿手动编辑——要改就改 `src/` 后重新构建。

**经典脚本资产管线**（I27 引入，`astro.config.mjs` 的 `buildAssets` integration）：

```
src/scripts/assets/*.ts →(esbuild IIFE, build:start)→ public/assets/*.js
src/scripts/vendor/*.js →(原样拷贝, build:start)→ public/assets/*.js
→(astro 拷贝 public/)→ templates/assets/*.js →(esbuild 压缩, build:done)→ 产物
```

- **`src/scripts/assets/` 是全部经典脚本源码（含 `// @ts-nocheck` 的 legacy 脚本），`public/assets/` 是纯产物目录，勿手改**。legacy 脚本（wave/navbar/wishes/upvote/banner-* 等）迁入后统一 `.ts` 后缀（兼容 nodemon watch，esbuild 照常编译）。
- `src/scripts/vendor/` 存放第三方 vendored 资产（如 qrcode.bundle.js UMD），构建期原样拷贝、不经过 esbuild 编译。
- `_` 前缀文件（如 `_theme-config.ts`）是被 import 的共享模块，不是独立入口，esbuild 会内联进各入口。
- 产物带 `/*__ETHEMEAL_MINIFIED__*/` 标记；build:done 只压缩白名单内 public 产物，不碰 Astro/Vite 的 hashed module 文件。
- **nodemon 的 ext 不含 js 是有意的**：编译产物写入 public/ 不会触发重建（防编译→重建死循环）。不要给 nodemon.json 加 js；改脚本源码统一用 `.ts` 后缀。

## README-Halo 转换

Halo 应用市场的 Markdown 渲染器不支持 `<picture>`（GitHub 深浅色徽章），`scripts/convert-readme.mjs` 把根目录 README.md 转换为降级版 README-Halo.md（每个 `<picture>` 块替换为内部首个 `<img>`，即浅色徽章）。

- **README-Halo.md 是生成产物**：已进 `.gitignore` 与 `.prettierignore`，不提交、勿手改；改徽章只改 README.md 源文件后重新转换。
- **触发方式三选一，产物一致**：`pnpm readme:halo` 单独触发；`pnpm format` 链尾自动刷新；提交涉及 README.md 时 pre-commit（lint-staged 的 `README.md` 任务）自动重新生成本地文件。
- 脚本路径基于 `import.meta.url` 解析（pnpm 脚本固定以包根为 CWD，按 `../README.md` 相对 CWD 写会指向项目外），任意目录下均可运行。
- 该文件仅供发布时手工粘贴到 halo.run 开发者后台的应用详情页——商店版本说明走 GitHub Release body（ci.yaml 用 release.md），与它无关。

## 目录结构速览

- `src/pages/*.astro` — 页面模板（`post.astro`、`index.astro`、`category.astro` 等）
- `src/components/*.astro` / `*.svelte` — 可复用组件（`PostCard.astro`、`PostList.astro` 等）
- `src/components/control/` — 分组页共享控件：`FilterTab.astro` / `FilterTabs.astro`（分组筛选 tab）/ `PageHeader.astro`（页头），把 Thymeleaf 表达式当字符串 prop 传（见「组件表达式 prop 约定」）
- `src/layouts/*.astro` — 页面布局（`Layout.astro`、`MainGridLayout.astro`）
- `src/styles/*.css` — 全局样式与 CSS 变量（`variables.css` 定义主题色/圆角等）
- `src/types/config.ts` — `theme.config` 的类型定义
- `src/utils/*.ts` — 工具（`post-list-config.ts`、`image-suffix.ts`）
- `src/scripts/assets/*.ts` — 经典脚本源码（全部脚本统一管理，esbuild 编译为 IIFE 输出到 `public/assets/`，见「构建与校验命令」）
- `src/scripts/vendor/` — 第三方 vendored 资产（原样拷贝，不经编译）
- `settings.yaml` — **后台主题设置表单**（改后台开关/设置项在这里）
- `theme.yaml` — 主题元信息，**版本号唯一来源**（`version` 字段）。**禁止修改 `version` 字段**：版本号只能由发布流程手工提升，AI 不得改动，否则会造成线上主题版本错乱。
- `i18n/` — 多语言文案
- `templates/` — 构建产物（勿手改，会被 `astro build` 覆盖）

## 主题设置（settings.yaml → theme.config）

`settings.yaml` 里每个 `group` 对应后台的一个设置页分组；每个 `name` 字段会出现在前端的 `theme.config?.<group>?.<name>`。

**改动设置项需要同步修改的地方（务必三处一致）：**

1. `settings.yaml` — 新增/修改表单项
2. `src/types/config.ts` — 给对应 interface 增加字段
3. 使用它的 `.astro` 模板 — 通过 `theme.config?.xxx?.yyy` 读取

读取默认值时用安全导航，例如 `theme.config?.layout?.postList?.descriptionLines == 0`。

## Halo/Thymeleaf 特有约定

- `th:text`（输出文本）、`th:if` / `th:unless`（条件）、`th:each`（循环）、`th:href`（链接）、`th:classappend`（追加类）——这些是 Thymeleaf 指令，不是前端属性。
- 模板变量来自 Halo 的 Finder API 与上下文：`post`、`posts`、`site`、`theme.config`、`theme.metadata` 等。
- `theme.config?.xxx` 用 `?.` 安全导航；字面值可写成 `|${...}|` 拼接。
- 静态资源用 `#theme.assets("/assets/...")` 或 `@{/assets/...}` 引用，构建后路径带 `/themes/Ethereal` 前缀。
- 图片拼 CDN 参数使用 `imageSuffixThWith(...)`（见 `src/utils/image-suffix.ts`），不要在模板里手写硬编码后缀。

## 组件表达式 prop 约定

`src/components/control/` 下的 `FilterTab.astro` / `FilterTabs.astro` / `PageHeader.astro` 把 Thymeleaf 表达式当**字符串 prop** 传，约定如下（务必遵守，否则只在 Halo 渲染期才暴露错误）：

- `activeExpr` / `allActiveExpr` 传**裸布尔表达式**（无 `${}`，如 `#lists.contains(param.group, group.spec.displayName)`）。组件会把它注入 `th:classappend` 的三元追加激活类。
- 其余表达式 prop（`hrefExpr` / `labelExpr` / `countExpr` / `showIfExpr` / `withExpr` / `countShowIfExpr` / `subExpr` / `titleExpr` / `filteredExpr` / `sepShowIfExpr` 及 `all*` 系列）传**完整表达式**（含 `${}` 或 `#{}`）。
- 动态 tab 列表用 `<div class="contents" th:each=...>` 包裹（组件标签上的 `th:each` 不会转发到根元素，故不能放 FilterTab 自身）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AloneNanNan/Halo-Theme-Ethereal](https://github.com/AloneNanNan/Halo-Theme-Ethereal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
