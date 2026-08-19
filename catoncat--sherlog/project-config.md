---
trigger: always_on
description: validates the live repo against that snapshot — HEAD drift, branch
---

# Sherlog Agent Guide

## 项目定位

`Sherlog`（CLI 命令 `shlog`）是一个面向本机 agent session 日志的渐进式检索 CLI。生产 runtime 已切换为 standalone Rust binary，内置 SQLite/FTS5；使用者运行 CLI 不需要 Node.js。当前公开 source 包括 `codex`、experimental `claude-code`、experimental `pi` 和 experimental `dsh`。

当前接受的产品边界：

- 命令面固定为：`status`、`sync`、`cold`、`find`、`read-range`、`read-page`、`list`、`stats`
- 主工作流固定为：先按问题选择 metadata projection / semantic recall / content read / coverage diagnosis；首次安装可用 `sync` 初始化默认 Codex index，coverage 不足时才做同范围 sync
- `sync` 是唯一 content/index writer；`cold add/remove` 只写 cold retention state；其余命令只读
- `find` / `read-*` / `list` / `stats` 只读 SQLite index，不扫描 raw transcript；`status` 不返回/检索正文且不写 index，但 inventory cache miss 会流式读取 raw，仅按 privacy allowlist 派生 inventory/fingerprint
- 默认接受手动增量同步，不做 watcher / daemon / realtime sync
- 这个仓库可以作为其他工具或 GUI 的 retrieval engine，但本仓库自身不以 GUI 为目标

## 当前实现真相

- 当前生产代码在仓库根 crate：`src/` 是 Rust，生成单个 `shlog` binary。`eval/` 是 TypeScript 评测 harness，只 fork `shlog`，不再保留第二套 TypeScript CLI。
- v8 SQLite 是唯一持久化真相源：`meta`、物理表 `session_rows`、只读兼容 view `sessions`、`source_files`、统一 `documents`、contentless `documents_fts`、`coverage`、`cold_roots`。没有 metadata sidecar。
- v8 writer 固定使用 rollback `journal_mode=DELETE` 与 `synchronous=FULL`。这是短命 CLI + 显式 single-writer 的有意识取舍：发布态 index 保持单文件，所有只读命令在 DB `0444`、目录 `0555` 时也不会创建 `-wal` / `-shm`；不要改成 `immutable=1` 或 close-time WAL seal，它们在并发 writer/reader 下会读旧数据或留下转换竞态。
- `sessions` view 故意没有 `INSTEAD OF` trigger：高级只读 SQL 保持兼容，旧 TypeScript writer 对 v8 写入时会 fail closed。
- 检索主链是 `SQLite candidate recall -> deterministic session ranking -> evidenceRead -> read-range/read-page`。真实 message 与 session profile 是两类 document；profile 命中不得伪装成 message evidence。`read-range --query` 无 message anchor 返回 typed `anchor_not_found`（含 `matchedProfileFields` 与闭包 read-page nextAction），不回退 seq 0；read payload 的 session 记录含 `compactText`/`reasoningSummaryText`。
- 查询面单分支：find/read/list 等内容命令只讲 v8；legacy v7 是 import 格式。v7 库上内容命令返回 typed `index_schema_upgrade_required` + nextAction `shlog sync`；`status` 正常工作并报告 `index.layout: legacy_v7`（升级 nudge）。迁移只发生在显式 writer 命令，副作用是 coverage 清空（需重新 sync 各 root）、保留 `*.v7.bak.*` 备份、legacy cold-roots.json tombstone、旧 TS writer fail-closed。
- `find` 的 `evidenceRead.command` 是 `executable:"inherit"` + 闭包 `--source/--db/--json` 的 `args` + `sideEffect:"read_index"`，custom DB 下原样执行必须读回原 candidate。无 `--root/--cwd/--selector` 的 find 解析为各 source 的 canonical default `all(root)`，recall/coverage/scanned 三 scope 一致。
- 有确切 source 或显式 `--root/--cwd/--selector` 的内容命令，`index_unavailable.nextAction.commands[].command` 闭包该 scope 的 sync（多 source 时每 source 一条，Codex 为 recommended），标记 `sideEffect:"write_index"`；兼容 `argv` 只作镜像。无显式 scope 的跨 source find 仍保留默认 Codex `all`+`cwd` alternatives。
- v8 tokenizer 使用 UAX #29 lowercase word 与重叠 CJK Unicode-scalar bigram。FTS column 权重为 body 1.0、title 8.0、summary 3.0、compact 4.0、reasoning summary 1.2。
- `source` / root / cwd / date / session / exclude 约束尽量在 SQL candidate generation 阶段下推，不先召回大集合再在 app 层过滤。
- 任何新增的快速候选层都必须返回 conservative superset：只能排除可证明不匹配的记录，tokenizer、delta 或 source 状态不确定时必须保守纳入并交给精确层；不得制造 false negative。
- `source_files` 保存 append cursor、digest、checkpoint 与 epoch。可证明 append 走增量 projection；truncate、prefix rewrite、identity/epoch 不安全时走 full replay。设计验收不变量是 `incremental projection == full replay projection`。
- `status` 返回 execution context、source inventory、index 状态与可选 requested coverage；它不返回或检索 raw content、不写 index。inventory cache miss 会流式解析 raw accepted records/body，仅按 privacy allowlist 派生 cwd/time/session identity 与 fingerprint；rejected/private record 不影响 proof。exact `mtime_ns`/checkpoint cache hit 不重 parse。
- coverage 只由成功的 `sync` 写入。裸 `sync` 是默认 Codex `all(root)` bootstrap；只读命令不会隐式 sync 或 migrate。
- strict sync 遇到选中输入错误时不发布部分 coverage；`--best-effort` 可提交成功文件，但不会伪造 complete coverage。`--prune` 只删除 hot 与 registered cold 都不存在的同 source 投影。
- v8 `cold_roots` 表是 cold registration 真相。legacy `cold-roots.json` 只作为首次 v7/v8 cutover 的一次性导入输入；导入后使用 tombstone 阻止旧 writer 复活配置。
- v7 -> v8 migration 只发生在授权 writer 路径，采用 copy/verify/backup/atomic publish；read-only commands 不迁移。cold-only projection 必须保留。
- native release pipeline 声明 `aarch64-apple-darwin`、`x86_64-apple-darwin`、`x86_64-unknown-linux-gnu` 三个目标。tag 只发 GitHub Release；Homebrew tap 与 sherlog.net 都要另走一步，步骤见「发布 / 更新闭环」。

不要把下面这些说成已完成：

- 真正独立的 stage-2 / resource-level reranker
- richer projection / range cache / event-level replay
- duplicate family collapse / diversity control
- 强约束的 gold set / rubric / error taxonomy
- 完整的 incremental/full-replay property/state-machine test 矩阵；当前只有聚焦 transition/migration tests
- candidate/filter/exact 各阶段完整可观测性与 `weakMatch`/`matchMode` 公共 contract
- 全正文 typo-tolerant fuzzy、evidence-read frecency
- Linux arm64、musl 或 Windows native archive
- watcher / daemon、LMDB 或第二状态真相源

## 代码地图

- [main.rs](src/main.rs): native binary entrypoint
- [cli.rs](src/cli.rs): fixed CLI surface and flags
- [app/](src/app): command orchestration, output and status
- [runner.rs](src/runner.rs): parse/dispatch/error routing
- [sources/](src/sources): source adapters, inventory and privacy projection
- [sync/](src/sync): lock, scan/project/stage, append/full transitions, cold retention and publish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catoncat/sherlog](https://github.com/catoncat/sherlog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
