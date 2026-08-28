---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

本地优先的个人工作台。第一批功能是日历与 todo，此后会持续加入深度定制的领域模块（秋招管理、社招管理等），**需求增长没有终点**。

因此本项目的首要目标不是实现某组功能，而是：**让第 10 个模块的加入成本，与第 2 个模块相同。** 所有架构选择都服务于这一条；遇到取舍时，以它为准。

当前状态：Walking Skeleton 完成，秋招模块与工作台模块已全量接入（UI 已全部迁移至
`modules/workbench` 聚合正主），系统设置支持主题、时区与工作台偏好全链路持久化落库（`app_settings` 表），
多账号体系、WebDAV 备份恢复与 Gist 设置零知识加密同步（TASK-038）已全栈贯通。
现有五个业务模块（todo、workbench、campus-recruit［界面称「招聘管理」］、habit、notes）、一层共享设计基座
（`packages/ui`：18+ 个组件 + SettingsProvider 与主题/时区/偏好三套上下文 + Apple-Style 胶囊开关 + 图标集）、以及带请求编号的错误追踪。

三次架构考试都过了，且考的是不同的东西——三格已经填满：

|                   | 有自有表 | 投影成 core Item | 代表   |
| ----------------- | -------- | ---------------- | ------ |
| 有实体、进 core   | 是       | 是               | 秋招   |
| 零自有表          | 否       | 只消费，不创建   | 工作台 |
| 有实体、不进 core | 是       | 否               | 习惯   |

- **秋招模块**：模块可以有自己的领域实体。core 只多了一个通用的 `delete(moduleId, id)`。
- **工作台模块**：模块也可以**零自有表**（`migrations: []`），纯粹是 core 之上的一个
  视图 + 一个动作。core 只多了一个通用查询维度 `unscheduled`。
- **习惯模块**：模块可以**有自有表却完全不碰 core Item**。由此得出一条供后续模块直接
  套用的判据（ADR-0023）：**要不要投影成 core Item，取决于它是否需要 core 提供的跨模块
  能力**（排程、日历、今日聚合、优先级）。需要就投影，不需要就不投影；
  「它看起来像一件事」不是理由。core 一行未改。

**todo 已不再是零自有表模块。** 2026-08 加入子任务 / 标签 / 重复任务，它长出了四张
自有表与一份迁移，模块定义随之从常量导出改为接收 Repository 的工厂函数（与秋招同形）。
三条铁律仍未破：core 一行未改。理由与全部取舍见 `docs/adr/0025`（该 ADR 原编号 0014，
2026-08-22 因与时区那份撞号而改编）。

**一处仍在的不对称，动 todo 前必须知道：`GET /api/todo/today` 不按 `sourceModule`
过滤**，秋招的事项也会出现在它的结果里；而所有写操作（完成、编辑、回收站、子任务、
标签）只认 `sourceModule === 'todo'` 的项。这个端点已无消费者、待退休，但只要它还在，
这条不对称就还在。

`modules/workbench` 的 UI 搬迁**已经完成**——`packages/web/src/modules.ts` 现在只注册
`workbenchUiModule` 与 `campusRecruitUiModule`，`modules/todo/src/ui/` 已不再挂载
（1380 行的 `TodayPage.tsx` 就此成为死代码）。两个 `today` 端点仍并存：

| 端点                       | 状态                                                 |
| -------------------------- | ---------------------------------------------------- |
| `GET /api/workbench/today` | 正主。跨模块聚合，带 `scheduled` 两分支形状与 `kind` |
| `GET /api/todo/today`      | 待退休。已无消费者，随 itemActions 那一轮一并删除    |

**注意：`modules/todo/src/ui/TodayPage.tsx` 虽已不再挂载，但仍在被改动**
（`1d16a57 时钟组件` 同时改了两份 TodayPage）。**删它之前必须先与对方对齐**，
不要因为「它是死代码」就单方面删。

**不要再往 todo 里加跨模块能力。** 跨模块视图调用源模块写操作的正确机制是 core 的
`itemActions` 能力槽，方案见
`docs/superpowers/specs/2026-08-18-item-actions-registry-design.md`。
`modules/workbench/src/ui/api.ts` 里那 11 条硬编码的 `/api/todo/...` 是待还的债，
文件顶部有 TODO 标注，并已由 lint 规则封住新增（见下）。

## 命令

| 命令                           | 用途                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------- |
| `npm run setup`                | **克隆后第一条命令**。装依赖时跳过多余的原生编译，再装回 git 钩子。别用 `npm install` |
| `npm run dev`                  | 同时启动后端（:3000）与前端（:5173）。Vite 代理 `/api` 到后端，浏览器只见一个源       |
| `npm run check`                | 提交前跑这个：format:check → typecheck → lint → test，四步全绿才算过                  |
| `npm run test`                 | 只跑测试                                                                              |
| `npx vitest run <路径>`        | 跑单个测试文件，例如 `npx vitest run packages/core/src/time.test.ts`                  |
| `npx vitest run -t "<用例名>"` | 按用例名筛选                                                                          |
| `npm run db:generate`          | 改完 `packages/data/src/schema.ts` 后生成迁移                                         |

本地数据在 `data/local/accounts/<账号 id>/workbench.db`（已 gitignore），默认账号是
`local-default`。删掉整个 `data/local/` 即可从空库重来。账号根目录由 `WORKBENCH_DATA_DIR`
决定（默认 `./data/local`）；`WORKBENCH_DB` 保留为**逃生舱**——显式设置时锁定单库、
跳过账号机制与一次性迁移，供 CI 与测试用。

完整数据目录布局：

```
data/local/
  accounts.json                     引导文件，原子写，可手工修
  accounts/
    local-default/workbench.db      主库文件（WAL 模式）
    <其他账号>/workbench.db
  .restore/                         恢复中工作目录
    state.json                      恢复状态，断电续命用
    incoming.db                     已下载解压的候选库
    rollback.db                     恢复前的本地快照 = 回退点
  credentials.json                  OS 保管库不可用时的退化明文存储
  server.log
```

启动时若 `accounts.json` 不存在而旧的 `data/local/workbench.db` 还在，会做一次性迁移：
**先正常打开旧库再 close 让 WAL checkpoint 掉**，然后只 rename 一个主库文件。顺序是
承重的——这样不必同时处理 `-wal`/`-shm`，也不会搬出半截状态。

**装依赖只走 `npm run setup`，不要直接 `npm install`。** 后者在没有 MSVC 工具链的机器上必挂：
`better-sqlite3` 带 `binding.gyp` 又没有 `install` 脚本，npm 于是默认触发 `node-gyp rebuild`。
**这次编译从头到尾是白跑的**——该包自带 `prebuilds/`，是 N-API 二进制（`NAPI_VERSION=10`），
跨 Node 版本与平台通用，运行时加载的一直是它，`build/Release` 目录压根不存在。
`setup` 用 `--ignore-scripts` 跳过编译，再单独跑 `npx husky` 把被一并跳过的 hook 装回来。

Node 要求 **≥ 22.22.1**，这不是随便一个 22：`react-router@8` 要 `>=22.22.0`，
`lint-staged@17` 要 `>=22.22.1`。根 `engines` 已与之对齐，配合 `.npmrc` 的
`engine-strict=true`，版本不够会在**根包**就报 EBADENGINE，而不是甩出一个指向第三方包的
误导性错误。

pre-commit hook 只跑 Prettier（lint-staged），**不跑测试**——测试是 CI 的职责，在 commit 时跑会抑制提交频率。

## 架构

### 分层与依赖方向

```
packages/core     纯领域逻辑，零 IO 依赖，不知道任何模块存在
packages/data     SQLite + Drizzle + 迁移 + 仓储实现 + 连接持有层 + 凭据保管
packages/sync     WebDAV 备份恢复、账号/Device Flow 契约、Gist 设置同步与加密（/contract 与 /node）
packages/server   Fastify，装配 core + data + sync + 已注册模块
packages/ui       共享设计基座与壳层 Context，依赖 @workbench/core

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ethan-Hang/Personal-WorkBench](https://github.com/Ethan-Hang/Personal-WorkBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
