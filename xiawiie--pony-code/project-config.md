---
trigger: always_on
description: 本文件是全仓唯一 Agent 规则真源；不要创建重复规则文件。用户、领域、架构、安全与发布说明分别见 README 及
---

# Pony Agent 工作约定

本文件是全仓唯一 Agent 规则真源；不要创建重复规则文件。用户、领域、架构、安全与发布说明分别见 README 及
`docs/domain-model.md`、`docs/architecture.md`、`docs/security.md`、`docs/verification.md`。

## 1. 产品目标与边界

Pony 1.0 是本地 coding agent：从仓库构造上下文，以受约束工具修改和验证代码，并把 Session、Run、Checkpoint、
Memory 与恢复证据保存在 `.pony/`。

维护优先级：

1. 一个可安装的 `pony` CLI/TUI 完成配置、运行、检查与恢复；
2. Provider 路由显式、可观察，不猜测、不 fallback；
3. 文件、secret、permission、mutation lock 与 legacy inspection 边界 fail closed；
4. 代码按领域集中，公共 API 和分发包最小；结论由当前 exact HEAD 的证据支撑。

非目标：动态 Provider registry、旧兼容层、真实任务失败后的 Provider/协议 fallback、OS Sandbox、distributed/remote/
multi-tenant 执行和无需求的抽象。发送真实任务前的 bounded synthetic Provider resolution 是显式产品能力，不属于
fallback。Host 不是 OS sandbox。

## 2. 开始工作与 Git 纪律

开始前先检查：

```bash
git status --short
git branch --show-current
git rev-parse HEAD
git log -1 --oneline
```

- 用户修改、未跟踪文件和其他 worktree 不覆盖、不移动、不重置、不顺手清理。
- 生产收口、大型重构和发布准备从用户指定或已验证的干净基线建独立 worktree 与 `codex/<topic>` 分支；记录
  base SHA，不得操作主工作区的用户文件。
- 普通任务不隐含 commit、push、tag、PR、Release 或 PyPI 授权。
- 搜索优先使用 `rg` / `rg --files`。先定位责任模块、最窄测试和文档真源，再动代码。

## 3. 公共边界与目录所有权

`pony/` 顶层只能有 `__init__.py` 和 `__main__.py`。`import pony` 只公开 `Pony`；console entry 是
`pony.cli.app:main`；构造合同为 `Pony(model_client, workspace, session_store, *, session=None, options=None)`。

可选设置只进入冻结的 `RuntimeOptions`。内部从所属模块导入；package `__init__.py` 只写说明，不做 re-export/shim。

| 路径 | 唯一责任 |
| --- | --- |
| `pony/agent/` | Action、Agent Loop、Canonical Messages、compaction、预算与观测 |
| `pony/cli/` | app、arguments、assembly、命令、REPL、人类/JSON 输出与 doctor |
| `pony/config/` | `.env`、Provider 规格与 `pony.toml` 校验 |
| `pony/context/` | Context source、chunk、escaping、render 与 digest |
| `pony/memory/` | User/Agent Notes、recall、retrieval、RepoMap 与 memory service |
| `pony/providers/` | 四个 wire adapter、Response、transport helper、factory 与 probe |
| `pony/runtime/` | `Pony` 装配、options、reporting、rewind、working memory 与 worktree child 调度 |
| `pony/security/` | path、private/workspace file、redaction 与 shell command policy 原语 |
| `pony/state/` | Session/Run、legacy artifact reader、TaskState 与 file lock |
| `pony/tui/` | 行内 prompt、slash completion、Markdown、状态与 permission/activity 渲染 |
| `pony/tools/` | registry、validation、executor、permission prompt、effect recorder 与 subprocess |
| `pony/workspace/` | root discovery、WorkspaceContext 与 observer |

开发资产不进入 runtime package；Fake Provider 只在 `benchmarks/support/`，evaluation 不回迁 `pony/`。

CLI/TUI 合同：

- 裸 `pony` 与 `pony repl` 进入同一个交互会话；`pony run <prompt...>` 一次执行后退出。
- `runs`、`sessions`、`session`、`checkpoints` 等显式管理命令保持独立；未知首 token 不得静默变成 prompt。
- TUI 是 presentation adapter，必须与纯文本 fallback 共用 REPL handler、Agent、Session、finalize 和错误语义。
- `/` 菜单只展示真实命令；不得增加绕过 permission check 的 `!` shell mode、动态 Provider registry、模型 catalog 或
  第二命令 registry。`/model` 只允许当前 Session 在相同 protocol/endpoint 内切换模型。
- `--permission-mode` 只适用于 `run/repl`，公开值与 Claude Code 一致：`manual|auto|acceptEdits|bypassPermissions|dontAsk|plan`；
  `manual` 只在 CLI 边界映射为内部 `default`。`bypassPermissions` 必须通过两个 dangerous bypass flag 之一显式启用。
- `/permissions` 与 `/allowed-tools` 共用 REPL handler 管理 allow/ask/deny 规则和 mode；CLI allowed/disallowed flags
  复用同一 rule parser 与 Session writer。`/plan` 进入或查看 Plan，旧 `/mode` 与 `/todo` 不再存在，`/plan clear`
  不再具有清空语义。
- 忙碌时最多保留五条内存 follow-up input；单一 worker 只在完整 turn 结束后按 FIFO 调用同一 REPL handler。
  `/queue [clear]` 只查询或清空未执行输入，零 Session 写；approval 始终先回到前台，local slash command 不抢占当前 turn。
  不持久化队列，不修改 immutable request，不实现 Provider/tool cancel、daemon 或第二 Session writer。
- 仓库 Skill 只从受信 root 的 `.claude/skills/<name>/SKILL.md` 发现，且仅由显式 `/name` 作为本 turn 的只读 context
  调用；可选 `resources` 只显式列出同一 Skill 目录内 bounded UTF-8 文件，不递归、不 glob。严格 frontmatter、
  bounded/no-follow/single-link/root identity 与 secret gate 任一失败即 catalog fail closed。用户请求优先于项目规则，项目
  规则优先于 Skill；诊断不得回显被拒路径或内容。不得
  读取 HOME/plugin/`.agents` 兼容路径，不得执行 Skill 脚本、安装、注册工具或持久化 loaded state。
- transient bypass capability 只进入冻结的 `RuntimeOptions`，不持久化；构造、resume、mode setter 与 Executor 都必须
  fail closed。`/plan open|share` 从非 Plan mode 调用时先进入 Plan；空 artifact 不打开 editor 或 share。
- TUI 只在 stdin/stdout 为 TTY、终端能力可用且至少 112 列时启用；必须遵守 `NO_COLOR` / `--no-color`。Windows 不要求
  `TERM`；其他平台拒绝缺失/空白或 `TERM=dumb` 的交互 TTY。
- 完整 TUI 只允许完整尺寸的马形 `PONY CODE` 欢迎资产；Logo/字标不可隐藏，`--quiet` 也不能抑制它。交互 TTY 低于
  112 列时以稳定 usage error 要求扩宽，不能进入无 Logo 的纯文本 REPL，也不得恢复 medium、micro、缩放或单行替代版。
  只有非 TTY 自动化 fallback 和 `pony run` 不输出装饰性 banner。
- 完整尺寸的马形 Logo、块状字标、欢迎页布局和视觉语言是用户冻结的产品资产。只有用户明确要求修改设计时才可变更；
  重构、Claude Code/Codex 交互对齐、性能优化和代码精简都不是修改授权。允许修复宽度、裁切、颜色能力和字符兼容 bug，
  但必须保持“完整大版是唯一 TUI 状态”的视觉意图，并通过 80/111 列拒绝、112/120 列完整大版和运行中缩放回归测试。
- 用户消息使用低对比块、无色侧轨且不加角色标签；Assistant Answer 以一次低对比 `Pony` 标识开始并使用内置、安全的
  Markdown renderer，输入使用非空 `› ` prompt，消息块之间只留一个视觉间距。
- `Working…` 是唯一可清除的瞬态活动状态；Tool 开始只原位替换为具体动作，完成后只显示一条永久语义回执；自动
  checkpoint 不进入对话区，成功、分页、截断、失败与中断必须可见。
- 输入框最多增长六行，completion 菜单最多显示五项。footer 只保留仓库/分支、permission mode、Provider/model，窄终端
  优先保留安全和模型信息；不得显示绝对路径、Session ID、API Base 或 checkpoint ID。
- 不显示或持久化 Provider reasoning，不增加全屏 transcript、主题系统或新的运行时依赖。Streaming 只由完整交互 TUI 的
  `--stream` 显式启用；默认、`run`、非 TTY、doctor、probe、resolution、compaction 和 delegate 保持 final-only。它不写
  配置或 Session，不 fallback；四种 adapter 各自解析协议流并返回完整 `Response`，完整 decode 前工具零执行。
- UI listener 只能在 trace durable append 后收到脱敏副本；Tool 摘要需要的参数/结果仅存在于该内存副本，不扩大

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiawiie/pony-code](https://github.com/xiawiie/pony-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
