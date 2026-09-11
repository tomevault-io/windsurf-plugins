---
trigger: always_on
description: opencode-supertask — 面向 OpenCode Agent 的 SQLite 任务队列与调度器。发布形态包括 OpenCode 插件、`supertask` CLI，以及可前台运行或由 pm2 托管的单实例 Gateway；Gateway 内部同时运行 Worker、Scheduler、Watchdog 和 Web Dashboard。
---

# AGENTS.md

## 项目概述

opencode-supertask — 面向 OpenCode Agent 的 SQLite 任务队列与调度器。发布形态包括 OpenCode 插件、`supertask` CLI，以及可前台运行或由 pm2 托管的单实例 Gateway；Gateway 内部同时运行 Worker、Scheduler、Watchdog 和 Web Dashboard。

## 系统链路

```text
OpenCode 插件 / CLI / Dashboard
              ↓
TaskService / TaskRunService / TaskTemplateService
              ↓
SQLite（tasks / task_runs / task_templates）
              ↑
Gateway ─ Worker + Scheduler + Watchdog + Dashboard
```

- 插件在 `plugin/supertask.ts` 注册 8 个 `supertask_*` 工具：`add/next/status/retry/list/get/schedule/upgrade`；运行态与执行终态只允许 Gateway 写入，不得恢复外部 `start/done/fail`。
- Worker 先启动等待握手的 launcher，持久化 launcher PID 后才通过参数数组执行 `opencode run --agent <task.agent> --format json [-m <model>] [--variant <variant>] <task.prompt>`；新 run 使用 `gated-v3-token-guardian`，每 run UUID 必须同时写入 `task_runs.locked_by` 和 launcher argv。launcher 只能在整个受管进程组排空后通过不传递给 OpenCode 的 IPC 发回绑定 UUID 的证明；无证明退出必须隔离，不得结算或释放批次。退出码决定成功或失败，Gateway 统一写任务状态和执行记录。Unix 独立进程组只能证明仍属于该组的进程排空，不得描述为任意后代的整树退出证明；主动调用 `setsid()` 或以 detached/daemon 方式离组的进程必须自行管理。Windows 在引入 Job Object 前必须拒绝启动 Worker，不得退回不完整的父子 PID 扫描。
- Worker 校验 drain proof 后必须通过同一 IPC 回送绑定 UUID 的确认，launcher 收件后才退出；不得依赖旧 Bun 不可靠的 `process.send` callback。最低支持 Bun 1.1.45，CI 必须用该版本真实执行构建后的 launcher IPC smoke test。
- 进程组排空后的结算失败不得立即释放内存所有权或丢失已知退出结果；Gateway 存活时必须保持任务、批次和心跳并重试，停机时也必须用完整 shutdown grace 继续结算，只有宽限期耗尽后才可停止持有并交给 Watchdog。
- Worker 启动的受管 OpenCode 进程设置 `SUPERTASK_MANAGED_RUN=1`；该上下文必须拒绝 `supertask_upgrade`，避免升级流程删除并等待承载自己的 Gateway。升级只能从外部 CLI 或非队列 OpenCode 会话发起。
- pm2 是可选守护层：仅显式运行 `supertask install` 时允许安装；插件加载不得静默安装全局依赖。前台运行使用 `supertask gateway`。

## 技术栈

- Bun runtime + TypeScript (strict)
- Drizzle ORM + SQLite (bun:sqlite)
- Hono (Web Dashboard SSR)
- Commander (CLI)
- pm2 (可选的 Gateway 守护进程)
- bun:test (测试框架)

## 常用命令

```bash
bun install           # 安装依赖
bun test              # 运行所有测试
bun run test:coverage # 测试并检查覆盖率基线
bun run test:browser  # 真实 Chromium Dashboard smoke
bun run build         # 构建 (tsup)
bun run typecheck     # TypeScript 类型检查
bun run typecheck:tests # 测试代码类型检查
bun run lint          # ESLint
bun run package:smoke # npm pack 后隔离安装验证
bun run dev           # CLI 开发模式
bun run gateway       # 启动 Gateway
bun run ui            # 单独启动 Web Dashboard
bun run db:generate   # 根据 Schema 生成 Drizzle migration
bun run db:migrate    # 手动运行数据库迁移
bun run dev -- db check  # 检查数据库完整性与业务统计
```

## 运行时数据

- 数据库默认位于 `~/.local/share/opencode/tasks.db`；测试或隔离运行通过 `SUPERTASK_DB_PATH` 覆盖。
- 配置文件位于 `~/.config/opencode/supertask.json`，默认值在 `src/gateway/config.ts`。
- 数据库初始化时启用 WAL、创建 `gateway_lock` 并自动执行 `drizzle/` migrations。
- 从 `0005` 起 migration 必须遵循 expand/contract 并保持 N-1 二进制兼容：只允许新增表、非唯一索引，以及可空或带默认值的新增列。删除、重命名、收紧约束和数据改写必须延后到旧版本不再是自动回滚目标后执行；测试会拒绝破坏该约束的 SQL。
- Gateway 用 SQLite `BEGIN IMMEDIATE` + `gateway_lock` 保证单实例；进程身份必须同时识别直接 Gateway 入口与公开的 `supertask gateway`/CLI 入口。Dashboard 默认只监听 `127.0.0.1:4680`。
- Gateway 必须先完成恢复收敛、Scheduler 初始化和 Dashboard 绑定，最后才启动 Worker，并把包版本与 `gateway_lock.ready_at` 一起写入；PM2 `online` 不能单独作为就绪依据，进程 PID、版本和运行作用域必须匹配新鲜 ready 锁。
- PM2 替换已有 Gateway 前必须先用保存的运行环境验证管理命令可执行，并用目标 Gateway 环境真实执行 `opencode --version`；若 OpenCode 不可执行，或新旧进程无法共用同一可回滚的 PM2 管理路径，必须在删除旧进程前失败关闭。
- PM2 替换、数据库维护、卸载与 macOS supervisor 检查必须先共用 `PM2_HOME/supertask-gateway.manage.sqlite` canonical SQLite 事务锁；兼容旧 custom lock 时还必须从 PM2 dump/运行环境和 LaunchAgent 恢复全部旧路径，并按固定顺序同时持有，不能因后续 CLI 缺少旧环境变量而绕过旧 supervisor。已安装 LaunchAgent 的 `PM2_HOME` 与当前 CLI 不同必须在任何修改前失败关闭，避免两个 PM2 daemon 争用同一 Gateway。不得恢复 PID/stale 文件锁。PM2 kill timeout 不得低于 Worker shutdown grace 加 15 秒，`stop/delete` 命令 timeout 不得低于实际 kill timeout 加 5 秒，管理锁必须持有到命令返回；显式低值必须在删除旧进程前失败关闭。
- macOS supervisor 只有在 `jlist` 成功且确认 Gateway 缺失、同时 `dump.pm2` 明确包含 Gateway 时才可 `resurrect`；状态未知、`errored`、`stopped` 和卸载后的空 dump 都不得触发重启。卸载必须停止并移除项目 LaunchAgent。
- `/health` 必须分别反映 Worker、Scheduler、Watchdog 和历史清理的活跃度与连续失败；`supertask doctor` 要分别验证当前终端和 PM2 保存的 Gateway 环境中的 OpenCode，再解析最终配置，要求唯一的精确插件版本，核对对应缓存、全局 CLI、PM2 实际 Gateway 入口包和 ready 锁版本，并验证 macOS LaunchAgent 与 PM2 dump 可恢复性。`doctor --smoke` 必须经真实数据库队列和 Gateway 执行 OpenCode、验证输出标记，普通 `doctor` 不得调用模型。浮动 `@latest`/`@next` 入口或任一组件版本不一致必须失败。`supertask upgrade` 在当前包、有效插件、缓存、CLI 和就绪 Gateway 已全部匹配 npm `latest` 时必须无副作用返回；`upgrade --force` 才允许在同版本下重新安装并重启。升级成功替换插件和 Gateway 后必须检测全局 CLI 的 npm/Bun 安装来源并同步精确版本；无法确认时返回明确的部分失败和人工命令。PM2 自动替换/恢复必须保留既有 Bun 路径、完整运行环境和数据库作用域。用户显式执行 `install`、版本变化的 `upgrade` 或 `upgrade --force` 时，目标 Gateway 可从当前终端刷新 OpenCode、XDG 与 Provider 执行环境，但必须固定旧 `HOME`、`PATH`、`PM2_HOME`、全部 `SUPERTASK_*`、Bun 路径、cwd 和数据库/配置作用域；失败回滚必须使用未修改的完整旧环境。
- 数据库检查、备份、清空和恢复统一经过 `DatabaseMaintenanceService`；CLI 清空/恢复必须显式确认并拒绝运行中任务，且只可自动停启 PID 与当前数据库新鲜 ready 锁一致的 PM2 Gateway；前台或无法确认归属的进程必须拒绝误杀。清空/恢复前必须自动创建校验通过的安全备份；清空必须动态删除全部业务表数据（包括 N+1 expand-only 表），通过延迟外键检查支持循环依赖，并保留 `gateway_lock` 与 migration 元数据。恢复来源必须从已打开的 SQLite 连接生成包含已提交 WAL 页的一致快照，并拒绝当前数据库的符号链接/硬链接别名。恢复必须动态校验 source/live 业务表和可写列：source-only 未知表/列在删除前失败关闭，共有未来列完整复制，live-only 列只允许可空/默认值且 live-only 新表必须清空，避免 N/N-1 形成混合时间点；随后在当前连接的排他事务内原位替换，不得关闭连接后 rename 换库。默认在操作失败时也恢复原 Gateway 状态，`--keep-stopped` 除外。
- Dashboard 清空只能豁免当前 Gateway PID，仍必须服务端确认、拒绝运行中任务并在同一事务内先备份后动态删除全部业务表；不得恢复为路由内直接 `DELETE` 的实现。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vbgate/opencode-supertask](https://github.com/vbgate/opencode-supertask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
