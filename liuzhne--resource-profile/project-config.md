---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Resource-Profile is a **Teacher-Student Resource Portrait System** (师生资源画像系统) — a microservices-based educational platform with a Vue 3 frontend, Spring Boot backend, and a Python AI inference sidecar (LLM + RAG over Milvus).

## EduCare 子系统路线图

The AI subsystem (agent-service + ai-inference-service + Multi-Agent + RAG + 本地 LLM) is tracked in **`docs/educare/EXECUTION_PLAN.md`** — single source of truth for Phase G/H/I/J 与 Release Readiness 的可执行原子任务清单、§1 下一步指针、§6 变更记录、§8 已知阻塞。当前状态（2026-08-27）：Phase G/H/I/J 与 R-1~R-4 已完结；保留交付面为 AgentLoop ReAct 默认主路径、Java student-data MCP :8094、Python knowledge-rag MCP :8095、dense RAG、干预反馈闭环及安全/运维基线。memory-server、Hybrid Retrieval、ModelRouter 与百分比灰度已经按瘦身决策删除，不得按历史勾选项误判为当前能力。下一步为 R-5 真模型/可观测验收和 R-6 上线总验收。设计源见 `docs/educare/IMPROVEMENT_2026_MAY.md`。**Read EXECUTION_PLAN.md first** instead of grepping git log or prior session jsonls. After finishing an execution-plan atomic task, follow §0 Update Protocol: 勾选 + 追加 `完成于 YYYY-MM-DD：备注` 行 + 更新 §1 指针 + 顶部"最近更新"。

## 修复方案文档同步规则（强制）

根目录的 [`ARCHITECTURE.md`](./ARCHITECTURE.md)、[`DECISIONS.md`](./DECISIONS.md)、[`RUNBOOK.md`](./RUNBOOK.md) 是项目级现状文档。任何任务只要**识别缺陷并提出修复方案**，或**实际落地修复**，就必须在同一次变更中同步维护这三份文件；文档未同步时，修复任务不得视为完成。

- `ARCHITECTURE.md`：更新受影响的模块边界、依赖、核心调用链、数据流或安全边界。若修复不改变架构，仍须在其“维护记录”中明确写“无架构影响”并指向受影响模块。
- `DECISIONS.md`：记录问题背景、选择该修复的原因、评估并放弃的方案、代价与后续约束。不要只写“修复 bug”。
- `RUNBOOK.md`：补充或修订复现、启动前提、验证命令、通过判据、排错步骤及必要的回滚方式。没有实际执行的验证必须标为“待验证”，不得写成已通过。
- 三份文件使用同一日期和同一修复标识/标题，链接到真实代码、配置、迁移或测试；删除/回退方案时也同步删除或标记过期内容，避免文档继续描述已不存在的能力。
- 纯格式、拼写或无行为影响的机械修改不属于“修复方案”；如果任务讨论了可执行的缺陷修复，即使代码最终未改，也属于本规则范围。

## Architecture

**Top-level layout:**
- `/backend` — Java/Spring Boot microservices (Maven multi-module)
- `/ai-inference-service` — Python FastAPI service for LLM inference & vector retrieval
- `/frontend` — Vue 3 + Vite single-page application
- `/docker` — Docker and docker-compose configuration
- `/sql` — Database initialization scripts

The Java side handles business logic, persistence, and orchestration. AI calls go through `agent-service`, which either talks to a local LLM via Spring AI's OpenAI-compatible client (default) or delegates to the Python `ai-inference-service` via Feign for RAG / embedding-heavy work.

## Backend — Spring Boot Microservices

**Build Tool:** Maven 3+
**Java Version:** 17
**Spring Boot:** 3.2.5
**Spring Cloud:** 2023.0.1
**Spring Cloud Alibaba:** 2023.0.1.0
**Spring AI:** 1.1.6 (GA, on Maven Central — no milestone repo needed)

**Microservices Modules:**

| Module | Port | Purpose |
|--------|------|---------|
| `gateway` | 8080 | Spring Cloud Gateway, routes to all services |
| `auth-service` | 8081 | Authentication & Authorization |
| `user-service` | 8082 | User management |
| `teacher-service` | 8083 | Teacher profile management |
| `student-service` | 8084 | Student profile management |
| `mental-service` | 8085 | Mental health assessment |
| `data-service` | 8086 | Data analysis and dashboard |
| `agent-service` | 8087 | LLM/Agent orchestration (Spring AI + Feign to Python) |
| `mcp-student-data` | 8094 | MCP server exposing student/academic/mental/attendance tools (Streamable HTTP `/mcp`) |
| `common` | — | Shared library (JWT, Result wrapper) |

**Key Dependencies:**
- MyBatis Plus 3.5.11 (ORM)
- JWT 0.12.5 (jjwt-api, jjwt-impl, jjwt-jackson)
- MySQL 8.0.33
- Spring Cloud Alibaba Nacos (Service Discovery + Configuration)
- Spring AI OpenAI starter (`spring-ai-starter-model-openai`) + MCP starters (`spring-ai-starter-mcp-client` / `spring-ai-starter-mcp-server-webmvc`) — used by `agent-service` and `mcp-student-data`
- Spring Cloud OpenFeign — used by `agent-service` to call `ai-inference-service`
- Lombok 1.18.30

**Backend Code Patterns:**

All services follow a consistent layered architecture:
- `entity/` — MyBatis Plus entities using Lombok `@Data`
- `mapper/` — Mappers extending `BaseMapper<Entity>` (no XML mappers; all SQL via annotations or MyBatis Plus wrappers)
- `service/` + `service/impl/` — Service interfaces and `@Service` implementations
- `controller/` — `@RestController` with `@RequestMapping` and `@RequiredArgsConstructor` for dependency injection
- `dto/` — Request/response DTOs using Lombok
- `config/` — Spring `@Configuration` classes
- `exception/GlobalExceptionHandler.java` — `@RestControllerAdvice` handling validation and runtime exceptions
- `feign/` — (agent-service) Feign clients for remote services like `AiInferenceClient`

**MyBatis Plus Global Config** (in each `application.yml`):
- Logic delete: field `deleted`, value `1` = deleted, `0` = not deleted
- ID type: `auto` (database auto-increment)
- `map-underscore-to-camel-case: true`

**API Response Pattern:**
All controllers return `Result<T>` from the `common` module:
- `Result.success(data)` → code 200
- `Result.error(message)` → code 500
- `Result.error(code, message)` → custom code

**Nacos Config Pattern:**
Every service imports two Nacos config files:
```yaml
config:
  import:
    - optional:nacos:{service-name}.yml?group=DEFAULT_GROUP
    - optional:nacos:common.yml?group=DEFAULT_GROUP
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuzhne/resource-profile](https://github.com/liuzhne/resource-profile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
