---
trigger: always_on
description: Agent 应在必要时更新本文件，但必须非常慎重，确保内容与仓库现状一致。若发现文档与代码不符，应优先修正文档中的错误描述，而不是延续旧说法。
---

# AGENTS.md - 夸克博客项目仓库

Agent 应在必要时更新本文件，但必须非常慎重，确保内容与仓库现状一致。若发现文档与代码不符，应优先修正文档中的错误描述，而不是延续旧说法。

- 应检查 `/src/docs/dev` 是否有未完成的 BUG 修复或功能更新任务
- 在进行重大架构调整或引入了新的第三方服务/API等情况下，应同步更新`/posts/copyright.md`
- 同样，在必要时检视项目介绍文档 `README.md` 并更新
- 根据下述要求，在必要时于项目说明文档 `/src/docs` 处或其他合适位置更新、新建说明文档

## 项目概述
- **项目名称**：夸克博客
- **简介**：基于 Astro 构建的静态 + 动态结合博客网站
- **仓库地址**：https://github.com/lsqkk/lsqkk.github.io
- **网站地址**：https://lsqkk.github.io
- **本地地址**：`D:\git\lsqkk\lsqkk.github.io`

## 技术栈
- **框架**：Astro 5
- **内容系统**：Markdown + `astro:content`
- **包管理器**：npm
- **部署平台**：
  - GitHub Pages 托管静态站点
  - Vercel 仅用于 `/api` serverless，不负责本站静态构建
- **数据库**：Firebase Realtime Database (RTDB)
- **网站登录认证**：GitHub OAuth + 站内账号
- **前端技术**：原生 JavaScript (ES6+、@ts-check + JSDoc)、Canvas、WebRTC、CSS3、TypeScript（渐进式）
- **后端语言**：Node.js (Vercel Serverless)、Python (Quark CLI)

## 目录结构
```text
lsqkk.github.io/
├── api/                         # Vercel serverless 函数
├── assets/                      # 主静态资源源目录（构建时复制到 public/assets）
│   ├── css/                     # 公共样式
│   ├── js/                      # 公共脚本与共享逻辑
│   ├── md/                      # 运行时会读取的 Markdown 数据，如 dt.md / log.md
│   ├── pages/                   # 各功能页专属资源（a/blog/games/tool）
│   ├── img/                     # 背景、logo、头像等图片
│   └── ...
├── posts/                       # 博客文章源文件与生成产物 posts.json / rss.xml
├── public/                      # 构建前同步出的公共静态目录（生成目录，不建议手改）
├── src/
│   ├── components/              # Astro 组件
│   ├── config/
│   │   ├── json/                # 站点配置源
│   │   └── site.js              # 从配置中派生站点标题 / logo / 背景
│   ├── docs/                    # 项目说明文档
│   │   ├── API/                 # serverless 与密钥相关文档
│   │   ├── 页面与功能/           # 页面功能相关文档
│   │   ├── dev/                 # 待修复的 BUG 和待实现的功能新增或修改
│   │   ├── UI/                  # 可复用的界面风格与布局规范
│   │   └── page_template/       # 页面模板与样式规范
│   ├── layouts/                 # 页面布局，如 Post/OJ/FireAlert/RealtimeRoom
│   ├── pages/                   # Astro 路由页面
│   └── utils/                   # 构建与页面工具函数（如 home-utils.ts、dynamic-entries.ts 等）
├── scripts/                     # Node 构建/同步/校验脚本
├── quark/                       # Python CLI 与配置编辑器
├── private/                     # 本地私有辅助文件，已 gitignore，不应提交
├── dist/                        # Astro 构建产物
├── .quark-artifact/             # quark build --mode artifact 导出的额外产物
├── astro.config.mjs             # Astro 配置与构建后注入逻辑
├── package.json
├── pyproject.toml               # quark CLI 包定义
├── requirements.txt
└── vercel.json                  # Vercel 仅保留 API，不做静态站点构建
```

## 关键事实与维护约定

### 1. 项目文档
- 添加或修改功能后，优先在以下位置补充说明：
  - `src/docs/xxx.md`
  - `src/docs/API/`
  - `src/docs/UI/`
  - `src/docs/page_template/`

### 2. 生成目录与禁止手改项
- 以下目录/文件主要由构建或脚本生成，除非明确在修生成逻辑，否则不要直接手改：
  - `public/`
  - `dist/`
  - `.quark-artifact/`
  - `posts/posts.json`
  - `posts/rss.xml`
  - `public/json/posts.json`
  - `public/assets/js/fa-icons.js`
  - `public/assets/js/fa-icons.json`
  - `dist/json/site-pages.json`

### 3. 私有目录与敏感信息
- `private/` 已被 `.gitignore` 忽略，用于本地私有脚本和中间数据，不应纳入提交。
- `.env`、`.env.local` 等环境变量文件不提交。
- `src/config/json/api.json` 只保存 API 基础地址，不存放真正密钥。

## 硬编码预防规范（新增/修改代码时必读）

### 核心原则

任何可能因环境、部署、用户或时间变化的值，都不应硬编码在代码中。

### 三步自查流程

新增或修改代码时，按此顺序检查每个值：

**第一步：是否已有对应的 config JSON？**
- 检查 `src/config/json/` 目录下是否有合适的配置文件
- 这些文件在构建时通过 `scripts/sync-public.mjs` 自动同步到 `public/json/`，前端可通过 fetch(`/json/xxx.json`) 读取
- 优先在现有配置文件中新增字段，而不是新建文件

**第二步：是否需要在 Astro 模板/服务端使用？**
- 通过 `src/config/site.js` 导出（构建时读取 config JSON 生成）
- 在 Astro 组件中 `import { XXX } from "@siteConfig"` 使用
- 示例：`PostLayout.astro` 中读取 `SITE_URL`、`AUTHOR_NAME` 等

**第三步：是否需要在浏览器端 JS 中使用？**
- 方案 A（推荐）：使用 `__API_BASE__` 占位符，构建时自动注入实际值
  - 在 JS 文件中写 `const BASE = "__API_BASE__";`
  - `astro.config.mjs` 和 `scripts/sync-public.mjs` 都会在构建后替换为 `api.json` 中的 `apiBase`
- 方案 B：使用 `define:vars` 在 Astro 模板中传递
  - 在 Astro 页面中 `<script define:vars={{ myValue }}>` 注入
  - 适用于 Astro 构建时可获取的值
- 方案 C：运行时从 `/json/*.json` fetch
  - 适用于运行时可以异步获取的配置
  - 例：`fetch('/json/giscus.json')`

### 什么必须进 config JSON

| 类型 | 示例 | 存放位置 |
|------|------|----------|
| API 地址 | `https://api.130923.xyz` | `api.json` → `apiBase` |
| 站点 URL | `https://lsqkk.github.io` | `api.json` → `siteUrl`（或 `site.js` 硬编码底线） |
| OAuth 相关 | GitHub OAuth client ID | 环境变量（不提交），通过 serverless 使用 |
| 用户标识 | Bilibili UID `2105459088` | `index.json` 的 `socialLinks` |
| 第三方服务配置 | Giscus repo/category ID | `giscus.json` |
| 评论系统主题 | Giscus theme | `giscus.json` |
| 站点导航 | 导航栏标题、链接 | `nav.json` |
| 个人信息 | 昵称、简介、头像 | `index.json` |
| 友链 | 朋友博客链接 | `friends.json` |
| PWA 配置 | manifest 字段 | `manifest.json` |
| 字体配置 | 字体族、来源 | `font.json` |
| 弹窗 | 首页弹窗内容 | `popups.json` |
| CORS 域名列表 | 允许的跨域来源 | `api/_cors.js` 的 `ALLOWED_DOMAINS` / `ALLOWED_ORIGINS` |

### 什么可以硬编码（底线值）

- `src/config/site.js` 中的 fallback 字符串（当 config JSON 读取失败时的兜底）
- CSS 中的视觉设计值（颜色变量已在 `tokens.css` 中定义，不应再硬编码颜色值）
- CDN 静态资源 URL（如 KaTeX、highlight.js 等，更换成本较高）

### 新增 API serverless 函数的约束

- Vercel 免费版 = 最多 12 个 serverless 函数
- 纯配置数据 → 放在 `src/config/json/`，不要使用 serverless 函数
- 新增 API 文件必须复用 `api/_cors.js` 中的 CORS 函数：
  - `allowOrigin(req, res, origins?)` — 精确 origin 匹配，用于 admin-auth 等需要严格控制的 API
  - `resolveOrigin(req, domains?)` — host 匹配（返回允许的 origin 或空字符串），用于 db、github-user 等

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsqkk/lsqkk.github.io](https://github.com/lsqkk/lsqkk.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
