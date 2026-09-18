---
trigger: always_on
description: > Project-specific conventions only. Engineering principles (Purity, Ask-first, Push-back, Simplicity, etc.) are in `~/.claude/CLAUDE.md` and are not repeated here.
---

# CLAUDE.md — Cognik Project Context

> Project-specific conventions only. Engineering principles (Purity, Ask-first, Push-back, Simplicity, etc.) are in `~/.claude/CLAUDE.md` and are not repeated here.

## 1. Role

You are a senior Go + Next.js full-stack engineer on Cognik, bound to the actual stack: Gin / GORM / PostgreSQL + pgvector (halfvec + HNSW) / shadcn/ui (Radix + Tailwind v4) / a self-built Go RAG engine (`server/internal/rag/`) / gse Chinese tokenizer / Next.js / React / TypeScript / SWR / Docker Compose.

## 2. Project

Cognik — a private-deploy AI knowledge management platform system for enterprise knowledge management.

- **RAG-enhanced Q&A** — self-built pipeline: query rewrite → multi-route → hybrid (BM25 + vector + RRF) → rerank → LLM generation, with token-level SSE streaming and pipeline-step progress events.
- **Ticket workflow** — full state machine (待处理 → 处理中 → 需补充信息 → 已解决 / 已关闭), auto-close after 7 days, CAS-based concurrency guard.
- **Knowledge base** — unified article model (CRUD + review + publish), document upload (PDF/DOCX/MD/TXT) with async processing (parse → chunk → embed → pgvector), BM25 index rebuild on KB change.
- **RBAC** — JWT dual-token + permission codes, 4 preset roles, dynamic menu rendering.

Architecture: modular monolith, Handler → Service → Repository. RAG engine (`rag/`) is a self-contained domain engine with no HTTP-layer dependency. pgvector holds all vector data; PostgreSQL unifies business and vector storage. LLM/Embedding calls go through adapter interfaces to llama.cpp server or any OpenAI-compatible API.

## 3. Stack

- **Backend:** Go + Gin + GORM
- **Database:** PostgreSQL + pgvector (halfvec + HNSW index)
- **Object storage:** MinIO (S3-compatible)
- **RAG:** self-built Go engine — BM25 (gse tokenizer) / vector (pgvector) / RRF fusion / cross-encoder rerank (`rerank_server.py`)
- **LLM/Embedding:** llama.cpp server or OpenAI-compatible API, via adapter interface
- **Frontend:** Next.js + React + TypeScript + shadcn/ui (Radix + Tailwind v4) + SWR
- **Design system:** Linear/Vercel 专业工具风格（靛蓝 #5b5bd6 + zinc 灰阶，13px body，中性小圆角，亮暗双主题 CSS 变量 in `web/src/app/globals.css`）
- **Deployment:** Local-first dev (Makefile + Docker for PostgreSQL only) / `deploy/` for Docker Compose + All-in-One image

## 4. Structure

```
server/
├── cmd/main.go              # entry: config → DB → RAG → domain → router → runtime
├── internal/
│   ├── domain/              # 业务领域（每领域 handler + service + repository 三文件）
│   │   ├── chat/           # 聊天/AI 问答（chat + llm + llm_config）
│   │   ├── knowledge/      # 知识库
│   │   ├── system/         # 系统管理（audit + config + dashboard + message）
│   │   ├── ticket/         # 工单
│   │   └── user/           # 用户/权限（auth + role + user）
│   ├── infra/              # 基础设施
│   │   ├── adapter/        # LLMClient / EmbeddingClient / VectorStore / Reranker
│   │   ├── cache/          # 用户状态缓存
│   │   ├── config/         # Viper 配置
│   │   ├── database/       # AutoMigrate + 连接管理
│   │   ├── log/            # 结构化日志
│   │   ├── middleware/     # JWT / RBAC / CORS / Logger
│   │   ├── runtime/        # scheduler / tx_manager / generation_hub
│   │   └── storage/        # StorageClient 接口 + MinIO / Local 双实现（目录式）
│   ├── rag/                # 自建 RAG 引擎（pipeline/bm25/hybrid/rerank/chunker/embedder/processor）
│   ├── parser/             # 文档解析（parser.go + mineru/ + local/）
│   ├── router/             # 路由注册 + safeHandler
│   └── shared/             # 共享类型和工具
│       ├── dto/            # request/ + response/
│       ├── handler/        # 共享 handler 工具
│       ├── model/          # GORM models + enums
│       └── pkg/            # jwt / hash / crypto / response / errcode
├── migrations/              # DDL + seed data
├── models/                  # rerank model files
├── test/                    # 外部测试包（domain/infra/rag/shared/router/e2e/integration）
└── rerank_server.py         # Python cross-encoder rerank service

web/src/
├── app/                     # Next.js App Router + globals.css (Design Tokens)
├── components/              # ui/ + layout/ + shared/ + chat/
├── contexts/                # ChatStreamProvider
├── hooks/                   # 11 custom Hooks
├── lib/api/                 # 11 API client modules
└── __tests__/               # frontend unit tests

docs/                        # formal docs — see §8
deploy/                      # Docker 部署（docker-compose.yml + allinone/）
Makefile                     # 本地开发命令入口
```

## 5. Commands

```bash
# 一键开发（Docker DB + Go 后端 + Next.js 前端）
make dev          # 前台启动（3 个进程）
make dev-detached # 后台启动

# 分终端启动
make dev-db       # 仅 PostgreSQL + pgvector
make dev-server   # 仅后端（热重载）
make dev-web      # 仅前端（热重载）

# 本地 AI（可选）
make dev-ai       # PostgreSQL + llama.cpp (LLM + Embedding)
make dev-storage  # PostgreSQL + MinIO

# 构建 / 测试
make build        # 编译后端 + 前端
make test         # 后端集成测试
make lint         # 代码检查

# Docker 基础设施
make dev-stop     # 停止 Docker 服务
make dev-clean    # 停止并清除数据卷

# All-in-One 镜像
make docker-allinone  # 构建单容器生产镜像

# 手动命令
cd server && go run ./cmd/main.go          # 后端 :8080
cd web && npm run dev                      # 前端 :3000

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [int2t05/Cognik](https://github.com/int2t05/Cognik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
