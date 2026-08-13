---
trigger: always_on
description: nexusx：从 SQLModel 类自动生成 GraphQL API，并提供 Core API 模式构建用例响应的 Python 库。
---

# CLAUDE.md

## 项目定位

nexusx：从 SQLModel 类自动生成 GraphQL API，并提供 Core API 模式构建用例响应的 Python 库。
- **GraphQL 模式**：SDL 自动生成 + DataLoader 批量关系加载 + MCP 服务集成
- **Core API 模式**：DefineSubset DTO + ErManager + Resolver 模型驱动的响应构建

## 技术栈

- Python >= 3.10
- 核心依赖：sqlmodel, graphql-core, fastapi, aiodataloader
- 可选依赖：fastmcp (MCP 服务)
- 构建工具：hatchling
- 测试：pytest + pytest-asyncio (asyncio_mode=auto)
- Lint：ruff (line-length=100, rules: E/F/I/UP/B, ignore B008)
- 类型检查：mypy (strict)
- 版本：1.0.0

## 目录结构

```
src/nexusx/            # 主包
├── decorator.py               # @query / @mutation 装饰器
├── handler.py                 # GraphQLHandler — 查询执行入口
├── sdl_generator.py           # SQLModel → GraphQL SDL
├── type_converter.py          # Python 类型 → GraphQL 类型
├── query_parser.py            # GraphQL 查询 → FieldSelection 树
├── standard_queries.py        # AutoQueryConfig 自动生成 by_id/by_filter
├── resolver.py                # Resolver — Core API 模型驱动遍历引擎
├── subset.py                  # DefineSubset / SubsetConfig — DTO 生成
├── context.py                 # ExposeAs / SendTo / Collector — 跨层数据流
├── relationship.py            # 自定义（非 ORM）关系定义
├── er_diagram.py              # Mermaid ER 图生成
├── introspection.py           # GraphQL 内省支持
├── graphiql.py                # GraphiQL UI
├── response_builder.py        # 响应构建工具
├── type_service.py            # 类型服务
├── execution/                 # 查询执行器
├── loader/                    # DataLoader 关系加载 (ErManager + factories + pagination)
├── discovery/                 # 实体发现
├── scanning/                  # 方法扫描 (@query/@mutation 发现)
├── use_case/                  # UseCase MCP 服务
│   ├── business.py            #   UseCaseService + BusinessMeta
│   ├── introspector.py        #   ServiceIntrospector
│   ├── server.py              #   create_use_case_mcp_server (4层工具)
│   ├── manager.py             #   UseCaseManager + UseCaseResources
│   ├── types.py               #   UseCaseAppConfig
│   └── context.py             #   FromContext
├── mcp/                       # MCP 服务 (GraphQL)
│   ├── builders/              #   Schema 格式化 + 类型追踪
│   ├── managers/              #   单应用 / 多应用管理器
│   ├── tools/                 #   MCP 工具实现
│   └── types/                 #   配置与错误类型
└── utils/                     # 工具函数

demo/                          # 演示应用
├── app.py                     # GraphQL demo (User/Post/Comment)
├── core_api/                  # Core API demo
├── use_case/                  # UseCase MCP demo
│   ├── mcp_server.py          #   UseCase MCP 服务 demo
│   ├── fastapi.py             #   FastAPI 集成 demo
│   └── voyager_demo.py        #   Voyager 可视化 demo
├── mcp_server.py              # MCP 多应用 demo (GraphQL)
└── mcp_server_simple.py       # MCP 单应用 demo (GraphQL)
tests/                         # 测试用例
```

## 公共 API

```python
from nexusx import (
    # 装饰器
    query,                    # 标记 GraphQL 查询方法
    mutation,                 # 标记 GraphQL 变更方法

    # GraphQL 模式核心类
    GraphQLHandler,           # SDL 生成 + 查询执行
    SDLGenerator,             # SDL 生成器（一般不直接使用）
    QueryParser,              # 查询解析器（一般不直接使用）
    FieldSelection,           # 查询解析结果类型
    AutoQueryConfig,          # 自动查询配置
    add_standard_queries,     # 手动注册自动查询

    # Core API 模式
    DefineSubset,             # DTO 基类（从 SQLModel 实体生成 Pydantic 模型）
    SubsetConfig,             # 声明式 DTO 配置
    ErManager,                # 实体关系管理器（DataLoader + Resolver 工厂）
    Loader,                   # resolve_* 方法中声明 DataLoader 依赖

    # 跨层数据流
    ExposeAs,                 # 父→后代上下文暴露
    SendTo,                   # 后代→祖先值收集
    Collector,                # 聚合 SendTo 收集的值

    # 自定义关系 & ER 图
    Relationship,             # 自定义非 ORM 关系定义
    ErDiagram,                # Mermaid ER 图生成

    # UseCase MCP 模式
    UseCaseService,           # 业务服务基类（定义 use case 方法）
    UseCaseAppConfig,         # 应用配置（name, services, description, context_extractor）
    FromContext,              # 标记从 MCP context 注入的参数
    create_use_case_mcp_server, # 创建 4 层渐进式披露 MCP 服务器
    create_flat_mcp_server,   # 创建扁平 MCP 服务器（每个方法一个 tool）
    create_use_case_router,   # 创建 FastAPI 路由
    create_use_case_voyager,  # 创建 UseCase Voyager 可视化
)

# MCP (GraphQL)
from nexusx.mcp import (
    create_simple_mcp_server, # 单应用 MCP 服务
    create_mcp_server,        # 多应用 MCP 服务
    AppConfig,                # 应用配置
)
```

## 开发命令

```bash
./scripts/check-ci.sh                                       # 本地执行与 CI 一致的检查
uv run pytest                                              # 运行测试
uv run ruff check src/ tests/                              # Lint 检查
uv run ruff check --fix src/ tests/                        # Lint 修复
uv run mypy src/                                           # 类型检查
uv run python -m demo.app                                  # 启动 GraphQL demo
uv run uvicorn demo.core_api.app:app --port 8001           # 启动 Core API demo
uv run --with fastmcp python -m demo.mcp_server            # 启动 MCP demo (stdio, GraphQL)
uv run --with fastmcp python -m demo.use_case.mcp_server   # 启动 UseCase MCP demo (stdio)
uv run uvicorn demo.use_case.fastapi:app --port 8007       # 启动 UseCase FastAPI demo
```

## 核心约定

### GraphQL 模式

#### 字段命名规则
`@query`/`@mutation` 方法自动生成字段名：`{entityName}{MethodName}`
- `User.get_all` → `userGetAll`
- `Post.create` → `postCreate`

#### 实体发现规则

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KLR-Pattern/nexusx](https://github.com/KLR-Pattern/nexusx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
