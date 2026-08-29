---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

> ⚠️ 本文件与 `CLAUDE.md` 内容应保持一致（两者是同一份指引的镜像）。修改本文件时同步更新 `CLAUDE.md`。
> ⚠️ 项目仍是「未发布版」（见 `README.md` 顶部提示），部分功能逻辑仍在修缮中。研究现状时以实际代码为准，本文件的功能状态描述可能滞后。

所有回答以及中间的思考过程，尽可能全部使用中文进行解答。

所有实现的代码，在完成后都需要反复核对逻辑是否正确，是否与用户提出的需求匹配，如果有问题修复后需要再次核对。

任何修改都应该遵守这个完整校验流程：

**前端代码修改完成后**：
1. `npm run dev` 启动前端，验证页面正常无报错
2. 验证无误后关掉进程
3. `npm test` 运行前端测试用例（`tests/unit/`），确保所有测试通过

**后端代码修改完成后**：
1. `mvn clean install -DskipTests` 编译所有模块无误
2. `mvn spring-boot:run` 正常启动 rule-engine-server 后端服务
3. `mvn test` 运行后端测试用例，确保所有测试通过

**Codex 本地验证服务超时**：
- 启动前端、后端、MOCK 等长期本地验证服务时，工具调用超时统一设置为 `1800000ms`（30 分钟），不得继续使用 `600000ms`（10 分钟）。
- 服务验证完成后必须主动停止相关进程，避免依赖超时自动清理。

**测试用例编写与修改规范**：
- 新增或修改测试用例时，必须仔细校对测试逻辑是否正确覆盖目标功能点
- 测试用例的断言条件、输入数据、预期结果必须与实际需求完全匹配
- 不得编写"假通过"的测试用例（断言永远为 true，或测试目标与断言无关）
- 修改现有测试用例前，先理解原有测试意图，避免破坏有效覆盖

修复任何代码或修改内容之前，必须先完整研究项目整体结构和相关功能的实现方式，所有改动必须与现有实现风格保持一致且逻辑连贯。不得在未理解上下文的情况下孤立修改某一处代码，导致同一功能出现多种实现方式或前后端逻辑不一致。

## 项目概述

天枢决策引擎（com.hengshucredit.rule，仓库名 qlexpress-rule）是一套基于 **Spring Boot 3.5** 与 **QLExpress 4** 的可视化风控决策系统：支持 **决策表、决策树、决策流、规则集、交叉表、评分卡、复杂交叉表、复杂评分卡、QL 脚本** 等 9 种模型的可视化编排，涵盖项目、变量、名单、外数 API、外部数据库、模型、函数、规则测试、血缘分析、分流实验、执行日志和账单管理等能力。

> 后端 Java 包名前缀统一为 `com.hengshucredit.rule.*`（注意不是 hscredit），Maven `groupId` 为 `com.hengshucredit.rule`，`artifactId: qlexpress-rule`。

## 环境要求

- JDK 17
- Maven 3.6+
- MySQL 8
- Redis（与 server 使用同一实例，含密码和 database）
- Node.js 20.19+（Vite 8 最低要求；不设置最高版本；建议使用当前维护中的 LTS，已验证 Node.js 26.4.0）

## 模块架构

| 模块 | 说明 | 端口 |
|------|------|------|
| `rule-engine-model` | 公共实体与 DTO | - |
| `rule-engine-core` | 规则编译与执行核心 | - |
| `rule-engine-server` | 管理端 REST API | 8080 |
| `rule-engine-client` | 客户端 SDK | - |
| `rule-engine-example` | 集成示例服务 | 7070 |
| `rule-engine-builder-ui` | Vue 3 前端控制台（独立部署） | 9090（dev）|
| `rule-engine-mysql` | MySQL docker-compose 配置 | - |
| `rule-engine-redis` | Redis docker-compose 配置 | - |

### 部署架构

```
浏览器 ←→ rule-engine-builder-ui（前端，dist/ 独立部署）
         ↓
      rule-engine-server（后端 API，8080）
      ↙              ↘
   MySQL           Redis（Pub/Sub 规则推送）
                       ↓
            业务应用（rule-engine-client SDK）
```

- **前后端分离**：`rule-engine-server/pom.xml` 中 `skip.ui.build` 默认为 `true`，前端 `npm run build` 产物在 `dist/` 独立部署，不混入后端目录
- **客户端不直连 MySQL**：通过 HTTP 拉取服务端已编译规则，缓存在进程内 L1 内存
- **Redis 必须与 server 同一实例**：项目规则/函数变更向 `rule:push:{projectCode}` 发布，GLOBAL 变更向 `rule:push:broadcast` 发布；客户端必须用真实 `projectCode` 订阅项目频道，`appName` 不参与项目路由
- **执行日志**：默认 HTTP 上报；classpath 中存在 `KafkaTemplate` Bean 时自动切换为 Kafka（主题 `rule-execution-log`）

## 开发命令

### 后端 (Maven)

```bash
# 编译所有模块
mvn clean compile

# 运行测试 / 单个测试类 / 单个测试方法
mvn test                                    # 运行所有后端测试
mvn test -Dtest=ClassName                   # 运行单个测试类
mvn test -Dtest=ClassName#methodName        # 运行单个测试方法

# 启动 rule-engine-server（端口 8080）
cd rule-engine-server && mvn spring-boot:run

# 启动 rule-engine-example（端口 7070）
cd rule-engine-example && mvn spring-boot:run

# 完整构建（跳过前端）
mvn clean package -DskipTests
```

### 前端 (Vue 3)

```bash
cd rule-engine-builder-ui

npm ci
npm run dev      # 开发模式（9090，/api 代理到后端 8080）
npm run build    # 生产构建
npm run lint     # 代码检查
npm test         # 运行所有单元测试（tests/unit/）
npm run test:watch   # 监听模式
npm run test:coverage # 覆盖率报告
npm run test:e2e:dist # 真实 dist + 模拟 API 的 Playwright 烟测
npm run test:e2e:full # 设置 E2E_BASE_URL 后执行真实前后端联调
```

### 基础设施

```bash
# 根目录 docker-compose 已包含 mysql + redis + 初始化（空数据卷依次执行 schema 与 export）
cp .env.example .env  # PowerShell 使用 Copy-Item .env.example .env
# 替换 .env 中全部占位值
docker compose --env-file .env up -d
# 单独启动：
cd rule-engine-mysql && docker-compose up -d   # MySQL（仅 compose 文件，无 dockerfile）
cd rule-engine-redis && docker-compose up -d    # Redis
```

### 数据库初始化

- `schema.sql` 只包含数据库、表和索引等结构 DDL；`export_202607161151.sql` 是当前唯一的初始数据快照，不生产 `data-system.sql`
- 空 Docker 数据卷首次启动依次执行 `01-schema.sql` 和 `02-export.sql`；根编排的 `mysql-init` 对已有数据卷只重复执行 schema，不自动重放会覆盖数据的 export
- 手工完整恢复顺序：删除 `rule_engine` 数据库、执行 `schema.sql`、执行 `export_202607161151.sql`；export 会清空其覆盖的全部数据表
- `data-example.sql` / `data-tianshu-example.sql` 仅作为可选示例数据脚本手动导入，不属于系统初始数据来源
- 仅在 README 的 12 节「实现边界」中保留的已知限制（如血缘仅静态识别脚本引用）才是真实待修缮项
- Compose 不提供共享默认密码；全新数据卷根据 `MYSQL_USERNAME` / `MYSQL_PASSWORD` 创建应用账号，已有数据卷需由数据库管理员预先创建或更新最小权限账号
- Maven/JAR 启动时会自动读取当前目录或上级目录的 `.env`；系统环境变量和命令行参数优先覆盖文件值，生产仍应通过 Secret/KMS 注入 MySQL、Redis、控制台和 `RULE_AUTH_MASTER_KEY` 等真实凭据

## 核心编译器（rule-engine-core）

`RuleCompiler` 接口定义 `compile(modelJson, varContext)` 方法，各模型有独立实现：

| 编译器 | 对应模型 |
|--------|----------|
| `DecisionTableCompiler` | 决策表 |
| `DecisionTreeCompiler` | 决策树 |
| `DecisionFlowCompiler` | 决策流 |
| `RuleSetCompiler` | 规则集 |
| `CrossTableCompiler` | 交叉表 |
| `ScorecardCompiler` | 评分卡 |
| `AdvancedCrossTableCompiler` | 复杂交叉表 |
| `AdvancedScorecardCompiler` | 复杂评分卡 |
| `ScriptPassthroughCompiler` | QL 脚本（直接透传） |

其他核心类：
- `CompileResult` - 编译结果，含生成的脚本和输出变量信息
- `VarContext` - 编译时变量上下文（`varId → scriptName` 映射，解决大小写不一致）
- `ConditionCompiler` / `ActionDataCompiler` - 条件与动作的子编译单元
- `QLExpressEngine` / `QLExpressEngineFactory` - QLExpress 执行引擎
- `AggregateBuiltinFunctionRegistry` - 内置聚合函数（sum/count/max/min/avg）

## 前端关键路径


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hengshu-credit/tianshu-decision-engine](https://github.com/hengshu-credit/tianshu-decision-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
