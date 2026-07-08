---
trigger: always_on
description: 本文件为 AI 编程助手（Codex / Claude Code 等）提供即页（jpage）项目的工作指南。读者应假设对项目一无所知；所有改动请以此文件为上下文基准。
---

# AGENTS.md

本文件为 AI 编程助手（Codex / Claude Code 等）提供即页（jpage）项目的工作指南。读者应假设对项目一无所知；所有改动请以此文件为上下文基准。

---

## 项目概述

**即页（jpage）** 是一个零配置的 HTML / Markdown 即时预览与分享工具。用户上传 `.html`、`.md`/`.markdown` 或 `.zip` 文件后，即可得到在线渲染页面与短链接（`/s/:key`），无需额外部署流程。项目同时面向 AI 工作流：内置 MCP Streamable HTTP 端点与 CLI 工具，支持通过 API Token 自动上传、管理文件。

核心能力：

- 即时预览 / 源码双模式
- Markdown 增强渲染（代码高亮、KaTeX 公式、Mermaid 图表）
- 文件版本历史与回滚
- 标签、分类、收藏
- 多用户 + admin / user 角色体系
- API Token（`jp_` 前缀）与全局 `MCP_TOKEN` 双认证
- 内容模板市场（用户上架 → 管理员审核 → 公开使用）
- Skills 注册与打包下载

---

## 技术栈

- **运行时**：Node.js ≥ 20
- **后端框架**：Express 4 + express-session（SQLite 会话存储）+ express-rate-limit
- **数据库**：SQLite3（`sqlite3`），启动自动迁移，WAL 模式
- **前端**：原生 ES Modules，无框架；`public/js/` 为源码，`public/dist/` 为 esbuild 生产产物
- **Markdown 渲染**：marked + marked-highlight + highlight.js + KaTeX + Mermaid
- **安全**：helmet（关闭内置 CSP，手写分级策略）、bcryptjs、CSP nonce
- **MCP**：`@modelcontextprotocol/sdk` Streamable HTTP
- **ZIP / 打包**：archiver、jszip、multer
- **邮件**：nodemailer（可选，用于邮箱验证）
- **测试**：Node 内置 `node:test` + supertest
- **Lint**：ESLint flat config（`eslint.config.mjs`）
- **容器**：Docker / Docker Compose，多阶段构建

---

## 目录与模块划分

```
server.js                 # 入口：app 装配、中间件、路由挂载、MCP/静态/catch-all、启动编排
logger.js                 # 结构化 JSON Lines 日志（info/warn/error/audit）
mailer.js                 # SMTP 初始化与邮件发送
mcp-server.js             # MCP 入口，re-export mcp/transport.js
migrations.js             # Migration runner + dbRun/dbGet/dbAll Promise 封装
skills-registry.js        # 扫描 skills/*/SKILL.md，提供列表/详情/ZIP 打包
build.js                  # esbuild 前端构建脚本

lib/                      # 共享层
  db.js                   # SQLite 连接与 PRAGMA 配置
  paths.js                # DATA_DIR / UPLOAD_DIR 常量
  util.js                 # now()、shareKey、clientIp、decodeFilename 等
  csp.js                  # 分级 CSP 策略与 nonce
  auth-state.js           # adminUserId 共享状态
  templates.js            # 样式模板加载 + marked/KaTeX 渲染管线
  render.js               # 文件 → HTML 渲染
  render-cache.js         # 渲染结果 LRU 缓存
  fts.js                  # FTS5 全文索引
  categories.js           # 分类名内存缓存
  view-counts.js          # 浏览数缓冲批量回写
  zip.js                  # ZIP 上传校验/解压/分类
  dispatch.js             # MCP 进程内请求分发器
  crypto.js               # API Token 明文 AES-256-GCM 加密
  usage.js                # 用户存储空间维护（增删改重算）
  middleware/
    auth.js               # requireAuth / loadSession / requireAdmin
    files.js              # loadFileWithPrivacy / checkFileOwnership
    usage.js              # /api 请求用量采集（API 调用计数 + 来源）

routes/                   # 按域拆分的 Express Router
  auth.js                 # /api/auth/*
  users.js                # /api/users/*（admin）
  tokens.js               # /api/tokens/*
  tags.js                 # /api/tags/*
  categories.js           # /api/categories、/api/templates
  admin.js                # /api/admin/*（导出/导入/统计/自动备份）
  skills.js               # /api/skills/*、/api/mcp/config
  content-templates.js    # /api/content-templates/* 与 /api/content-templates/market/*
  files/                  # /api/files/* 子路由
    index.js              # 路由装配
    crud.js               # 单文件元数据 / 更新 / 删除 / 批量
    list.js               # 列表与搜索
    upload.js             # multipart / JSON / ZIP-base64 上传
    overwrite.js          # 覆盖上传（multipart / JSON）
    versions.js           # 版本历史
    share.js              # 短链设置（别名/过期/密码）
    detail-serve.js       # content / render / download / asset
    associations.js       # tags / star / category
    _shared.js            # multer、限流、版本备份、下载头等共享常量

mcp/                      # MCP 实现
  transport.js            # /mcp 路由挂载、session 生命周期
  server.js               # McpServer 工厂，注册 17 tools + 2 resources
  tools-files.js          # 文件相关 tool
  tools-versions.js       # 版本相关 tool
  tools-tags.js           # 标签 tool
  tools-categories.js     # 分类 tool
  tools-content-templates.js # 内容模板 tool
  resources.js            # jpage://files / jpage://file/{id}
  util.js / constants.js  # 共享

bin/                      # CLI 工具（npm 包入口 `jpage`）
  jpage.js                # CLI 入口与命令分发
  args.js / config.js / client.js
  commands/               # upload、ls、cat、url、mv、rm、star、tags、skills、whoami、update

public/                   # 前端静态资源
  index.html              # SPA 模板，含 landing / home / preview / market 等模板块
  css/style.css           # 源样式（开发模式直接加载）
  js/                     # 前端 ESM 源码
    app.js                # hash 路由 + 动态 import 入口
    api.js / theme.js / utils.js
    pages/                # landing、login、home、preview、market、share-settings
    components/           # dialog、toast、users-modal、tokens-modal
  dist/                   # esbuild 产物（gitignore）

templates/                # Markdown 渲染样式模板（default/github/academic/dark-pro）
skills/                   # Claude Code / Desktop Skill 包
migrations/               # 001-021 按序执行的 schema 迁移
test/                     # 单元 + 集成测试 + e2e/性能 harness
data/                     # SQLite DB、session、上传文件（gitignore，运行期自动创建）
```

---

## 构建与运行命令

**推荐部署（Docker Compose）**：

```bash
cp .env.example .env          # 编辑 ADMIN_PASSWORD、SESSION_SECRET 等
docker-compose up -d
docker-compose logs -f
docker-compose down
```

**本地开发**：

```bash
npm install
npm run dev                   # nodemon 热重载；开发无需构建
npm start                     # 直接运行

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code2rich/jpage](https://github.com/code2rich/jpage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
