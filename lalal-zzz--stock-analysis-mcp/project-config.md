---
trigger: always_on
description: - **Dual-runtime**: `index.js` (Node shim, ESM) spawns `python -m stock_analysis_mcp.server` via stdio and proxies I/O. The Python server is the real MCP implementation. Python interpreter resolution: `STOCK_ANALYSIS_PYTHON` env → `~/.stock-analysis/runtime.json` (uv-managed venv) → `python`.
---

# AGENTS.md

## Architecture

- **Dual-runtime**: `index.js` (Node shim, ESM) spawns `python -m stock_analysis_mcp.server` via stdio and proxies I/O. The Python server is the real MCP implementation. Python interpreter resolution: `STOCK_ANALYSIS_PYTHON` env → `~/.stock-analysis/runtime.json` (uv-managed venv) → `python`.
- **Entrypoint**: `src/stock_analysis_mcp/server.py` — uses `mcp.server.stdio` and a `@register(name, desc, schema)` decorator to wire exactly 20 tools to MCP handlers; every result is wrapped in a `{data, meta, warnings, error}` envelope. The dispatcher validates required/unknown fields, basic JSON types, enum values, and numeric ranges, and promotes handler warnings to the outer envelope.
- **Node CLI**: `bin/stock-analysis.js` + `lib/` — installer commands `install` / `setup` / `doctor` / `uninstall` / `config show`. `lib/installer.js` creates a `uv` venv under `~/.stock-analysis/runtime/` and writes `config.toml` / `runtime.json` / `install-state.json`; `lib/adapters.js` auto-configures 5 agents with backups — Claude Code (`~/.claude.json`), Codex (`~/.codex/config.toml`), Cursor (`~/.cursor/mcp.json`), VS Code Copilot (user `mcp.json`, `servers` key + `type: stdio`), Qoder (`~/.qoder/mcp.json`) — and copies the skills from root `skills/` to skill-aware agents (Claude Code / Codex / Qoder); skill ids = `skills/` directory names, discovered dynamically (no mapping table); `lib/paths.js` centralizes `~/.stock-analysis` paths.
- **Source layout**:
  - `core/config.py` — settings resolution: env vars → `~/.stock-analysis/config.toml` → defaults (`get_settings()`)
  - `core/constants.py` — schema/indicator/pattern engine versions, research bar and coverage defaults
  - `core/registry.py` — provider/feature registries reserved for future extensions
  - `data/network.py` — HTTP client (curl_cffi) + Edge cookie extraction + symbol normalization + K-line host rotation (`try_kline_hosts`)
  - `data/indicators.py` — technical indicator calcs (MA/RSI/MACD/BOLL/KDJ/ATR)
  - `data/storage/` — SQLite storage engine (stock database + sector database) with WAL mode, decoupled read-lock concurrency, and column padding defense
  - `data/sync.py` — full init download + incremental daily update coordinator (stale-symbol selection, 10-calendar-day overlap, merged-local indicator recalculation, semaphore-pipelined async concurrency; `init_all_data(quick=True)` = fast mode)
  - `data/search.py` — local DB queries with fallback to network APIs
  - `charting.py` — K线图渲染 (包顶层呈现层, 非数据层; 日K→周K/月K 重采样, 蜡烛图+MA+成交量 PNG, 供 chart-trend skill 与 `render_stock_charts` MCP 工具看图分析; 依赖 `chart` extra 的 matplotlib, 延迟导入)
  - `data/progress.py` — progress-bar shim: tqdm → null (stderr-only, auto-silent on non-TTY)
  - `data/sources.py` — extended data sources (移植自旧“股票信息”项目): `fetch_full_spot` (push2 clist 三镜像分页), `fetch_kline_history` (腾讯 fqkline 日期分段翻页 + 全局限速, 回退 akshare/搜狐), `fetch_guba_rank_history` (股吧年文件 AES-CBC 解密, key=md5("getUtilsFromFile")), `fetch_xuangu_rankings` (dataapi/xuangu 分页), `is_trade_day`/`previous_trade_day` (新浪交易日历缓存)
  - `data/build/` — 重建/回填/采集/清理: `rebuild_full_data` (步骤化全量重建 + rebuild_progress 断点续传 + `--dry-run` 预览不联网), `backfill_data` (缺口检测补齐), `daily_capture` (晚间采集: xuangu 排名 + spot 快照 + K 线增量 + 指标缓存, 跳过非交易日), `cleanup_database` (冗余表清理 + VACUUM), 板块全量 K 线下载 + 板块指标缓存
  - `tools/stock_data.py` — stock list, history, indicators, search, multi-period K-line (`get_stock_kline_period`: klt 1/5/15/30/60/101/102/103)
  - `tools/stock_rank.py` — popularity rankings (gainers/volume/turnover)
  - `tools/sector_data.py` — sector list, members, K-line (`get_sector_kline_net` accepts `klt`)
  - `tools/pattern_scan.py` — technical pattern screening
  - `tools/sector_screen.py` — sector screening + capital flow analysis
  - `tools/data_manager.py` — local data management MCP tools (init/update/search/sector→stocks)
  - `tools/analysis.py` — individual stock technical analysis reports (support/resistance/risk/position)
  - `tools/research.py` — top-20 rising-candidate ranking and per-stock monthly/weekly/daily evidence packets
  - `strategies/patterns.py` — 形态引擎: 波段/Pivot、关键位、因子列和5类底层检测器 (trend_pullback/ma_rebound/w_bottom/m_neckline/box_breakout); 高层将旧名规范为 major_ma_rebound/neckline_reclaim，并把 Fibonacci confluence 作为第6类增强证据; **双宇宙抽象**支持 stocks/sectors, MIN_BARS=260, pivot 右确认避免未来函数
  - `strategies/pattern_backtest.py` — 形态回测 CLI (6 份报告: 形态×变体胜率/关键位分层/wave_phase 分层/因子分层/过滤阈值搜索/分年度胜率), `--universe stocks|sectors`
  - `strategies/pattern_optimize.py` — beam search 多因子规则搜索 + train/test 时间切分防过拟合, 尝试记录 markdown 输出
  - `strategies/trading_backtest.py` — callable 事件研究 + 次日开盘/止损/2R/20日退出交易回测，输出 Markdown/CSV 且不自动改规则
  - `strategies/similarity.py` — 价格成交量跨股票/跨周期相似形态引擎: 固定长度归一化、DTW价格路径、回撤/振幅/量能/Pivot转向评分及历史后验统计；全市场候选复用本地日K并重采样
  - `cli.py` — 统一 CLI 入口 `python -m stock_analysis_mcp.cli`: rebuild / backfill / daily-capture / cleanup / pattern-scan / pattern-backtest / pattern-optimize, 通用 `--data-dir` (等价 STOCK_ANALYSIS_DATA_DIR) 与 `--dry-run`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lalal-zzz/stock-analysis-mcp](https://github.com/lalal-zzz/stock-analysis-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
