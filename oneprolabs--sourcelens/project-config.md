---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with
project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial
tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes,
simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```text
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it
work") require constant clarification.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer
rewrites due to overcomplication, and clarifying questions come before
implementation rather than after mistakes.

## 项目概述

SourceLens 是一个保留基础架构的新项目，包含 Django REST API 后端和 Vue 3 前端。当前核心能力集中在用户认证、权限/角色管理、agentcore 管理台集成、任务调度基础设施和通知/LLM 管理能力。

核心开发目录：
- `backend/` — Django REST API、accounts、core 配置和 agentcore 集成
- `frontend/` — Vue 3 + Vite 前端和管理台页面

## 常用命令

### Docker 本地开发
```bash
cp env.sample .env.dev
docker-compose -f docker-compose.dev.yml up -d
# Web: http://localhost:8000
# API docs: http://localhost:8000/swagger/
# Celery monitor (Flower): http://localhost:5555
```

### 开发调试基本原则（容器内热加载）

本项目基于 **Django + Celery**，开发全程在容器内完成，依赖
`docker-compose.dev.yml`。代码以 volume 挂载方式进入容器
（`./backend:/opt/backend`），因此热加载行为按服务区分：

| 服务 | 容器名 | 代码改动后 | 说明 |
|---|---|---|---|
| `backend-api` | `sourcelens-api-dev` | **自动重载，无需重启** | 改动 Python 代码后 API 自动重启加载 |
| `backend-worker` | `sourcelens-worker-dev` | **必须手动重启** | Celery worker 不会热加载，任务代码改动后须重启 |
| `backend-scheduler` | `sourcelens-scheduler-dev` | 同 worker，按需重启 | Celery Beat，调度/任务代码改动后重启 |
| `lensnode` | `sourcelens-lensnode-dev` | **必须手动重启** | Python 服务，挂载的 `lensnode/lensnode` 不会热加载，代码改动后须重启 |

**唯一需要重启 `backend-api` 的场景**：新增了 migrations 文件并需要作用于
数据库时，重启容器以执行迁移。

```bash
# 新增 migration 后，重启 api 使其生效（会执行 migrate）
docker restart sourcelens-api-dev

# worker / scheduler 代码改动后必须重启
docker restart sourcelens-worker-dev
docker restart sourcelens-scheduler-dev

# lensnode 代码改动后必须重启（挂载的 Python 代码不会热加载）
docker restart sourcelens-lensnode-dev
```

> 速记：**改普通代码** → api 自动生效、worker / scheduler / lensnode 手动重启；
> **加 migration** → 额外重启 api。

### Docker 生产构建（蓝绿零停机）

生产使用 `scripts/install.sh` 做蓝绿部署，**不要**直接 `docker compose up -d`
（API/UI 是 blue/green profiled 服务，裸 `up` 不会启动任何一个颜色）。详见下方
「零停机部署 (Blue/Green)」章节。

```bash
# 首次安装 / 每次升级，同一条命令幂等：
curl -fsSL https://raw.githubusercontent.com/HyperBDR/sourcelens/<tag>/scripts/install.sh \
    -o install.sh && chmod +x install.sh && ./install.sh <tag>
# 默认端口: HTTP 10080, HTTPS 10443
```

### Python 开发（非 Docker）
```bash
pip install -e .[dev]
```

### 测试与代码质量
```bash
pytest                              # 运行所有测试
pytest path/to/test.py              # 单个测试文件
python backend/manage.py test        # Django test runner（部分测试）
black --check backend/              # 检查格式
isort --check backend/              # 检查 import 顺序
```

### Django 管理命令
```bash
python backend/manage.py migrate
python backend/manage.py register_periodic_tasks   # 注册所有定时任务
python backend/manage.py createsuperuser
```

### 前端
```bash
cd frontend
npm install
npm run dev          # 开发服务器
npm run build        # 生产构建
npm run lint         # ESLint
npm run test:e2e     # Playwright E2E
```

## 架构概览

### Django 应用结构（backend/）

每个 Django app 都应尽量自包含：拥有自己的 models、views、serializers、services、migrations、periodic_tasks 和 tests。跨 app 的回归测试放在 `backend/tests/`。

主要 App：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oneprolabs/sourcelens](https://github.com/oneprolabs/sourcelens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
