---
trigger: always_on
description: `nop-entropy` 是一个可替代 Spring 的全栈 Java 框架（Nop 平台），采用可逆计算原理。
---

# AGENTS.md - Nop Entropy Development Guide

## Project Overview

`nop-entropy` 是一个可替代 Spring 的全栈 Java 框架（Nop 平台），采用可逆计算原理。

**Tech Stack**: Java 21, Maven (wrapper 4.0.0-rc-5), JUnit 5, Nop AutoTest, XLang。

模块分组和依赖关系见 `docs-for-ai/01-repo-map/module-groups.md`。

---

## Documentation Routing

## Hard Stop: Generated Files

- **Do not hand-edit any generated file or generated directory.**
- This includes **all** `_`-prefixed files such as `_*.xml`, `_*.java`, `_*.xmeta`, `_app.orm.xml`, `_service.beans.xml`, and anything under `_gen/`.
- If a change appears to require editing one of those files, stop and move one level up: edit the source model, Delta, non-underscore retention file, or generator template, then regenerate.

## Mandatory Full-Read Before Execution

**执行 `docs-for-ai/` 或 `ai-dev/` 下的任何 `.md` 文件描述的工作流前，必须完整读取该文件全文，不得只读一部分就开始执行。文件大小不是跳过阅读的理由。**

**`docs-for-ai/INDEX.md` is the authoritative docs navigation baseline.** 下表覆盖最常见的 agent 工作流。**当 By Task 和 By Code Location 都命中时，合并去重阅读清单。**

### By Task

| Task | Read first | Then read |
|------|-----------|-----------|
| 修改业务逻辑（service、action、biz） | `docs-for-ai/02-core-guides/service-layer.md` | `docs-for-ai/02-core-guides/api-and-graphql.md` |
| 新增或修改 ORM 模型 / 数据库表 | `docs-for-ai/02-core-guides/model-first-development.md` | `docs-for-ai/02-core-guides/delta-customization.md` |
| 修改 IoC 配置、注入、bean 定义 | `docs-for-ai/02-core-guides/ioc-and-config.md` | — |
| 修改 GraphQL API、crud 操作 | `docs-for-ai/02-core-guides/api-and-graphql.md` | `docs-for-ai/02-core-guides/service-layer.md` |
| 新增或修改 API 模型（`*.api.xml`）、理解 API codegen 生成物 | `docs-for-ai/02-core-guides/api-model-and-codegen.md` | `docs-for-ai/02-core-guides/api-and-graphql.md` |
| 修改 Delta 定制、覆盖已有模块 | `docs-for-ai/02-core-guides/delta-customization.md` | `docs-for-ai/02-core-guides/model-first-development.md` |
| 处理错误码、异常处理 | `docs-for-ai/02-core-guides/error-handling.md` | — |
| 修改工作流定义（`.xwf`）、理解审批流配置 | `docs-for-ai/02-core-guides/workflow-configuration.md` | `docs-for-ai/03-runbooks/build-approval-flow.md` |
| 修改平台框架核心（nop-core/nop-xlang/nop-xdef 等） | `docs-for-ai/02-core-guides/xlang-and-xpl-basics.md` | `docs-for-ai/02-core-guides/xdef-and-xdsl.md` |
| 编写或修改测试 | `docs-for-ai/02-core-guides/testing.md` | — |
| 编写或运行平台 E2E 测试（Playwright） | `nop-entropy-e2e/README.md` | — |
| 执行特定类型任务（按 runbook） | `docs-for-ai/03-runbooks/README.md` | 对应 runbook 文件 |
| 新增模块或修改包结构 | `docs-for-ai/01-repo-map/module-groups.md` | `docs-for-ai/01-repo-map/domain-module-pattern.md` |
| Draft, execute, or audit a plan under `ai-dev/plans/` | `ai-dev/plans/00-plan-authoring-and-execution-guide.md` | `ai-dev/logs/00-log-writing-guide.md` |
| Write or update daily dev log | `ai-dev/logs/00-log-writing-guide.md` | — |
| Write a bug fix note | `ai-dev/bugs/00-bug-fix-note-writing-guide.md` | — |
| Write an analysis | `ai-dev/analysis/00-analysis-writing-guide.md` | — |
| Write a design doc | `ai-dev/design/00-design-writing-guide.md` | `ai-dev/design/README.md` |
| Write or update skills | Check existing skills under `.opencode/skills/` first | — |

### By Code Location

> **⚠️动手前检查 Protected Areas**：如果代码位于下表 Protected Area 中，必须先遵守对应规则（plan-first / ask-first），再读技术文档。

| When touching this code | Read this |
|------------------------|-----------|
| `nop-core/`, `nop-xlang/` | `docs-for-ai/01-repo-map/module-groups.md` |
| `nop-biz/`, service/action 层 | `docs-for-ai/02-core-guides/service-layer.md` |
| `nop-dao/`, ORM/model 层 | `docs-for-ai/02-core-guides/model-first-development.md` |
| `nop-graphql/` | `docs-for-ai/02-core-guides/api-and-graphql.md` |
| `nop-chaos/` (demo app) | `docs-for-ai/01-repo-map/domain-module-pattern.md` |
| 业务模块（`nop-auth/`, `nop-job/`, `nop-wf/`, `nop-task/` 等） | `docs-for-ai/02-core-guides/service-layer.md` + 对应 runbook |
| 其他 `nop-*` 模块 | `docs-for-ai/01-repo-map/module-groups.md` |
| `nop-entropy-e2e/` (e2e tests) | `nop-entropy-e2e/README.md` |
| `ai-dev/` (any subdirectory) | 对应子目录的 `00-*-guide.md` 或 `README.md` |

---

## Docs Maintenance

**Docs live in `docs-for-ai/`** and are the primary source of platform usage knowledge. Always consult `docs-for-ai/INDEX.md` first for navigation. See `ai-dev/logs/00-log-writing-guide.md` for log writing conventions.

**`ai-dev/`** records the development process of the platform itself; it is not normative documentation for platform users.

### Documentation Domains

| | `docs-for-ai/` | `ai-dev/` |
|---|---|---|
| **定位** | **使用** Nop 平台所需的知识 | **开发** Nop 平台本身所需的知识 |
| **内容** | API、约定、开发模式、runbook | 设计决策、执行计划、复杂 bug 分析、经验教训 |
| **读者** | 基于 Nop 构建应用的 AI / 开发者 | 改造 Nop 框架本身的 AI / 开发者 |
| **权威性** | source of truth（规范性文档） | `design/` 规范性；`logs/`、`plans/`、`bugs/` 过程记录 |
| **路由入口** | `docs-for-ai/INDEX.md` | `ai-dev/logs/index.md` |

### Mandatory Updates

After completing any significant **CODE CHANGE**, you MUST:

1. **Update the daily dev log** at `ai-dev/logs/{year}/{month}-{day}.md` (reverse chronological, see `ai-dev/logs/00-log-writing-guide.md` for format).
2. **Update relevant owner docs** under `docs-for-ai/` when changing:
   - Service layer patterns → `docs-for-ai/02-core-guides/service-layer.md`
   - ORM/model patterns → `docs-for-ai/02-core-guides/model-first-development.md`
   - API or GraphQL → `docs-for-ai/02-core-guides/api-and-graphql.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entropy-cloud/nop-entropy](https://github.com/entropy-cloud/nop-entropy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
