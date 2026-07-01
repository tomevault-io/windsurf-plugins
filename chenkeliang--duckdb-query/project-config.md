---
trigger: always_on
description: > **更新时间**：2026-05-18
---

# DuckQuery 项目 AGENT 规则（v3.3）

> **更新时间**：2026-05-18  
> **适用范围**：全项目（前端、后端、测试、文档）  
> **权威性**：本文件为唯一 AGENT 约束来源。  
> **契约真相表**：`frontend/src/api/*` 在用路径、部署入口与字段语义见 [`docs/API_CONTRACT_FE_BE.md`](docs/API_CONTRACT_FE_BE.md)（与 §9 响应格式互补）。**改 API 时须先更新该表，再改后端与 `frontend/src/api/*`。**  
> **调用链路**：入湖 / 查询 / 元数据 / 异步的全局调用图见 [`docs/ARCHITECTURE_CALL_MAP.md`](docs/ARCHITECTURE_CALL_MAP.md)；改相关逻辑前先对照该文档。

---

## 目录
1. 项目架构与技术栈  
2. 目录结构与关键文件  
2.1 前后端契约与 PR 模板  
3. 运行与测试  
4. 前端开发规范  
5. UI / 样式规范（**禁止自定义**）  
6. 查询结果表格（TanStack DataGrid）  
7. 状态管理与数据获取  
8. 后端开发规范  
9. API 与响应规范  
10. 测试规范  
11. 质量检查清单  
12. 代理行为约束

---

## 1. 项目架构与技术栈

### 技术栈
| 层级 | 技术 |
|------|------|
| 前端框架 | React 18 + Vite + TypeScript |
| UI 组件 | shadcn/ui + Tailwind CSS |
| 状态管理 | TanStack Query 5.x + React Hooks |
| 表格 | TanStack Table + DataGrid（查询结果区） |
| 后端框架 | FastAPI + Python 3.11+ |
| 数据库 | DuckDB（本地）+ MySQL/PostgreSQL/SQLite（联邦查询） |
| 国际化 | react-i18next |

### 入口文件
| 入口 | 路径 | 说明 |
|------|------|------|
| 前端主入口 | `frontend/src/main.tsx` | React 应用入口 |
| 查询工作台 | `frontend/src/QueryWorkbenchPage.tsx` | 查询主页面 |
| 后端入口 | `api/main.py` | FastAPI 应用入口 |

---

## 2. 目录结构与关键文件

```
duckdb-query/
├── api/                              # 后端 FastAPI
│   ├── core/                         # 核心模块
│   │   ├── common/                   # 通用工具（时区、配置、缓存）
│   │   ├── data/                     # 数据处理（文件导入、Excel）
│   │   ├── database/                 # 数据库引擎
│   │   └── services/                 # 服务层（任务管理）
│   ├── routers/                      # API 路由
│   ├── models/                       # Pydantic 模型
│   ├── utils/                        # 工具函数（响应格式）
│   └── tests/                        # 后端测试
├── frontend/
│   └── src/
│       ├── api/                      # TypeScript API 模块 ⭐
│       │   ├── client.ts             # Axios 客户端配置
│       │   ├── types.ts              # 共享类型定义
│       │   ├── queryApi.ts           # 查询 API（含取消同步查询等）
│       │   ├── tableApi.ts           # 表 API
│       │   ├── dataSourceApi.ts      # 数据源 API
│       │   ├── databaseSchemasApi.ts # 外部库 schemas / 表列表 / 表详情
│       │   ├── settingsShortcutsApi.ts # 设置：快捷键 API
│       │   ├── fileApi.ts            # 文件 API
│       │   ├── asyncTaskApi.ts       # 异步任务 API
│       │   ├── pivotQueryApi.ts      # 透视 generate/preview、SQL 收藏、应用配置 API
│       │   ├── setOperationsApi.ts   # 集合运算 API
│       │   ├── joinQueryApi.ts       # 多表 JOIN（POST /api/query）
│       │   └── index.ts              # 统一导出（@/api）
│       ├── hooks/                    # 共享 Hooks（TanStack Query）⭐
│       │   ├── useDuckDBTables.ts    # DuckDB 表列表
│       │   ├── useDataSources.ts     # 数据源列表
│       │   ├── useDatabaseConnections.ts # 数据库连接
│       │   ├── useTableColumns.ts    # 表列信息
│       │   ├── useSchemas.ts         # Schema 列表（经 @/api → listConnectionSchemas）
│       │   ├── useSchemaTables.ts    # Schema 下表列表（经 @/api）
│       │   └── ...
│       ├── utils/                    # 工具函数 ⭐
│       │   ├── cacheInvalidation.ts  # 缓存失效工具
│       │   ├── sqlUtils.ts           # SQL 工具
│       │   └── ...
│       ├── Query/                    # 查询相关组件
│       │   ├── SQLQuery/             # SQL 查询编辑器
│       │   ├── JoinQuery/            # 连接查询
│       │   ├── PivotTable/           # 透视表
│       │   ├── SetOperations/        # 集合操作
│       │   ├── ResultPanel/          # 结果展示面板
│       │   ├── DataGrid/             # TanStack DataGrid
│       │   ├── DataSourcePanel/      # 数据源树形面板
│       │   ├── AsyncTasks/           # 异步任务面板
│       │   └── QueryTabs/            # 查询标签页
│       ├── DataSource/               # 数据源管理
│       ├── Layout/                   # 布局组件
│       ├── Settings/                 # 设置页面
│       ├── components/               # 通用组件
│       │   └── ui/                   # shadcn/ui 组件库
│       ├── providers/                # Context Providers
│       ├── styles/                   # 样式文件
│       │   └── tailwind.css          # Tailwind 主题变量
│       └── i18n/                     # 国际化
├── config/                           # 配置文件
├── docs/                             # 文档（索引见 docs/README.md）
│   ├── API_CONTRACT_FE_BE.md         # 前后端 API 契约
│   ├── ARCHITECTURE_CALL_MAP.md      # 分域调用图
│   └── frontend/QUERY_EXECUTION_FLOW.md
├── .github/                          # CI / PR 模板
│   └── pull_request_template.md      # PR：契约与验证勾选
└── docker-compose.yml
```

**关键文件索引**

| 文件 | 用途 |
|------|------|
| `frontend/src/api/index.ts` | API 模块统一导出（`@/api`） |
| `frontend/src/api/types.ts` | 共享类型定义（StandardSuccess, StandardError 等） |
| `frontend/src/api/client.ts` | `apiClient`、`normalizeResponse`、错误归一化 |
| `frontend/src/api/queryApi.ts` | DuckDB / 联邦执行、`cancelSyncQuery` 等 |
| `frontend/src/api/pivotQueryApi.ts` | 透视 generate/preview、SQL 收藏、应用配置 |
| `frontend/src/api/joinQueryApi.ts` | 结构化 JOIN：`performJoinQuery` |
| `frontend/src/api/setOperationsApi.ts` | 集合运算 generate / validate / execute 等 |
| `frontend/src/api/databaseSchemasApi.ts` | 外部库 schemas / 表 / 表详情 |
| `frontend/src/api/settingsShortcutsApi.ts` | 快捷键配置 API |
| `docs/API_CONTRACT_FE_BE.md` | 端点契约表（与后端路由、`normalizeResponse` 对齐） |
| `docs/ARCHITECTURE_CALL_MAP.md` | 全局分域调用图（入湖 / 查询 / 元数据 / 异步 / 透视） |
| `.github/pull_request_template.md` | PR 自检：契约 / lint / pytest |
| `frontend/src/hooks/useDuckDBTables.ts` | DuckDB 表列表 Hook |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chenkeliang/duckdb-query](https://github.com/Chenkeliang/duckdb-query) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
