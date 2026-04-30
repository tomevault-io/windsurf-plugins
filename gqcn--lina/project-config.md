---
trigger: always_on
description: `Lina`是一个`AI驱动的全栈开发框架`，提供核心宿主服务、默认管理工作台、插件扩展机制与 AI 协作研发工作流。
---

# 项目概述

`Lina`是一个`AI驱动的全栈开发框架`，提供核心宿主服务、默认管理工作台、插件扩展机制与 AI 协作研发工作流。

- **前端**: `Vben5 + Vue 3 + Ant Design Vue + TypeScript（pnpm monorepo）`
- **后端**: `GoFrame + MySQL + JWT + Wasm 插件运行时`
- **研发流程**: `OpenSpec + SDD + AI 辅助协作`
- **参考项目**: `/Users/john/Workspace/github/imdap/ruoyi-plus-vben5`（默认管理工作台的前端样式和功能交互参考）

## 默认账号

- 用户名称: `admin`
- 登录密码: `admin123`

## 目录结构

```text
apps/                → MonoRepo项目目录
  lina-core/         → 全栈开发框架的核心宿主服务（GoFrame）
    api/             → 请求/响应 DTO（g.Meta 路由定义）
    internal/        → 后端核心代码实现
      cmd/           → 服务启动 & 路由注册
      consts/        → 全局常量定义
      controller/    → HTTP控制器（gf gen ctrl 自动生成骨架）
      dao/           → 数据访问层（gf gen dao 自动生成）
      model/         → 数据模型
        do/          → 数据操作对象（自动生成）
        entity/      → 数据库实体（自动生成）
      service/       → 业务逻辑层
    manifest/        → 交付清单
      config/        → 后端配置文件
      sql/           → DDL + Seed DML（版本 SQL 文件）
        mock-data/   → Mock 演示/测试数据（不随生产部署）
  lina-vben/         → 默认管理工作台（Vben5 前端 pnpm monorepo）
    apps/web-antd/   → 默认管理工作台应用（Ant Design Vue）
    packages/        → 共享库（@core, effects, stores, utils 等）
  lina-plugins/      → 插件样例与插件开发参考入口
hack/                → 项目脚本及测试用例文件
  tests/             → E2E 测试（Playwright）
    e2e/             → 测试用例文件
    fixtures/        → 测试 fixtures（auth, config）
    pages/           → 页面对象模型
openspec/            → OpenSpec相关文档
  changes/           → OpenSpec变更记录
```

## 常用命令

### 开发环境

```bash
make dev                         # 启动前后端（前端:5666, 后端:8080）
make stop                        # 停止所有服务
make status                      # 查看服务状态
make test                        # 运行完整E2E测试
make init                        # 初始化数据库（DDL + Seed 数据）
make mock                        # 加载 Mock 演示数据（需先执行 init）
make up                          # 默认用 claude 生成 commit message 并推送
make up tool=codex               # 使用 codex 生成 commit message 并推送
make up t=codex                  # tool 的短别名
make up tool=codex               # codex 默认模型为 gpt-5.1-codex-mini
make up tool=codex model=gpt-5.2 # 指定 AI 工具和模型（兼容 m=...）
```

### 后端

```bash
cd apps/lina-core
go run main.go          # 运行
make build              # 构建
make dao                # 生成 DAO/DO/Entity（修改 SQL 后）
make ctrl               # 生成控制器骨架（修改 API 定义后）
```

### 前端

```bash
cd apps/lina-vben
pnpm install                   # 安装依赖
pnpm -F @lina/web-antd dev     # 开发模式
pnpm run build                 # 构建
```

### E2E 测试

```bash
cd hack/tests
pnpm test              # 运行全部测试
pnpm test:headed       # 带浏览器界面运行
pnpm test:ui           # 交互式测试界面
pnpm test:debug        # 调试模式
pnpm report            # 查看 HTML 报告
```

测试文件命名规范：`TC{NNNN}*.ts`（如 `TC0001-login.ts`），放在 `hack/tests/e2e/` 对应模块目录下。

# 文档编写规范

`README.md`等技术文档编写需遵循规范 @.agents/instructions/markdown-format.instructions.md 。

- 仓库内所有目录级主说明文档统一使用英文 `README.md`，并同步提供内容一致的中文镜像 `README.zh_CN.md`。
- 新增目录说明文档时，必须在同一次变更中同步创建上述两份 README，不允许只维护单语版本。

# 开发流程规范

本项目采用`SDD`驱动开发，使用`OpenSpec`工具辅助落地。变更记录存放在 `openspec/changes/` 目录下。每个变更包含：`proposal.md`（提案）、`design.md`（设计）、`specs/`（增量规范）、`tasks.md`（任务清单）。

**执行流程**：
1. 通过`/opsx:explore`斜杠指令在给定需求描述的前提下进行探索式对话，分析问题、设计方案、评估风险。
2. 当探索式对话结束，形成清晰的解决方案时，通过`/opsx:propose`斜杠指令将其转化为正式的`OpenSpec`变更提案文档。命令形如`/opsx:propose feature-name`，其中`feature-name`为当前变更的描述性名称（使用`kebab-case`格式，如`user-auth`、`data-export`）。随后会在`openspec/changes`目录下会自动生成一个新的变更文件夹，包含增量规范系列文档(`spec/`)、技术实现方案(`design.md`)、变更提案与思路(`proposal.md`)和实现任务清单(`tasks.md`)。
3. 随后执行`/opsx:apply`开始按照`tasks.md`中的任务清单逐条执行，完成代码实现、测试、文档更新等工作。任务完成后需要调用`/openspec-review`技能进行代码和规范审查。如果涉及前端页面交互的功能，那么都需要创建`e2e`测试用例，并且在执行过程中自动运行测试用例，确保功能实现的正确性。
4. 用户反馈的问题或者改进点，需要调用`/openspec-feedback`技能进行修复和验证，并更新相关`OpenSpec`文档。任务完成后需要调用`/openspec-review`技能进行审查。
5. 用户确认本次迭代功能已完成没有问题后，则执行`/opsx:archive`斜杠指令将本次变更归档。归档前需要调用`/openspec-review`技能进行全面的变更审查，确保代码质量和规范遵循。

**关键规则**：
- 当用户报告问题缺陷/改进建议时（无论中文或英文），如果当前项目存在活跃的`OpenSpec`变更，那么必须调用`openspec-feedback`技能。**无论反馈内容是否与当前活跃迭代的主要功能相关，都必须追加到当前活跃迭代中**，便于统一管理和归档。
- 审查技能`/openspec-review`自动在以下节点触发：`/opsx:apply`任务完成后、`/opsx:feedback`任务完成后、`/opsx:archive`归档前。

# 架构设计规范

## 顶级要求

### 项目定位统一要求

1. `Lina` 的统一项目定位是`AI驱动的全栈开发框架`。所有前端、后端、配置、脚本输出、OpenAPI 描述、系统信息页和项目文档中的项目介绍，都必须围绕这一定位展开。
2. 默认管理工作台、系统管理模块、用户权限模块等能力属于 `Lina` 提供的默认入口和内建通用能力，不构成项目的唯一产品边界。

### 核心宿主边界要求

1. `apps/lina-core` 是全栈开发框架的核心宿主服务，负责提供通用模块接口能力、组件能力、系统治理能力与插件扩展能力。
2. `lina-core` 的设计必须优先保证通用性、稳定性和可复用性，不得与具体管理工作台页面的展示结构、交互细节或前端框架实现强绑定。
3. 若需求仅来源于表格列、筛选项、树选择器、路由装配、工作台聚合、下拉选项等工作台展示变化，应优先通过工作台适配接口或前端适配层解决，而不是直接修改 `lina-core` 的核心领域契约、通用 service 语义或存储模型。

## 模块设计规范

### 模块功能设计规范

1. 业务模块的枚举值都应当使用字典模块维护其对应的字典类型和字典数据，而不是在代码中硬编码枚举值。比如：操作日志中的操作类型、操作结果，登录日志中的登录状态、文件管理的业务场景等都应该使用字典类型进行维护。

### 模块解耦设计原则

所有前后端模块必须采用解耦设计，业务模块支持按需启用/禁用。设计和实现时须遵循以下原则：

1. **模块可禁用**：每个业务模块（如部门、岗位、字典等）应当是独立的，可以通过配置禁用。禁用某模块后，所有依赖该模块的功能必须自动降级或隐藏，不能出现报错或空白区域。
2. **前端联动隐藏**：当一个模块被禁用时，前端所有涉及该模块的`UI`元素（菜单项、表单字段、表格列、搜索条件、按钮等）必须完全隐藏，而非仅禁用或置灰。例如：禁用"部门"和"岗位"模块后，用户管理页面中不应出现任何部门和岗位相关的筛选条件、表格列或表单字段。
3. **后端松耦合**：后端服务间的依赖应通过接口或可选引用实现，避免硬依赖。当被依赖的模块被禁用时，相关字段返回零值或忽略即可，不应抛出错误。
4. **数据完整性**：模块禁用仅影响功能和展示层，不应删除或破坏已有数据。重新启用模块后，历史数据应能正常恢复使用。

# 接口设计规范

所有前后端`API`必须严格遵循`RESTful`设计规范，`HTTP`方法与操作语义必须一一对应：

| HTTP 方法 | 语义 | 适用场景 |
|-----------|------|---------|
| **GET** | 读取（无副作用） | 列表查询、详情获取、树形数据、导出、下拉选项等所有只读操作 |
| **POST** | 创建资源/执行动作 | 新增记录、文件上传、导入、登录、登出等 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gqcn/lina](https://github.com/gqcn/lina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
