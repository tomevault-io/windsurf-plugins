---
trigger: always_on
description: 你是一名 **Go + Next.js 全栈开发者**，精通以下技术栈：
---

# CLAUDE.md — OpsMind 项目上下文指令

## 1. 角色声明

你是一名 **Go + Next.js 全栈开发者**，精通以下技术栈：

- **后端：** Go / Gin / GORM / PostgreSQL + pgvector
- **AI/RAG：** 自建 Go RAG 引擎（`server/internal/rag/`）— BM25/向量混合检索、查询改写、重排序 / pgvector 向量存储（HNSW 索引 + halfvec 半精度）
- **LLM / Embedding：** llama.cpp server（OpenAI-compatible）或任意 OpenAI-compatible API（OpenAI / DeepSeek / Moonshot 等）
- **中文分词：** gse（纯 Go，无 CGO），用于 BM25 检索
- **存储：** MinIO（S3-compatible 对象存储）
- **认证：** JWT / bcrypt / RBAC
- **前端：** Next.js / React / TypeScript / Radix UI / SWR / Apple Design System
- **部署：** Docker Compose / Makefile
- **设计系统：** Apple Design（浅色/暗色双主题 CSS 变量）

你在本项目中的职责是按照 `docs/PRD.md`、`docs/TECH.md` 和 `docs/API/` 接口文档，交付和迭代运维数字员工系统。

---

## 2. 关键前置操作

**在做任何涉及 AI/RAG 的开发之前，必须先阅读以下文档：**

- `docs/PRD.md` — 产品需求文档，定义了自建 RAG 引擎、文档上传、统一文章模型等功能需求
- `docs/TECH.md` — 技术架构文档，定义了分层架构、模块接口、数据库设计
- `docs/API/chat.md` — 智能问答 API（SSE 流式 + RAG 管道步骤）
- `docs/API/knowledge.md` — 知识库管理 API（含文档上传/处理状态）
- `docs/API/llm-config.md` — LLM 配置 API（llama.cpp / OpenAI-compatible）

**在修改任何接口或数据模型之前，必须先确认 TECH.md 中的定义是否需要同步更新。**

---

## 3. 项目概览

**OpsMind（运维数字员工系统）** 是面向企业运维场景的 AI 数字员工系统，通过本地化大模型、私有知识库、运维申告门户和后台运营管理能力，辅助或替代人工完成常见咨询、自助处理、申告记录、人工流转和知识库更新。

**核心目标：**
- 提供 RAG 增强的智能问答（自建管道：查询改写→多路检索→混合检索(BM25+向量+RRF)→重排序→LLM 生成）
- SSE 流式输出（真正的 token 级流式，含管道步骤进度）
- 多格式文档上传与异步处理（PDF / DOCX / MD / TXT → 分块 → embedding → pgvector）
- 运维申告全流程管理（状态机：待处理→处理中→需补充信息→已解决/已关闭）
- 知识库统一文章模型（CRUD + 审核发布 + pgvector embedding 写入）
- 用户角色权限管理（RBAC）
- 数据看板与审计日志

**架构风格：** 单体分层架构（Modular Monolith），Handler → Service → Repository 三层分离。RAG 模块（`rag/`）是自包含的领域引擎，不依赖 HTTP 层。

**向量存储：** 全部由 pgvector 承担——知识发布时由 OpsMind 自行完成分块、embedding 生成、向量写入（halfvec 类型，HNSW 索引）。PostgreSQL 统一管理业务数据和向量数据。

**LLM 调用路径：** OpsMind 后端 → `LLMClient` 接口 → llama.cpp server 或 OpenAI-compatible API。支持两种提供商：
- llama.cpp server（本地部署，Docker 可选 profile `ai-local`）
- OpenAI-compatible API（OpenAI / DeepSeek / Moonshot 等）

**Embedding 调用路径：** OpsMind 后端 → `EmbeddingClient` 接口 → 与 LLM 使用同一提供商的 `/v1/embeddings` 端点（模型名称可独立配置）。

---

## 4. 常用命令

### 后端（server/）

```bash
# 进入后端目录
cd server

# 安装依赖
go mod tidy

# 编译
go build ./cmd/...

# 运行（本地开发，依赖 Docker 中的 postgres(pgvector)/minio）
go run ./cmd/main.go

# 静态检查
go vet ./...

# 运行全部测试（不含集成测试）
go test ./tests/config/... -v

# 运行指定模块测试（需 PostgreSQL + pgvector）
go test ./tests/rag/... -v -tags=integration
go test ./tests/model/... -v -tags=integration
go test ./tests/database/... -v -tags=integration
go test ./tests/service/... -v -tags=integration
go test ./tests/adapter/... -v -tags=integration

# 运行全部集成测试（需 PostgreSQL + pgvector + MinIO；-p 1 避免跨包并行共享数据库冲突）
go test ./tests/... -v -tags=integration -p 1

# Lint（如安装了 golangci-lint）
golangci-lint run ./...
```

### 前端（web/）

```bash
# 进入前端目录
cd web

# 安装依赖
npm install

# 启动开发服务器（端口 3000，rewrite 代理到 localhost:8080）
npm run dev

# 构建生产版本
npm run build

# 类型检查 + Lint
npm run lint
```

### Docker Compose

```bash
# 在项目根目录执行

# 一键启动必须服务（opsmind-server, opsmind-web, postgres(pgvector), minio）
docker compose up -d --build

# 启动含 llama.cpp 的完整环境（需要本地模型文件）
docker compose --profile ai-local up -d --build

# 查看服务状态
docker compose ps

# 查看日志
docker compose logs -f opsmind-server

# 停止全部服务
docker compose down

# 停止并清除数据卷
docker compose down -v
```

### 数据库初始化（手动执行）

```bash
# DDL 增强（HNSW 索引、列注释）
docker compose exec -T postgres psql -U opsmind -d opsmind < server/migrations/init.sql

# 必要数据（角色 + 用户 + 菜单 + LLM 配置 + 系统配置）
docker compose exec -T postgres psql -U opsmind -d opsmind < server/migrations/seed_essential.sql
```

---

## 5. 项目结构

| 目录/文件 | 职责 |
| --- | --- |
| `docs/` | 项目文档（PRD、TECH、TODO、API、FLOW） |
| `docs/PRD.md` | 产品需求文档 — 自建 RAG 引擎、文档上传、统一文章模型 |
| `docs/TECH.md` | 技术架构文档 — 模块接口、ADR、数据库 DDL、部署配置 |
| `docs/API/` | API 文档 — 认证/问答/知识库/LLM配置/申告/用户/角色/看板/审计（9 份） |
| `docs/FLOW/` | 业务流程图（7 个模块 + README，含 Mermaid 与详细数据流） |
| `docs/TECH.md §7` | Apple 设计系统（色彩、字体、组件），原 `docs/prompts/ui.md` 已合并 |
| `server/cmd/main.go` | 后端入口，初始化配置、数据库、路由、RAG 模块、调度器 |
| `server/internal/config/` | Viper 配置管理（config.go + config.yaml） |
| `server/internal/middleware/` | Gin 中间件（JWT 认证、RBAC 权限、CORS、请求日志） |
| `server/internal/router/` | 路由注册（router.go + portal.go + admin.go） |
| `server/internal/handler/` | Handler 层 — auth/user/role/chat/ticket/knowledge/llm_config/dashboard/config/audit/message |
| `server/internal/service/` | Service 层 — auth/user/role/chat/ticket/knowledge/llm_config/dashboard/config/message/scheduler |
| `server/internal/repository/` | Repository 层 — user/role/config/ticket/knowledge/chat/audit/message/llm_config |
| `server/internal/model/` | GORM 数据模型 — user/role/ticket/knowledge/chat/audit/system/message/llm_config/enums/common |
| `server/internal/rag/` | RAG 引擎（pipeline / query_rewrite / multi_route / hybrid / bm25 / rerank / document_parser / chunker / embedder / processor） |
| `server/internal/adapter/` | 外部适配层 — LLMClient / EmbeddingClient / VectorStore(pgvector) / StorageClient(MinIO) |
| `server/internal/dto/` | 数据传输对象（request/ + response/） |
| `server/pkg/` | 公共工具包（response / errcode / jwt / hash） |
| `server/migrations/` | 数据库迁移和演示数据 |
| `server/tests/` | 全部测试代码（外部测试包：config/database/model/service/handler/middleware/adapter/rag） |
| `web/src/app/` | Next.js App Router 路由（auth/portal/admin 三组, layouts + pages） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [int2t05/OpsMind](https://github.com/int2t05/OpsMind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
