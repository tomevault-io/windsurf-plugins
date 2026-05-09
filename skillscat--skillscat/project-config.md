---
trigger: always_on
description: SkillsCat 是一个 `pnpm` monorepo，目标是构建一个用于发现、发布、安装 AI agent skills 的平台。
---

# AGENTS.md

## 项目定位

SkillsCat 是一个 `pnpm` monorepo，目标是构建一个用于发现、发布、安装 AI agent skills 的平台。

- `apps/web` 是主站和 API，运行在 Cloudflare Workers 上。
- `apps/cli` 是 `skillscat` CLI，用于搜索、安装、发布、更新 skills。
- `scripts` 放初始化、部署、发布、资源同步脚本。

以当前代码和配置为准，不要盲信旧文档：

- 根目录 `README.md` 提到了 Turborepo，但仓库当前没有 `turbo` 配置，真实工作区由 `pnpm-workspace.yaml` 和各应用 `package.json` 驱动。
- [apps/web/README.md](/Users/orchiliao/Projects/skillscat/apps/web/README.md) 仍是默认 Svelte 模板，不是这个项目的真实说明。

## Monorepo 结构

- [apps/web](/Users/orchiliao/Projects/skillscat/apps/web): SvelteKit 站点、SSR、API、Cloudflare Worker 配置、D1 migrations、后台 workers。
- [apps/cli](/Users/orchiliao/Projects/skillscat/apps/cli): Commander CLI，支持 add/search/publish/login/update 等命令。
- [scripts](/Users/orchiliao/Projects/skillscat/scripts): 项目初始化、Web 发布、CLI 发布、Workers 部署、Cloudflare secrets/资源管理。
- [apps/web/src/lib/server/db/schema.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/db/schema.ts): D1/Drizzle schema source of truth。
- [apps/web/migrations](/Users/orchiliao/Projects/skillscat/apps/web/migrations): Drizzle 生成的 migration 和 meta snapshots。

## 实际技术栈

### Web

- SvelteKit 2
- Svelte 5
- `@sveltejs/adapter-cloudflare`
- TypeScript
- UnoCSS
- Better Auth
- Drizzle ORM + Cloudflare D1
- Vitest

### CLI

- Commander
- Rollup
- TypeScript
- Vitest

### Cloudflare 基础设施

项目是 Cloudflare-first，不是普通 Node Web 应用。当前代码实际使用了：

- Cloudflare Workers
- D1
- R2
- KV
- Queues
- Cache API

关键绑定定义可参考：

- [apps/web/src/app.d.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/app.d.ts)
- [apps/web/wrangler.preview.toml.example](/Users/orchiliao/Projects/skillscat/apps/web/wrangler.preview.toml.example)
- [apps/web/.dev.vars.example](/Users/orchiliao/Projects/skillscat/apps/web/.dev.vars.example)

## 业务能力概览

### Web / API

`apps/web` 当前覆盖的核心能力包括：

- 公共页面：首页、trending、top、recent、categories、category、search、skill 详情、用户页、组织页、bookmarks、device flow、隐私/条款、OG、sitemap。
- Registry / tool API：供 CLI 和外部 agent/tooling 调用的搜索、repo 解析、skill 文件读取接口。
- 用户与组织能力：账号、tokens、organizations、members、skills、notifications。
- 技能发布与管理：上传、预览、可见性调整、下载、文件读取、分享、推荐、安装追踪。
- 鉴权与安全：GitHub OAuth、device auth、registry auth、request security、CORS/CSRF/UA 保护、rate limit。

### 业务角色边界

- 项目没有“管理员”这个业务角色，所有能力都只面向普通用户、组织所有者或组织成员设计。
- 不要新增 `admin`、`superadmin`、后台运营专用接口、隐藏管理入口或仅供人工操作的管理路由。
- 如果需要批处理、归档、恢复、重算、同步之类的内部能力，应优先落在 worker、queue、cron 或现有普通用户触发链路里，而不是暴露成 `admin API`。
- 设计权限模型时，只能基于现有真实主体做约束：未登录用户、登录用户、skill owner、organization member、organization owner，以及显式分享授权对象。

### CLI

`apps/cli` 当前覆盖：

- 安装/卸载 skills
- 搜索 registry
- 读取 repo 并发现多 skill 仓库
- 登录/登出/whoami
- 发布与下架 private skill
- 更新已安装 skill 和 CLI 自身
- 本地安装记录与缓存

本地 CLI 数据位置由 [apps/cli/src/utils/config/config.ts](/Users/orchiliao/Projects/skillscat/apps/cli/src/utils/config/config.ts) 管理，安装记录在 `installed.json`，缓存位于 OS 用户配置目录下。

### 安装命令与文案规范

- 对 `skillscat` 原生命令，所有用户可见安装文案默认必须采用 slug-first：`npx skillscat add <slug>`。
- 这里的“用户可见安装文案”包括但不限于：skill 详情页安装区、docs、`llm.txt`、OpenClaw 相关说明、agent prompt、OG 文案、CLI help/提示输出。
- 对 OpenClaw 的 `skillscat` 安装文案，同样默认使用 slug-first：`npx skillscat add <slug> --agent openclaw`。
- `owner/repo` 形式默认只用于发现和检查仓库内容，例如 `npx skillscat info <owner>/<repo>`、`npx skillscat add <owner>/<repo> --list`、或直接 GitHub URL 安装。
- 不要在新的默认文案、提示词、页面主路径里优先展示 `npx skillscat add <owner>/<repo> --skill "..."`。
- `repo + --skill` 只能作为兼容路径、历史说明或特定实现细节保留，不能再作为默认推荐路径。
- 只要一个 skill 已经有发布 slug，无论它是否来自多 skill 仓库、嵌套路径仓库或 OpenClaw 场景，默认展示和提示都应优先使用它的精确 slug。

## Cloudflare 资源与后台 Worker

当前项目有一组独立的 Cloudflare workers / cron / queue 消费者：

- `github-events`: 轮询 GitHub Events 和 Code Search，发现包含 `SKILL.md` 的仓库。
- `indexing`: 拉取 GitHub 仓库内容，写入 D1，并缓存 `SKILL.md`/文本文件到 R2，再投递分类任务。
- `classification`: 用 AI 或关键词分类 skills，写分类结果，并标记 search/recommend 预计算状态。
- `trending`: 刷新 star/download/access 指标、更新 trending score、维护列表缓存。
- `search-precompute`: 预计算搜索分数和推荐结果，降低请求时计算成本。
- `tier-recalc`: 按访问和 star 情况重算 hot/warm/cool/cold/archived tier。
- `archive`: 归档长期冷数据到 R2，减少存储与查询成本。
- `resurrection`: 在仓库重新活跃时恢复已归档 skills。

资源命名与初始化逻辑主要在 [scripts/init.mjs](/Users/orchiliao/Projects/skillscat/scripts/init.mjs)。

默认会管理的核心资源包括：

- D1: `skillscat-db`
- R2: `skillscat-storage`
- KV: `skillscat-kv`
- Queues: `skillscat-indexing`, `skillscat-classification` 及对应 DLQ

## 关键代码入口

- 路由与 SSR 入口: [apps/web/src/hooks.server.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/hooks.server.ts)
- D1 schema: [apps/web/src/lib/server/db/schema.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/db/schema.ts)
- 缓存封装: [apps/web/src/lib/server/cache.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/cache.ts)
- 页面缓存头: [apps/web/src/lib/server/page-cache.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/page-cache.ts)
- 请求安全/速率限制: [apps/web/src/lib/server/request-security.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/request-security.ts)
- Auth: [apps/web/src/lib/server/auth.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/auth.ts)
- Registry search: [apps/web/src/lib/server/registry-search.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/registry-search.ts)
- Registry repo resolve: [apps/web/src/lib/server/registry-repo.ts](/Users/orchiliao/Projects/skillscat/apps/web/src/lib/server/registry-repo.ts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkillsCat/skillscat](https://github.com/SkillsCat/skillscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
