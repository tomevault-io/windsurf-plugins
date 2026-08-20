---
trigger: always_on
description: 本文件是仓库内 coding agent 的执行入口。项目术语以根目录 `CONTEXT.md` 为准；实现状态以源码、配置和测试为准；README 与验收文档用于理解产品目标和已验证边界。
---

# AGENTS.md

本文件是仓库内 coding agent 的执行入口。项目术语以根目录 `CONTEXT.md` 为准；实现状态以源码、配置和测试为准；README 与验收文档用于理解产品目标和已验证边界。

## 项目定位

PE Reverse Analyzer 是一个面向逆向分析、软件安全研究和源码重构的本地优先平台。目标不是只输出反编译文本，而是把样本、分析计划、工具执行、证据、知识图谱、源码工程、真实构建和行为比较组织为可追溯流程。

当前仓库同时包含三类交付物：

1. **分析内核**：Python 包 `reverse_analyzer`，提供 CLI、静态/动态/GUI 分析、Provider/Capability、证据清单、PE/APK/协议处理、补丁和源码重构。
2. **生产平台**：Go 控制面、React 中文工作台、PostgreSQL 持久化和隔离 Runner，承载上传、Experiment/Flow、模型 Provider、知识库、制品、构建、补丁、终端与访问控制。
3. **技能路由包**：`reverse-skills/skills/`，供 AI 客户端按目标、意图和工具条件选择逆向工作流；它可独立使用，也被平台目录索引。

完整源码重构主链路为：

```text
样本/归档上传
  -> 解包、反编译、资源与静态证据
  -> 知识图谱与模块关系
  -> 模型逐模块理解和源码修改
  -> 工程结构与依赖锁定
  -> 隔离真实构建
  -> 编译诊断与有界修复
  -> 原程序/重构程序行为比较
  -> 可信工件门禁派生 complete_buildable
```

`complete_buildable=true` 只表示当前受支持目标已通过结构、依赖、构建和行为门禁，不表示逐字恢复编译前原始源码，也不能替代其他平台、设备或签名环境的独立验收。详细门禁见 `docs/acceptance/p11_complete_reconstruction.md`。

## 运行架构

```text
Browser
  -> Go Control Plane :8090
       -> PostgreSQL（生产元数据、租户、审计、知识、Provider）
       -> Workspace Volume（上传、日志、制品、源码、补丁、报告）
       -> Runner API :8091
            -> Docker/Podman 隔离 Worker
                 -> python -m reverse_analyzer ...
                 -> archive_reconstruct / web_patch_bridge / CMake build
       -> Provider Broker
            -> 外部 OpenAI-compatible 模型

Go Control Plane 同时提供 frontend/dist 静态文件和 /api/* JSON/SSE 接口。
```

关键运行原则：

- Go 是 Web/API、认证、持久化和任务编排的事实入口；Python 不再承担生产 Web 服务。
- Worker 默认只读根文件系统、丢弃 capabilities、启用 `no-new-privileges`、限制 CPU/内存/PID，并默认 `network=none`。
- 模型请求由 Go Provider Broker 发出；密钥不进入分析或构建 Worker。
- 创建 Experiment 只产生计划。实际分析必须提交确认短语 `EXECUTE_LOCAL_ANALYSIS`。
- 生产模式强制 PostgreSQL、认证和远程 Runner；持久化不可用时 readiness 降级，不回退到本地 JSON。
- 本地非生产模式可把 Experiment 保存为 `experiments/<id>.json`、事件保存为 `experiments/<id>/events.jsonl`。

部署关系见 `Dockerfile`、`Dockerfile.runner`、`deploy/compose.production.yml` 和 `docs/web-deployment.md`。生产站点为 `https://pe.toporeduce.cn`；远程代码知识图谱端点为 `https://pe.toporeduce.cn/codegraph/rpc`。

## 核心对象

不要混用以下三套对象：

### 平台 Experiment

定义在 `cmd/reverse-analyzer-server/main.go`，是 Web 平台的持久化任务聚合根，包含：

- 样本、选项、状态、历史、制品与错误；
- `ReconstructionState`：分析、源码、结构、依赖、构建、行为和 `complete_buildable` 门禁；
- `OrchestrationState`：Flow、Task、Subtask、ToolCall 与最后事件序号；
- 追加式 `Event`：sequence、type、status、message、data，通过 SSE 推送给前端。

Experiment 状态变化必须同步更新持久化、事件和编排快照。控制面重启时遗留 `running` 任务会转为 `failed`，并要求重新生成构建与行为证据。

### Python Session / Flow

定义在 `reverse_analyzer/core/`、`runtime/` 和 `cli.py`，用于 CLI 与分析内核中的工具、Provider、审计和工件编排。它不是 Go 平台 Experiment 的数据库模型；二者通过命令参数、JSON、文件和事件连接。

### 技能 Case

由 `reverse-skills/skills/scripts/case-init.py` 创建，结构为 `case.json`、`notes.md`、`evidence/`、`reports/`，用于 AI 技能工作流的本地证据记录。它不是平台 Experiment，也不是 Python Session。修改 Case schema 时同步检查 `case-review` 脚本和相关测试。

## 关键事实源

| 主题 | 事实源 | 说明 |
| --- | --- | --- |
| 项目术语 | `CONTEXT.md` | Case、Plan、Skill、Tool index、Evidence 等共享词汇。 |
| Python CLI | `reverse_analyzer/cli.py`、`reverse_analyzer/__main__.py`、`pyproject.toml` | `reverse-analyzer` 与 `python -m reverse_analyzer` 入口。 |
| Go API 与模型 | `cmd/reverse-analyzer-server/main.go` | 路由、Experiment、编排、事件、worker 和静态前端服务。 |
| RBAC | `cmd/reverse-analyzer-server/auth.go` | `viewer`、`analyst`、`admin` 权限和 workspace 约束。 |
| 数据库迁移 | `cmd/reverse-analyzer-server/migrations/*.sql` | `database.go` 内嵌并按连续版本执行；`schema.sql` 不是运行时迁移事实源。 |
| Runner 边界 | `cmd/reverse-analyzer-runner/main.go` | 允许的 job kind、参数、环境变量、容器限制与输出协议。 |
| Provider 配置 | `cmd/reverse-analyzer-server/providers.go`、`provider_broker.go` | 平台 Provider、模型协议、优先级、密钥槽与用量。 |
| Python 运行目录 | `reverse_analyzer/config.py` | workspace、knowledge、sessions、reports 的环境变量规则。 |
| 证据清单 | `reverse_analyzer/evidence/manifest.py` | 显式声明工件、相对路径、SHA-256、不可用阶段和验证规则。 |
| 技能路由 | `reverse-skills/skills/config/routing.json` | 技能工作流唯一事实源。 |
| 技能工具契约 | `reverse-skills/skills/config/tool-manifest.json` | 工具 ID、依赖和执行边界。 |
| 技能导航 | `reverse-skills/skills/INDEX.md` | 生成文件，只用于导航。 |
| 前端契约 | `frontend/src/App.tsx`、`frontend/src/api/catalog.ts` | API DTO、页面状态、SSE 消费和目录归一化。 |
| 生产部署 | `deploy/compose.production.yml`、`Dockerfile*` | Web、PostgreSQL、Runner、卷、secret 和容器边界。 |
| 生产验收 | `docs/acceptance/p10_platform_production.md` | 多租户、迁移、审计、备份恢复和生产门禁。 |
| 重构验收 | `docs/acceptance/p11_complete_reconstruction.md` | 真实模型、隔离构建、行为比较和可信工件门禁。 |

`README.md` 描述面向用户的能力与命令；`docs/skill_parity_matrix.md` 描述能力实现状态；`.understand-anything/knowledge-graph.json` 和 `docs/项目知识图谱.md` 用于跨模块导航。事实冲突时，优先级为：源码/配置/测试 > 当前验收报告 > README/知识图谱 > 历史工作记录。

## 模块导航

### Python 分析内核

- `reverse_analyzer/cli.py`：CLI 参数、分析生命周期和 Provider 调度。
- `reverse_analyzer/core/`：Session、Flow/Task/Subtask、Capability DTO、审计和 IR 基础模型。
- `reverse_analyzer/runtime/`、`reverse_analyzer/agent/`：实验运行时、规则式 Agent 和工具编排。
- `reverse_analyzer/tools/`：静态分析、Android、协议、内存、GUI、patch 等无状态工具入口。
- `reverse_analyzer/providers/`：需要生命周期、外部依赖或运行时状态的能力实现。
- `reverse_analyzer/evidence/`：证据 manifest、来源链和校验。
- `reverse_analyzer/source/`、`source_reconstruction.py`：工程生成、依赖、构建和行为验证。
- `reverse_analyzer/patch/`：补丁计划；底层事务写入逻辑位于工具层。
- `reverse_analyzer/report/`、`knowledge/`、`dashboard.py`：报告、知识演化和可视化数据。

Provider 生命周期保持：

```text
supports -> plan -> validate -> execute -> optional rollback -> collect_artifacts
```

每一阶段都应保留明确状态和审计；缺失依赖不得伪装成成功。

### Go 控制面


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DamonZS/PE-reverse-skill](https://github.com/DamonZS/PE-reverse-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
