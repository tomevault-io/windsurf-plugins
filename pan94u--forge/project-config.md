---
trigger: always_on
description: > **使命**：重构企业 IT 组织的人机分工。
---

# Forge Platform — AI-Native IT 交付与治理基础设施

> **使命**：重构企业 IT 组织的人机分工。
>
> AI 已经能够以零摩擦、高速、可靠的方式执行软件交付的大部分作业。在这个前提下，让人类继续做执行者，是一种巨大的浪费。Forge 让 AI 承接全部 SDLC 执行作业，让人类从执行中解放，专注于目标对齐、价值验收和战略决策；Enterprise Console 以 Agentic Loop 覆盖九大 IT 治理域，让 IT 治理也整体 AI-Native 化。
>
> **终极目标：解放人，聚焦价值创造，实现无限可能。**
>
> **定位**：AI-Native IT 基础设施，双层协同——
> - **执行层（Forge）**：SuperAgent + 28 Skill + 24 MCP 工具，三环架构（人类控制环主导 + 智能环执行 + 进化环学习），将 5-7 人团队的交付能力压缩到 1-2 人 + AI；人类角色重构为目标对齐者、价值验证者、结果裁判者
> - **治理层（Enterprise Console）**：Governance AI 以 Agentic Loop 覆盖九大 IT 治理域（架构/预算/团队/安全/数据/流程/合规/容量/供应商），Forge 执行数据通过 Governance Data API 驱动治理洞察与决策
>
> 面向全软件交付生命周期，支持集团-域-产业多层级组织规模化落地、可管理、可运营、可演进。

## 交互偏好

- **用中文交流**。所有回复、注释、文档默认使用中文
- **行动优先**：用户说"执行"/"继续"/"全部"时直接做，不要反问
- **简洁回复**：少说多做。避免冗长解释，给结果和关键数据
- **遵循四大开发纪律**（见下方"开发纪律"章节）
- **Git 提交署名**：commit 末尾必须包含 `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`

## Quick Start

```bash
# 环境要求：JDK 21（必须！JDK 8/17 会编译失败）
java -version  # 确认 21+

# 构建后端 jar（跳过测试加速）
./gradlew :web-ide:backend:clean :web-ide:backend:bootJar -x test

# 构建前端
cd web-ide/frontend && npm install && npm run build && cd ../..

# 构建企业控制台
cd enterprise-console && npm install && npm run build && cd ..

# Docker 一键部署（7 容器：backend + frontend + console + nginx + postgres + 2 MCP）
# 前置：先启动 SSO（cd infrastructure/sso && docker compose up -d）
cd infrastructure/docker
docker compose -f docker-compose.trial.yml up --build -d

# 访问 http://forge.local:19000（需 /etc/hosts 配置 127.0.0.1 forge.local sso.forge.local）

# 运行全量单元测试（当前 335 个）
./gradlew :web-ide:backend:test :adapters:model-adapter:test
./gradlew :agent-eval:test
```

## Architecture

Forge is a Gradle monorepo (Kotlin DSL) with the following modules:

- **plugins/**: Claude Code plugin system (Skills, Profiles, Commands, Hooks)
- **mcp-servers/**: MCP Server implementations (Kotlin/Ktor, HTTP transport)
- **web-ide/**: Web IDE (Next.js 15 frontend + Spring Boot 3 backend)
- **cli/**: Forge CLI (Kotlin + GraalVM Native)
- **adapters/**: Adapter layer isolating stable/volatile concerns
- **agent-eval/**: SuperAgent evaluation framework
- **skill-tests/**: Skill validation framework
- **knowledge-base/**: Knowledge repository (13 docs: profiles, conventions, ADR, runbooks)

## Key Design Decisions

1. **SuperAgent over Multi-Agent**: One intelligent agent dynamically switches Skill Profiles
2. **Skill over Prompt**: Professional knowledge encoded as reusable, composable Skills
3. **Baseline guarantees quality floor**: Baseline scripts must pass regardless of model
4. **Dual-loop architecture**: Delivery Loop (what) + Learning Loop (getting better)
5. **Adapter isolation**: Skills/baselines stable; models/runtime swappable via adapters
6. **MCP 工具聚合**: 5 MCP Server × 20 细粒度工具 → McpProxyService 20 聚合工具（5 handler 拆分路由）
7. **本地构建 + Docker 打包**: 不在 Docker 内编译（网络不可靠），本地 bootJar/npm build 后 Docker 只 COPY 产物

## Language & Conventions

- **Backend**: Kotlin 1.9+ on JDK 21, Spring Boot 3.3+
- **MCP Servers**: Kotlin + Ktor
- **Frontend**: TypeScript, React 19, Next.js 15 (App Router)
- **Build**: Gradle Kotlin DSL
- **Testing**: JUnit 5 + MockK + AssertJ (backend), Jest + Playwright (frontend)

## 认证 (@opc-ai/auth SDK)

- 前端: `@opc-ai/auth/client`, `initAuth({ tokenKey: 'forge_access_token' })` 兼容现有 6 个 API 文件零改动
- 后端: Kotlin `GatewayUserFilter.kt` (与 SDK 的 `synapseAuth()` 模式等价, 语言不同)
- Next.js 必须 `transpilePackages: ['@opc-ai/auth']` (SDK 发布 .ts 源文件)
- trial compose 的 enterprise-console 容器占 :19001, 本地联调 Gateway 用 :19002 避开

## Docker 部署注意事项

- **后端 Dockerfile**: 单阶段 `eclipse-temurin:21-jre-alpine`（Alpine 无 bash）
- **前端 Dockerfile**: COPY 预构建 `.next/standalone`, 需先在宿主机 `bun run build`
- **trial 模式**: `FORGE_SECURITY_ENABLED=false`, 无 Gateway, 无认证
- **production 模式**: Gateway sidecar, OIDC 认证, X-User-* headers
- **Volume 挂载**: `plugins/` 和 `knowledge-base/` 必须挂载为只读

## MCP 工具清单（20 个）

| 工具 | 来源 | 说明 |
|------|------|------|
| search_knowledge | builtin | 搜索知识库文档 |
| read_file | builtin | 读取知识库文件 |
| get_service_info | builtin | 获取服务信息 |
| run_baseline | builtin | 运行 baseline 脚本（Docker Alpine 内不可用） |
| query_schema | builtin | 查询数据库 schema |
| list_baselines | builtin | 列出可用 baselines |
| read_skill | builtin | 按需读取 SKILL.md 或子文件内容 |
| run_skill_script | builtin | 执行 Skill 脚本，60s 超时 |
| list_skills | builtin | 列出可用 Skill metadata |
| page_create | knowledge | 创建知识页面（local mode 写 Markdown 到 knowledge-base/） |
| workspace_write_file | workspace | 写文件到 workspace（需 workspaceId） |
| workspace_read_file | workspace | 读取 workspace 文件 |
| workspace_list_files | workspace | 列出 workspace 文件树 |
| workspace_compile | workspace | 编译/构建项目，自动检测项目类型 |
| workspace_test | workspace | 运行测试，自动检测测试框架 |
| workspace_start_service | workspace | 在 workspace 内启动服务进程（command + port） |
| workspace_stop_service | workspace | 按端口停止 workspace 内运行的服务 |
| update_workspace_memory | memory | Agent 主动更新 workspace 记忆 |
| get_session_history | memory | 读取历史 session 摘要 |
| analyze_codebase | memory | 对 workspace 执行结构分析，返回 JSON |
| plan_create | planning | 提交任务计划，等待用户确认后开始执行 |
| plan_update_task | planning | 更新单个任务状态（in_progress/done/failed/blocked）|
| plan_ask_user | planning | 向用户提选择题+问答题，阻塞等待回答 |
| plan_complete | planning | 提交执行总结并写入记忆 |

**注意**: workspace 工具通过 `callTool(name, args, workspaceId)` 三参数版调用，workspaceId 从 arguments 中提取。

## Security Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pan94u/forge](https://github.com/pan94u/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
