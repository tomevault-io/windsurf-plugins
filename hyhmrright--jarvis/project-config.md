---
trigger: always_on
description: This file provides guidance to Codex when working in this repository.
---

# AGENTS.md — AI Agent Instructions / AI 编程助手指南

This file provides guidance to Codex when working in this repository.
本文件为 Codex 在此代码库中工作时提供指导。

## Branch Strategy / 分支策略

- **main**: Release only. Never commit or develop directly here. Only accepts merges from dev or other development branches.
- **dev**: Primary development branch (GitHub default). All daily development, bugfixes, and feature work go here or on sub-branches.
- After development is complete: dev → merge → main → push. No steps may be skipped.

- **main**：仅用于发版，不得直接提交或开发。只接受来自 dev 等开发分支的 merge。
- **dev**：主开发分支（GitHub 默认分支），所有日常开发、bugfix、功能开发均在此分支或其子分支进行。
- 开发完成后：dev → merge → main → push，不得跳过。

### Branch Naming / 协作分支命名规范

All feature branches are created from `dev`, named `<type>/<short-description>`:
所有功能分支从 `dev` 创建，命名格式 `<类型>/<简短描述>`：

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New features | `feature/rag-agent-integration` |
| `fix/` | Bug fixes | `fix/sse-disconnect` |
| `docs/` | Documentation only | `docs/api-reference` |
| `infra/` | Docker, CI, deployment | `infra/add-healthcheck` |

### Commit Message Format / Commit 消息规范

Follow [Conventional Commits](https://www.conventionalcommits.org/): `<type>: <description>`

遵循 [Conventional Commits](https://www.conventionalcommits.org/)：`<type>: <description>`

Types / 类型：`feat`、`fix`、`docs`、`style`、`refactor`、`test`、`chore`、`ci`

## Worktree Parallel Development / Worktree 并行开发

### When to Use Worktrees / 何时使用 Worktree

- Developing multiple features simultaneously without interference
- Reviewing a PR without disrupting current work
- Emergency fix while the current branch has unfinished work

- 需要同时开发多个功能且互不干扰
- Review 他人 PR 时不想影响当前工作
- 紧急修复但当前分支有未完成的功能

### Creating and Managing Worktrees / 创建与管理

```bash
# Create worktree (branch off dev)
git worktree add .worktrees/<name> -b feature/<name> dev

# Initialize environment in worktree
cd .worktrees/<name>
cp ../../.env .                          # Copy environment variables
cd backend && uv sync && cd ..           # Install backend deps
cd frontend && bun install && cd ..      # Install frontend deps
cd backend && uv run alembic upgrade head && cd ..  # DB migration

# List all worktrees
git worktree list

# Remove worktree (after merge)
git worktree remove .worktrees/<name>

# Prune deleted worktree references
git worktree prune
```

### Using Worktrees in Codex / Codex 中使用 Worktree

```bash
# Launch isolated Codex session
Codex --worktree feature-xxx

# Or request in conversation
> "在 worktree 中开发这个功能"
```

### Port Assignments / 端口分配

Docker base services (postgres/redis/qdrant/minio) are shared across all worktrees. Dev servers need different ports.
Docker 基础服务（postgres/redis/qdrant/minio）所有 worktree 共享。开发服务器需分配不同端口：

| Working Directory | Backend Port | Frontend Port |
|-------------------|-------------|---------------|
| Main (root) | 8000 | 3000 |
| Worktree 1 | 8001 | 3100 |
| Worktree 2 | 8002 | 3200 |

```bash
# Specify ports when starting in a worktree
uv run uvicorn app.main:app --reload --port 8001
bun run dev --port 3100
```

### Notes / 注意事项

- `.env` is not tracked by git; copy it manually when creating a new worktree.
- Avoid modifying `alembic/versions/` in multiple worktrees simultaneously (migration conflicts).
- `.worktrees/` is in `.gitignore` and will not be accidentally committed.
- The same branch cannot be checked out by two worktrees at the same time.

- `.env` 文件不在 git 中，新建 worktree 需手动复制。
- 避免多个 worktree 同时修改 `alembic/versions/`（数据库迁移冲突）。
- `.worktrees/` 已在 `.gitignore` 中，不会被意外提交。
- 同一分支不能被两个 worktree 同时检出。

## Project Overview / 项目概述

JARVIS is an AI assistant platform with RAG knowledge base, multi-LLM support, and streaming conversations, using a monorepo structure.

**Completed features (Phase 1-6)**: Multi-channel messaging (Slack/Discord/Telegram/Feishu/WhatsApp/Webhook), sandboxed tool execution, LLM failover, RAG knowledge base, dynamic skills (SKILL.md), personal API keys, live Canvas, Voice (TTS/STT), multilingual UI, monitoring stack (Grafana/Loki/Prometheus), Traefik gateway.

JARVIS 是具备 RAG 知识库、多 LLM 支持、流式对话的 AI 助手平台，采用 monorepo 结构。

**已完成功能（Phase 1-6）**：多渠道消息（Slack/Discord/Telegram/Feishu/WhatsApp/Webhook）、工具沙箱执行、LLM 故障切换、RAG 知识库、动态技能（SKILL.md）、个人 API Key、实时 Canvas、语音（TTS/STT）、多语言 UI、监控栈（Grafana/Loki/Prometheus）、Traefik 网关。

## Core Architecture / 核心架构

```
JARVIS/
├── backend/           # FastAPI backend (Python 3.13 + uv)
│   ├── app/
│   │   ├── main.py    # FastAPI entry point, lifespan manages infra connections
│   │   ├── agent/     # LangGraph ReAct + expert agents (graph/llm/state/persona)
│   │   ├── api/       # HTTP routes (auth/chat/conversations/documents/settings/logs/usage/admin/keys)
│   │   ├── channels/  # Messaging channels (Slack/Discord/Telegram/Feishu/WhatsApp/Webhook)
│   │   ├── core/      # Config (Pydantic Settings), security (JWT/bcrypt/Fernet), rate limiting, logging (structlog), logging middleware
│   │   ├── db/        # SQLAlchemy async models and sessions
│   │   ├── gateway/   # Channel router + session manager
│   │   ├── infra/     # Infrastructure client singletons (Qdrant/MinIO/Redis/Ollama)
│   │   ├── plugins/   # Plugin loader + SKILL.md parser/registry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyhmrright/JARVIS](https://github.com/hyhmrright/JARVIS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
