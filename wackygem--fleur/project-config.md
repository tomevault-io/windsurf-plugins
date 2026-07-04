---
trigger: always_on
description: ├── pipeline/           # Python 数据工作区，由 uv 管理
---

# AGENTS.md — fleur 项目指南

## 项目结构

```
fleur/
├── pipeline/           # Python 数据工作区，由 uv 管理
│   ├── scheduler/      # Dagster 调度项目（scheduler）
│   ├── elt/            # dbt 转换项目（elt）
│   ├── contract_tools/ # 数据契约校验与生成工具
│   ├── contracts/      # 数据契约注册表（字段事实源）
│   └── migrate/        # Alembic 数据库迁移
├── engines/            # Rust 后端和计算引擎工作区，由 Cargo 管理
├── deploy/             # 部署配置
│   ├── docker-compose.yml
│   ├── postgres/       # PostgreSQL 配置
│   └── release-manifest.yml
├── app/                # 前端应用工作区（racingline）
├── docs/               # 项目文档与计划
├── .env                # 环境变量（不提交）
└── .env.example        # 环境变量模板
```

## 文档入口

- 文档总入口：`docs/README.md`
- 多工程项目状态：`docs/architecture/project-status.md`
- 数据平台地图：`docs/architecture/data-platform.md`
- 数据治理地图：`docs/architecture/data-governance.md`
- Furnace 计算引擎地图：`docs/architecture/furnace.md`
- Rearview 后端服务地图：`docs/architecture/rearview.md`
- Racingline 前端工作台地图：`docs/architecture/racingline.md`
- 部署与运行地图：`docs/architecture/deploy-ops.md`
- 架构边界：`docs/architecture/`
- 模块边界：`docs/architecture/scheduler-module-boundaries.md`
- 长期决策：`docs/ADR/`
- 方案与历史设计：`docs/RFC/`
- 执行计划：`docs/plans/README.md`
- 技术债与质量优化：`docs/issues/`
- 运行报告：`docs/jobs/reports/`
- dbt 模型设计：`docs/architecture/dbt_layer/`
- 接口、数据字典和样例：`docs/references/`
- 项目 skills：`docs/skills/`
- Rust engines 文档地图：`engines/README.md`

## Python 与工作区

- 使用 `uv` 管理 Python 依赖和虚拟环境。
- Python 版本固定在 `3.12.13`，配置于 `pipeline/.python-version`。
- 所有 Python、dbt、Dagster 和 `dg` 命令必须在 `pipeline/` 目录下通过 `uv run` 执行。
- 同步完整工作区：

```bash
cd pipeline
uv sync --all-packages --all-groups
```

### 子项目

| 子项目 | 路径 | 包管理器 | 说明 |
|--------|------|----------|------|
| scheduler | `pipeline/scheduler/` | uv (pyproject.toml) | Dagster 调度与资产定义 |
| elt | `pipeline/elt/` | uv (pyproject.toml) | dbt 数据转换 |
| contract_tools | `pipeline/contract_tools/` | uv (pyproject.toml) | contract registry 校验与生成 |
| migrate | `pipeline/migrate/` | uv (pyproject.toml) | Alembic 数据库迁移 |

## Rust 与 engines 工作区

- Rust workspace 路径：`engines/`
- 使用 Cargo 管理 Rust crate，不放入 `pipeline/` 的 uv 工作区。
- 所有 Rust / Cargo 命令在 `engines/` 目录下执行。
- engines 文档地图：`engines/README.md`
- Furnace 设计入口：`docs/RFC/archive/0016-rust-furnace-compute-engine.md`
- 当前 crates：

| Crate | 路径 | 类型 | 说明 |
|-------|------|------|------|
| furnace | `engines/crates/furnace/` | binary | `furnace kdj` CLI 入口、参数解析、请求校验和 JSON summary 输出 |
| furnace-core | `engines/crates/furnace-core/` | library | KDJ 参数、输入/输出模型、单证券 RSV/KDJ 纯计算；不依赖 ClickHouse、Dagster、dbt、Rayon 或环境变量 |
| furnace-io | `engines/crates/furnace-io/` | library | ClickHouse DDL/SQL、`clickhouse-client` 执行、RowBinary 读写、按证券并行调度、staging/partition replace 和运行摘要 |

- Rust API 文档：

```bash
make rust-doc
make rust-doc-serve
```

### Furnace 边界

- 指标公式只放在 `furnace-core`，不要在 Python asset、dbt SQL 或 ClickHouse SQL 中重写 RSV/KDJ 递推公式。
- ClickHouse、RowBinary、Rayon 并行、staging 和分区替换逻辑放在 `furnace-io`。
- Dagster 通过 `pipeline/scheduler/src/scheduler/defs/resources/furnace.py` 调用 Rust CLI，传入运行参数并读取 JSON summary。

## Dagster（scheduler）

- 项目路径：`pipeline/scheduler/`
- 项目名称：`scheduler`
- 在 `pipeline/` 目录下使用 `uv run dg ...` 和 `uv run dagster ...`
- 优先使用 `dg` CLI 进行项目检查和脚手架操作
- Dagster 主目录：`/storage/program/fleur/.dagster`
- 架构入口：`docs/architecture/scheduler-architecture.md`
- 模块边界和禁止模式：`docs/architecture/scheduler-module-boundaries.md`
- 回填操作：`docs/skills/fleur-dagster-backfill-runbook/SKILL.md`

## dbt（elt）

- 项目路径：`pipeline/elt/`
- 项目名称：`elt`
- 在 `pipeline/` 目录下使用 `uv run dbt ...`
- 优先使用定向命令，除非明确要求，不要运行整个 dbt 项目
- 开发时优先使用 `dbt build --select ...` 而非 `dbt run`
- 初始 `models/example` 内容已移除，保留空目录结构
- dbt canonical 字段治理入口：`pipeline/elt/metadata/field_glossary.yml`
- dbt staging 清洗边界：`docs/ADR/0007-dbt-staging-cleaning-boundary.md`
- 新增或重写 staging model 前先使用 `docs/skills/fleur-dbt-model-readiness/SKILL.md`，并维护 `docs/references/raw_profile/<dataset>.md`
- 修改 staging model 后运行：`uv run dbt parse --project-dir elt --profiles-dir elt`、`uv run python elt/scripts/validate_staging_readiness.py` 和 `uv run python elt/scripts/validate_field_glossary.py`

## 数据契约（contracts）

- 字段事实源：`pipeline/contracts/datasets/*.yml`，范围到 ClickHouse raw 层为止
- 生成/校验工具：`pipeline/contract_tools/`
- dbt `sources.yml` 和 `docs/references/data_dict/*.md` 由 contract 生成或校验
- dbt `staging.yml`、`stg_*.sql`、stg 字段描述和 tests 由 `pipeline/elt` 项目维护，不写入数据契约
- 修改字段事实后运行：

```bash
cd pipeline
uv run fleur-contracts validate
uv run fleur-contracts generate --check
```

## 数据库迁移（migrate）

- 迁移路径：`pipeline/migrate/`
- 使用 Alembic 管理 PostgreSQL 表结构
- 执行迁移：

```bash
cd pipeline/migrate
uv run alembic upgrade head
```

## 前端与浏览器调试

- `racingline` 前端路径：`app/racingline/`
- 前端架构事实文档：`docs/architecture/racingline.md`
- 快速启动 Rearview + Racingline dev 环境：`make dev`；按端口清理：`make stop`；停止 Docker dev 依赖：`make down`
- Playwright CLI 使用全局安装的 `@playwright/cli`，命令为 `playwright-cli`
- 当前浏览器调试环境通过 Docker `vnc-mini-desktop` 暴露 Chromium CDP 端口，默认 `PLAYWRIGHT_CDP_ENDPOINT=http://127.0.0.1:9222`
- 使用 Playwright CLI 调试前端时，默认连接现有 CDP 浏览器：`playwright-cli attach --cdp="${PLAYWRIGHT_CDP_ENDPOINT:-http://127.0.0.1:9222}"`；不要用 `playwright-cli open` 启动本机 Chrome
- CDP 连通性检查：

```bash
node scripts/check_playwright_cdp.mjs
```

## 质量门禁

提交代码前必须通过以下检查：

文档-only 变更至少运行：

```bash
make docs-check
git diff --check
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WackyGem/Fleur](https://github.com/WackyGem/Fleur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
