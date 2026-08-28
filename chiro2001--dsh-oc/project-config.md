---
trigger: always_on
description: 本文件面向接手本仓库的 AI Agent 与开发者，替代 README 中的开发章节。
---

# AGENTS.md — dsh-oc 开发/Agent 指南

本文件面向接手本仓库的 AI Agent 与开发者，替代 README 中的开发章节。
使用者文档见 [README.md](README.md)；人类贡献流程见
[CONTRIBUTING.md](CONTRIBUTING.md)。

## 项目是什么

dsh-oc 是 DeepSeek Harness（dsh）的 OpenCode TUI 前端：

- **前端**：官方 opencode CLI（`attach` 模式），只负责渲染、键盘、终端生命周期。
- **后端**：dsh 负责 Agent、Session、工具、模型、权限、提问。
- **桥接**：dsh-oc 在 dsh 进程内提供 OpenCode 兼容的 HTTP/SSE 服务（`oc-bridge`），
  并启动官方 TUI 子进程（`oc-tui`）。

```text
dsh (Node) ── dsh-oc bundle ── oc-bridge (HTTP/SSE) <── opencode TUI (attach)
                  │
                  └─ DSH Agent/Session/Tools/LLM/Approval/Questions
```

仓库：`chiro2001/dsh-oc`；npm 包名 `@chiro2001/dsh-oc@0.1.0-rc.1`（未发布
registry，安装/更新走 GitHub 源 `#main` / `#develop`）。

## 代码结构

```text
src/
  bridge/            # OpenCode 协议桥
    router.ts        # 核心：SSE、缓存/预取、命令/权限 helper、桥服务接线
    routes/          # 路由注册（按域拆分，聚合于 routes.ts）
      boot.ts        # v1/v2 启动与目录路由（/path /project /config /provider /agent ...）
      session-v1.ts  # v1 会话路由（列表、消息、prompt、abort、command、todo、diff）
      session-v2.ts  # v2 会话路由
      permission.ts  # permission / question 路由
    routes.ts        # 聚合器（注册顺序即匹配顺序，勿乱）
      vcs.ts         # 真实 git 信息/状态/diff 路由
      fs.ts          # 工作区文件读取/列表/查找路由
    events.ts        # SSE 事件翻译（turn.*、message.*、session.*、工具流、权限）
    state.ts         # 桥内存状态（缓存、标题、preset、活动标记）
    convert/         # dsh 事件 → opencode 消息/会话/模型/权限转换
    http.ts sse.ts rpc.ts errors.ts stubs.ts git.ts fs.ts
  tui/               # opencode 子进程解析/下载/spawn、信号转发、退出处理
  help.ts            # --help / /help 静态能力摘要
tui-branding/        # DSH OC 品牌 logo TUI 插件
scripts/             # 自测/压测/能力矩阵/清理工具
tests/               # vitest 单测 + e2e（scripts/e2e-*）
docs/                # FEATURES / PROTOCOL / ROADMAP / CHANGELOG / perf
lib/                 # 构建产物（必须随提交推送，GitHub 直装依赖它）
```

## 常用命令

```bash
pnpm install
pnpm build                 # tsdown → lib/
pnpm typecheck             # tsc --noEmit
pnpm test                  # vitest
pnpm run probe             # opencode 1.18.18 协议路由探针（62/62）
pnpm run e2e               # 全量 e2e（真实 opencode TUI，稳定套件约 15–20 分钟）
pnpm run e2e:api           # API e2e 子集（快速回归）
pnpm run perf              # 会话性能压测（--sessions N --scale ...）
pnpm run features:update   # 刷新 docs/FEATURES.md 自动追踪
bash scripts/check-all.sh --e2e          # 一键全量门槛
bash scripts/check-all.sh --e2e --scale 5000
bash scripts/verify-release-artifacts.sh # 发布工件审计（check-all 内置：lib 零差异、pack 无绝对路径、产物哈希）
bash scripts/e2e-install-rollback.sh     # manual：远端 full SHA 冷装 + TUI smoke + 回滚演练
bash scripts/cleanup-e2e-runs.sh --keep 20 --apply   # 清理 .e2e 旧 run（默认 dry-run）
bash scripts/replay-session-audit.sh <session.jsonl[.zstd]>
                                          # 真实会话回放审计：桥翻译无未处理事件/错误
bash scripts/audit-local-sessions.sh [sessions-dir]
                                          # 批量审计 $DSH_HOME/sessions 全部会话（含消息 id/role 冲突检查）
bash scripts/e2e-recovery-crash.sh        # 故障域：SIGKILL 中途崩溃后 --session 重启恢复
bash scripts/e2e-recovery-sse-reconnect.sh # 故障域：观察者 SSE 断流重连后 exactly-once
node scripts/build-replay-corpus.mjs      # 重新生成 tests/fixtures/replay 合成语料（结构保持，无真实内容）
node scripts/replay-corpus-manifest.mjs   # 扫描真实会话的 feature 覆盖，输出语料覆盖差集（无内容外泄）
bash scripts/e2e-golden-trace.sh          # 冻结/校验 1.18.18 黄金 SSE 轨迹（DSH_OC_GOLDEN_OVERWRITE=1 刷新）
bash scripts/e2e-queued-order-repro.sh    # 工具+排队+后续文本的官方 TUI 顺序复现记录
bash scripts/upgrade-lane.sh --bin <candidate>
                                          # 候选 opencode 升级 lane：黄金轨迹语义差分
bash scripts/update-local-install.sh [ref]
                                          # 更新本地 dsh profile 安装并校验 resolved commit/version
bash scripts/e2e-real-queued-order.sh     # manual：真实模型排队错序 wire/面板证据
bash scripts/e2e-minimal-server-repro.sh  # 官方最小 server 归因：脚本化事件 → 官方 TUI 渲染顺序
 ```

本地直连 dsh profile（实时验证）：`dsh plugin --profile oc add .`；改代码后
`pnpm build` 立即生效（link 方式）。从 GitHub 分支安装验证：
`dsh plugin --profile oc add 'github:chiro2001/dsh-oc#<branch>'`。`#develop`
是可变 ref，pnpm 可能复用旧解析 SHA；重复执行 add 会刷新锁文件，用
`rg 'codeload.github.com/chiro2001/dsh-oc/tar.gz' pnpm-lock.yaml` 核对实际
解析到的 commit。

## 安装、更新与本地开发

使用者只关心安装命令时见 README；Agent/开发者用以下方式验证与更新：

GitHub 源安装/更新（与 README 同款命令，重复执行即更新到该分支最新）：

```bash
dsh plugin --profile oc add chiro2001/dsh-oc                       # #main
dsh plugin --profile oc add 'github:chiro2001/dsh-oc#develop'      # 指定分支
dsh --profile oc --help                                            # 验证版本
```

- `bin/dsh-oc.mjs` 提供 `dsh-oc` 简写（`dsh --profile oc` 参数透传、退出码
  透传）；安装后位于 profile 的 `node_modules/.bin`，README 提供 PATH 指引；
  本机可直接 `ln -sf ~/.dsh/profiles/oc/node_modules/.bin/dsh-oc
  ~/.local/bin/dsh-oc` 让简写常驻 PATH。
- npm 包名 `@chiro2001/dsh-oc` 未发布 registry；安装/更新一律走 GitHub 源。
- 本地开发：`dsh plugin --profile oc add .` 以 `link:` 方式链接仓库，改
  `src/` 后 `pnpm build` 立即生效；但 `lib/` 必须随提交推送，GitHub 直装才
  会包含最新构建产物。

## opencode 二进制与网络策略

- 直接使用官方 opencode 二进制，版本锁定 `opencode-version.json`（当前
  `1.18.18`）；启动时 `resolveOpenCodeBinary` + `verifyOpenCodeVersion` 双重
  校验，显式 `DSH_OC_OPENCODE_BIN` 版本不匹配会直接报错，不回退到 PATH 上
  的其它版本。
- 解析优先级：`DSH_OC_OPENCODE_BIN` → `$DSH_HOME/opencode/bin/<version>`
  → PATH 上版本匹配的 `opencode` → 官方 npm 平台包（惰性安装，npm integrity
  校验）→ profile 内 `opencode-ai` 包 → GitHub Release 惰性下载。
  `opencode-assets.json` 覆盖各平台/架构变体，每个 asset 独立 sha256 与 npm

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chiro2001/dsh-oc](https://github.com/chiro2001/dsh-oc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
