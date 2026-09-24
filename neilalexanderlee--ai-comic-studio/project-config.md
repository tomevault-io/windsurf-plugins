---
trigger: always_on
description: > 本文件是面向 AI 助手（Codex）的项目级开发指南。
---

# AI漫剧工坊 — Codex 开发指南

> 本文件是面向 AI 助手（Codex）的项目级开发指南。
> 每次开始任务前请先读本文件，所有改动必须符合此处记录的约定。

---

## 沟通语言约定

**所有回复必须使用中文。** 无论用户用什么语言提问，Codex 的回复一律用中文。代码注释和代码本身保持原有风格（英文或中文均可），但解释、分析、提问、确认等文字部分全部用中文。

---

## 项目概述

AI漫剧工坊（英文 **AI Comic Studio**，仓库名 `ai-comic-studio`）是一个基于 AI 的漫剧/短剧分镜生成工具。用户可以：
1. 创建项目 → 编写剧情大纲和剧本
2. 将剧本解析为分镜版本（storyboard versions）
3. 为每个分镜生成首帧/尾帧（keyframe 模式）或参考帧（reference 模式）
4. 用帧驱动视频生成（Seedance / Kling / Jimeng / Veo 等）
5. 将视频合并为完整剧集

---

## 技术栈

| 层 | 技术 |
|---|---|
| 前端框架 | Next.js 15 (App Router, Turbopack) |
| 语言 | TypeScript (strict) |
| 样式 | Tailwind CSS |
| 数据库 | SQLite via better-sqlite3 |
| ORM | Drizzle ORM |
| 状态管理 | Zustand |
| AI SDK | `ai` (Vercel AI SDK), `@ai-sdk/openai`, `@ai-sdk/google` |
| i18n | next-intl |
| 测试 | Vitest |

---

## 目录结构

```
ai-comic-studio/   # 本地目录建议名；历史亦可能为 AIComicBuilder
├── AGENTS.md                  # ← 本文件
├── docs/
│   ├── ARCHITECTURE.md        # 系统架构详解
│   ├── EVAL.md                # Eval 框架说明
│   └── APIs/                  # 火山方舟/Kling 官方 API PDF 文档
├── drizzle/                   # SQL migration 文件
│   └── meta/_journal.json     # Migration 注册表（必须与 .sql 文件同步更新）
├── src/
│   ├── app/
│   │   ├── api/               # Next.js Route Handlers (Server)
│   │   │   └── projects/[id]/
│   │   │       ├── generate/route.ts   # 核心生成入口（SSE 流式输出）
│   │   │       ├── import/split/       # 导入时自动分集
│   │   │       ├── episodes/           # 剧集 CRUD
│   │   │       └── ...
│   │   └── [locale]/          # 国际化页面
│   ├── lib/
│   │   ├── ai/
│   │   │   ├── types.ts               # AIProvider / VideoProvider 接口
│   │   │   ├── provider-factory.ts    # 按 protocol 字符串创建 provider 实例
│   │   │   ├── character-router.ts    # 角色图片路由（智能状态选择）
│   │   │   ├── prompt-enhancer.ts     # 模型感知 prompt 增强
│   │   │   ├── prompts/               # Prompt 构建函数 + 注册表
│   │   │   │   ├── outline-expand.ts  # AI 自动生成（大纲→S级剧本）
│   │   │   │   ├── frame-strategy-judge.ts  # 帧生成策略 LLM judge prompt
│   │   │   │   └── ...
│   │   │   └── providers/             # 各模型实现（openai/gemini/kling/seedance/jimeng/...）
│   │   ├── db/
│   │   │   ├── schema.ts              # Drizzle 表定义（单一事实来源）
│   │   │   └── index.ts               # DB 实例 + idempotent migration runner
│   │   ├── storyboard/                # 分镜工具函数
│   │   │   ├── frame-generation-strategy.ts  # 智能帧生成策略（三层决策）
│   │   │   ├── detect-structured-storyboard.ts
│   │   │   ├── extract-shot-script.ts
│   │   │   └── complete-extracted-shots.ts
│   │   └── bootstrap.ts              # 启动序列（migrations → providers → worker）
│   ├── stores/                # Zustand 客户端状态
│   │   ├── project-store.ts
│   │   ├── episode-store.ts
│   │   └── model-store.ts
│   └── __tests__/
│       ├── setup.ts
│       ├── unit/              # 纯函数单元测试
│       └── integration/       # API 集成测试
└── src/lib/evals/             # AI Eval 评估框架
```

---

## AI Provider 系统

### Protocol 字符串

每个 AI 能力由一个 `protocol` 字符串标识。这是整个 provider 系统的 key。

| Protocol | 模型 | 用途 |
|---|---|---|
| `openai` | GPT-4o / DALL-E / 兼容接口 | 文本 + 图片 |
| `gemini` | Gemini / Veo | 文本 + 图片 + 视频 |
| `doubao` | Seedream（火山方舟 ARK API） | 图片 |
| `jimeng` | 即梦 AI | 图片 |
| `jimeng-video` | 即梦 AI | 视频 |
| `kling` | 可灵 | 图片 + 视频 |
| `seedance` | Seedance（火山方舟 ARK API） | 视频 |

**规则**：`provider-factory.ts` 的 switch-case 是增加新 provider 的唯一入口。`prompt-enhancer.ts` 必须为新 protocol 同步添加对应的 system prompt。

### 三种 Provider

```typescript
AIProvider       // generateText + generateImage
VideoProvider    // generateVideo
```

`resolveAIProvider` / `resolveImageProvider` / `resolveVideoProvider`（在 `provider-factory.ts`）从请求体中的 `modelConfig` 读取配置，用户未配置时 fallback 到 `getAIProvider()` 全局默认。

---

## 数据库规范

### 迁移流程（必须严格遵守）

1. 在 `src/lib/db/schema.ts` 添加字段
2. 在 `drizzle/` 创建 `NNNN_<描述>.sql`（仅写增量 DDL）
3. 在 `drizzle/meta/_journal.json` 添加对应条目（idx 递增，when 递增）
4. 不需要手动运行迁移 — `bootstrap()` 在启动时自动执行

**Boolean 列**：统一用 `integer("col_name").notNull().default(0)`（0/1），不用 SQLite 的 BOOLEAN。

**当前最新迁移索引**：`idx 54` — `0054_shot_anchor_first_continuity_mode`

### 关键表

| 表 | 说明 |
|---|---|
| `projects` | 顶层实体，含 `visualStyle`、`generationMode`（legacy）、`enhancePrompts`、`linkShotsViaCutPoint`、`useProjectPrompts` |
| `episodes` | 分属 project 的剧集，含 `generationMode`（可覆盖 project 级） |
| `storyboard_versions` | 分镜版本，每个版本对应一批 shots |
| `shots` | 单个分镜；帧字段：`anchorFirst`、`anchorLastAi`、`cutPoint`、`chainSourceShotId`、`chainSourceType`（语义见 `docs/ARCHITECTURE-FRAMES.md` §0「Plan B」） |
| `characters` | 项目/剧集角色，含 `visualHint`、`voiceHint` |
| `character_assets` | 角色图片（morph/blueprint 类型，带 tag 状态标签） |
| `episode_characters` | 多对多：角色参与哪些剧集 |

---

## 核心约定（必须遵守）

### 1. filterShotCharacters — 绝不 fallback 到全量角色

```typescript
// ✅ 正确：无匹配时传空列表
const charsForFrame = filterShotCharacters(shotText, projectCharacters);

// ❌ 错误：无匹配时 fallback 到所有角色（群演场景会注入无关角色图）
const charsForFrame = shotCharacters.length > 0 ? shotCharacters : projectCharacters;
```

`filterShotCharacters` 在 shot 文本里找不到角色名时返回 `[]`，这是正确行为。调用方不得覆盖这个结果。

### 2. episodeId — 始终优先用 URL 参数

```typescript
// ✅ 正确
const episodeId = urlEpisodeId || useProjectStore.getState().currentEpisodeId;

// ❌ 错误：Zustand store 在首次渲染时可能未水合（null）
const episodeId = useProjectStore.getState().currentEpisodeId;
```

Zustand store 的 `currentEpisodeId` 在客户端水合前是 `null`。任何需要 episodeId 的操作（版本创建/删除、分镜生成）必须先读 `useParams()` 里的 `urlEpisodeId`。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neilalexanderlee/ai-comic-studio](https://github.com/neilalexanderlee/ai-comic-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
