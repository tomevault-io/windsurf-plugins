---
trigger: always_on
description: **Updated:** 2026-05-02
---

# PROJECT KNOWLEDGE BASE

**Updated:** 2026-05-02
**Branch:** main

## OVERVIEW
通达信(TDX) stock data importer — loads .day/.01/.5 files to DuckDB/ClickHouse, calculates preclose/turnover/market-value (basic), and 后复权因子 (hfq_factor). basic/factor 链路覆盖 stock + ETF (含 LOF/老封基/B股)。

Current schema version: **v3.0** (`model.SchemaMajor=3, SchemaMinor=0`). The `_meta` table stores the schema version; init/cron check major version compatibility at startup.

## STRUCTURE
```
./
├── calc/       # Financial calculations (basic indicators + 复权因子)
│   ├── basic.go           # preclose, turnover, floatmv, totalmv
│   ├── fq_quantaxis.go    # HFQ factor (QUANTAXIS-based)
│   └── fq_quantaxis_test.go
├── cmd/        # CLI commands (init, cron, convert)
├── database/   # DB interface + implementations (duckdb/clickhouse)
├── model/      # Data models, table registry, view registry
├── tdx/        # TDX binary format parsing
├── utils/      # Utilities (cache, pipeline, CSV, download)
├── workflow/   # Task execution framework (DAG, calendar, work plan)
└── main.go     # Cobra CLI entry point
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Add new database backend | ./database/ | Implement DataRepository interface |
| Parse new TDX format | ./tdx/ | Binary format parsers (day, minline, blocks) |
| Modify basic calculation | ./calc/basic.go | preclose / turnover / market-value |
| Modify HFQ factor | ./calc/fq_quantaxis.go | 后复权因子算法 |
| Add CLI command | ./cmd/ + main.go | Cobra subcommand with ctx cancel support |
| Data model changes | ./model/ | Schema + struct tags + table/view registry |
| Database queries | ./database/*/dml.go | DB-specific query implementations |
| Add/modify workflow task | ./workflow/tasks.go | Define task with dependencies |
| Run specific tasks | ./workflow/engine.go | Use TaskExecutor with task names |

## CODE MAP
| Symbol | Type | Location | Role |
|--------|------|----------|------|
| main | func | main.go:37 | Cobra root + ctx setup |
| DataRepository | interface | database/interface.go:9 | DB abstraction (Connect/Import/Query) |
| NewDB | func | database/factory.go:11 | Driver factory (duckdb/clickhouse) |
| Task | type | workflow/engine.go:46 | Task definition with dependencies |
| TaskExecutor | type | workflow/engine.go:66 | DAG-based task execution |
| Init | func | cmd/init.go | Full import via workflow |
| Cron | func | cmd/cron.go:11 | Incremental update via workflow |
| Convert | func | cmd/convert.go | TDX to CSV conversion |
| CalculateBasicDaily | func | calc/basic.go:115 | Core basic calculation (preclose/turnover/MV, stock+ETF) |
| calculateFullHfq | func | calc/fq_quantaxis.go:86 | Core HFQ factor calculation |
| BuildWorkPlan | func | workflow/plan.go:34 | 读 holidays + 各表最新日期，决定哪些任务要跑 |
| TradingCalendar | type | workflow/calendar.go:5 | 节假日/周末判定 + 最近交易日查找 |
| KlineDay | type | model/schema.go:14 | Raw daily OHLCV |
| BasicDaily | type | model/schema.go:49 | Calculated basic (preclose/turnover/MV, stock+ETF) |
| Factor | type | model/schema.go:41 | Adjust factor (hfq_factor) |
| GbbqData | type | model/schema.go:61 | 股本变迁 data (cat 1=除权, 11=ETF份额折算, 2/3/5/7/8/9/10=股本变动) |
| ClassifyCode | func | model/classify.go:58 | symbol → stock/index/etf/block/unknown |
| PriceScale | func | model/classify.go:89 | TDX 原始整数价格换算 (股票=100, ETF/B股=1000) |
| SchemaFromStruct | func | model/tables.go:54 | Reflect-based table registration |

## CONVENTIONS

**Database URI format:**
- ClickHouse: `clickhouse://[user[:password]@][host][:port][/database][?http_port=8123]`
- DuckDB: `duckdb://[path]`

**Table naming:**
- `raw_*` — raw imported data (raw_kline_daily, raw_kline_1min, raw_kline_5min, raw_basic_daily, raw_adjust_factor, raw_gbbq, raw_symbol_class, raw_holidays)
- `v_*` — views (v_bfq_daily, v_qfq_daily, v_hfq_daily) — 通过 `raw_symbol_class` 过滤，仅保留 stock + etf
- `_meta` — schema version metadata (key/value store)

**Table registration:**
- All tables auto-registered via `SchemaFromStruct()` init-time calls in `model/tables.go`
- Views registered via `DefineView()` in `model/views.go`
- Use `model.Table*` / `model.MetaTable` constants for table references (never hardcode table names)

**TDX file collection (cmd/common.go):**
- All .day/.01/.5 files collected by suffix, filtered only by `^(sh|sz|bj)\d+$` regex
- No prefix whitelist — full ingest of everything TDX provides
- Symbol classification via `raw_symbol_class` table (rebuilt after each daily import)

**Symbol classification (model/classify.go):**
- `ClassifyCode(symbol) → stock/index/etf/block/unknown`
- Rules match by (market, numeric prefix) — longest prefix wins
- basic/factor calculation uses `GetSymbolsByClass("stock", "etf")` — index/block 仍排除在 calc 输出之外
- 复权视图 (`v_*fq_daily`) 通过 `raw_symbol_class` 过滤，仅保留 stock + etf
- B 股 (sh900/sz20) 归为 stock；ETF/LOF 归为 etf
- `PriceScale(symbol)`：股票/指数/板块 价格单位 0.01 元(=100)，ETF/LOF/B股 0.001 元(=1000)
- `SymbolFromCode(code)`：6 位裸数字反查市场前缀（仅考虑 stock/etf）
- Class `unknown` 包括：可转债 (sh11xxxx, sz12xxxx, sz13xxxx), 国债 (sh24xxxx) 等

**Schema versioning (cmd/schema_version.go):**
- `model.SchemaMajor` / `model.SchemaMinor` define current version
- DB stores version in `_meta` table via `ReadSchemaVersion()` / `WriteSchemaVersion()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jing2uo/tdx2db](https://github.com/jing2uo/tdx2db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
