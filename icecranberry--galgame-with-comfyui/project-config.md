---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 项目概述

本地 AI 图像生成智能体。用户与可自定义人格的 AI 角色对话，AI 根据对话上下文判断是否需要生成图像，通过 ComfyUI (SDXL/FLUX) 出图后推送到前端。

## 技术栈

| 组件 | 技术 | 端口 |
|------|------|------|
| 前端 | Vue 3 + Pinia + Vue Router + Vite + vue-easy-lightbox | 5173 |
| 主控后端 | Node.js + Express (ESM) | 3099 |
| 向量服务 | Python FastAPI + ChromaDB + ONNX Runtime + 爬虫 | 8765 |
| LLM | DeepSeek API (兼容 OpenAI SDK，透传) | - |
| 生图引擎 | ComfyUI (WebSocket + HTTP) | 8188 |
| 数据库 | SQLite (better-sqlite3) + FTS5 | - |
| 嵌入模型 | Jina v2 base zh (768d, ONNX, 均值池化 + L2 归一) | - |

## 常用命令

```bash
# 一键启动全部开发服务（清理端口 → 检查环境 → 启动三项服务 → 自动打开浏览器）
npm run dev          # 项目根目录

# 分别启动
cd agent-core && npm run dev          # Express + nodemon --watch
cd web-ui && npm run dev              # Vite HMR
cd vector-service && ./venv/Scripts/python.exe -m uvicorn server:app --host 0.0.0.0 --port 8765

# 停止所有开发服务（优雅退出 → 等待 → taskkill）
npm run stop

# 生产部署（PM2）
pm2 start ecosystem.config.cjs

# 测试 ComfyUI 连接
curl http://localhost:3099/api/images/comfyui-health
```

## 项目结构

```
project-root/
├─ agent-core/              # 主控后端 (Express, :3099)
│  ├─ app.js                # 入口：中间件、路由挂载、WAL 定期 checkpoint、优雅退出
│  ├─ data/                 # 运行时数据（DB、图片、头像，gitignore）
│  ├─ public/               # web-ui build 产物（gitignore）
│  └─ src/
│     ├─ config.js          # 配置中心（dotenv + DB 持久化 + .env 写回，三通道同步）
│     ├─ db/index.js        # SQLite 表/FTS5/触发器/索引/种子/迁移/repairFtsIndex
│     ├─ llm/deepseek.js    # OpenAI 兼容客户端（chatSync + chatStream，含重试/超时/截断）
│     ├─ middleware/errorHandler.js
│     ├─ routes/
│     │  ├─ chat.js         # SSE 流式对话 + 三种生图触发 + 后处理（情绪/记忆/画像/好感度）
│     │  ├─ images.js       # 生图 API（tasks CRUD、直接生成、测试画风、健康检查）
│     │  ├─ characters.js   # 角色 CRUD + AI 生成（含联网搜索）+ 头像上传/AI 生成 + 送礼
│     │  ├─ config.js       # 配置读写（ComfyUI/LLM/用户/规则/画师收藏/功能开关）
│     │  ├─ moments.js      # 朋友圈（帖子生成含 SSE 推送、评论、点赞、AI 自动回复）
│     │  ├─ memory.js       # 记忆检索、碎片查询、情绪历史
│     │  ├─ relationships.js      # 角色间关系 CRUD（有向图）
│     │  ├─ userRelationships.js  # 用户→角色关系 CRUD
│     │  ├─ portraits.js          # 用户画像 CRUD（角色视角）
│     │  └─ notifications.js      # 主动聊天 SSE 推送 + 未读红点 + 强制触发
│     └─ services/
│        ├─ emotionEngine.js         # VAD 三维情绪引擎（双层衰减 + 规则表/LLM + 好感度变化）
│        ├─ memoryExtractor.js       # 异步记忆碎片提取（DeepSeek → 向量化 → ChromaDB + SQLite）
│        ├─ memorySearch.js          # 三路召回 + RRF 融合（keyword + vector + entity）
│        ├─ summarizer.js            # 滚动摘要（每 50 条消息触发一次）
│        ├─ vectorClient.js          # Python 向量服务 HTTP 客户端（embed/search/upsert/delete）
│        ├─ imageSkill.js            # 生图调度（提示词优化 → 注入 workflow → 提交 ComfyUI → 兜底文件夹）
│        ├─ comfyClient.js           # ComfyUI GUI→API 转换 + WebSocket 进度 + 轮询兜底
│        ├─ momentScheduler.js       # 朋友圈定时调度（每 10 分钟扫描，排队发帖）
│        ├─ proactiveChatScheduler.js # 主动聊天三线调度器（VAD + 频率 + 启动，含动机分档/配图）
│        ├─ portraitExtractor.js     # 用户画像异步提取（每 10 条触发，向量去重）
│        ├─ notificationBus.js       # 通知总线（SSE 广播，services↔routes 解耦）
│        ├─ webSearch.js             # 联网搜索（萌娘百科→Bing 降级，含 LLM 关键词提取）
│        └─ seeds.js                 # 默认角色种子数据
├─ web-ui/                  # Vue 3 前端 (Vite HMR, :5173)
│  ├─ vite.config.js        # Vite 配置（含 SSE 代理 timeout:0 防断开）
│  └─ src/
│     ├─ main.js            # 入口：hash mode 6 路由（/chat/:id, /moments, /gallery, /tavern, /settings）
│     ├─ userConfig.js      # 用户自有配置（头像/昵称/自画像，独立于角色）
│     ├─ stores/
│     │  ├─ chat.js         # 核心 store（角色/消息/SSE 消费/重试/主动消息处理/好感度实时更新）
│     │  ├─ moments.js      # 朋友圈 store（帖子分页/SSE 监听/评论/点赞）
│     │  ├─ settings.js     # 设置 store
│     │  └─ notifications.js # 主动聊天通知 store（未读红点/SSE 监听）
│     ├─ api/index.js       # 后端 API 封装（含 SSE ReadableStream 解析 + 多通道 SSE 连接）
│     ├─ views/             # 5 个视图页面
│     └─ components/        # 8 个通用组件（NavBar/Sidebar/Gallery/AvatarCropper 等）
├─ vector-service/          # 向量服务 (Python FastAPI, :8765)
│  ├─ server.py             # /embed /search /upsert /delete /health /scrape
│  ├─ embedding.py          # ONNX 推理（Jina v2, mean pooling + L2 normalize）
│  ├─ chroma_store.py       # ChromaDB 持久化（cosine 空间）
│  └─ download_model.py     # 模型下载脚本（~155MB, hf-mirror.com）
├─ workflow/                # ComfyUI workflow 模板
├─ scripts/
│  ├─ dev.mjs               # 一键 dev 启动（端口清理含进程身份验证→环境检查→模型下载→三进程+浏览器）
│  └─ stop.mjs              # 一键停止所有 dev 服务
├─ ecosystem.config.cjs     # PM2 生产配置（agent-core + vector-svc）
└─ CLAUDE.md
```

## 核心架构决策

### 消息双表设计

- **`raw_messages`**: 完整 LLM 对话原文（包括 `{"prompt":"..."}` JSON 格式），给 LLM 构建上下文用。每轮 user + assistant 各一条。
- **`messages`**: 分句展示气泡，按句子拆分存储，每个气泡一条。带 `images`/`is_proactive` 列。前端通过 `seq` 排序。
- 这样把"LLM 需要的完整上下文"和"前端需要的分句展示"解耦。

### 人格引擎三层叠加

```
固定人格 (Base Prompt) → 动态情绪 (VAD Emotion Engine) → 动态记忆 (RRF RAG Recall)
```

情绪引擎细节：双层 VAD 模型 — `mood` (decay=0.98, 长期底色) + `instant` (decay=0.85, 即时反应)，综合情绪 = mood×0.4 + instant×0.6。刺激评估优先走规则表（高频场景），未命中才调 DeepSeek 兜底。每次对话后计算情绪变化并写入 `emotion_snapshots`（每 conversation 仅保留最新一条，UNIQUE 约束）。

### 好感度系统 (Affinity)

每次对话后情绪引擎计算 `affinity_delta`（-5~+5），更新 `user_relationships.affinity`（0~100，初始 50）。

- **回归衰减**: 连续 24h 未互动，自动衰减 -1（下限 0）。`last_interaction_at` 记录最近互动时间。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icecranberry/galgame-with-comfyUI](https://github.com/icecranberry/galgame-with-comfyUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
