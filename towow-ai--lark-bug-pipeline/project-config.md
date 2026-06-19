---
trigger: always_on
description: 飞书 Bug 反馈 → 自动修复 → PR 的端到端流水线。用户在飞书群里 @bot 扔一句话 bug，15 分钟后 GitHub 上自动开好 PR。依赖 Claude Code harness（headless `claude -p`）做 triage 和 guardian-fixer 的 8 Gate 闭环。
---


# 飞书 Bug → PR 全自动流水线

## 一句话

**用户在飞书群 @bot 发一条 bug，Mac 本地两个 launchd 常驻进程捡起来，调 `claude -p` 跑 triage + guardian-fixer 8 Gate 修复流程，最后自动开 PR 到 GitHub。**

你（维护者）只需要：
1. 飞书建个自建应用 + 把 bot 拉进目标群（不需要多维表格，默认 IM-only）
2. 填 `~/.towow/.env.lark`（4 个必填变量）
3. `bash .claude/skills/lark-bug-pipeline/install.sh`
4. 关电脑不管它

下次开机登录桌面后，两个 LaunchAgent 自动拉起，pipeline 继续待命。

---

## 这个东西解决的问题

早期产品有一堆 bug，但开发者看不到 / 用户懒得写 / 开发者懒得修的三重循环。传统方案是 Sentry + Linear + Jira 三件套，需要用户愿意填 form、开发者愿意分诊、团队愿意挤进 sprint。三个链条任何一环断了，bug 就烂在后台。

这个 skill 把这条链路全部压缩到 15 分钟：

| 环节 | 传统做法 | 本 skill |
|---|---|---|
| 用户反馈 | 提 GitHub Issue / 写邮件 / 填表 | 飞书群 @bot 一句话 |
| 分诊 | 产品经理人工判断优先级 | `lark-triage` skill 自动判断 auto / needs_user_clarification / out_of_scope / needs_nature |
| 修复 | 开发者进入 sprint 排期 | `guardian-fixer` skill 8 Gate 走完（PLAN → REVIEW → TASK → REVIEW → 实现 → TEST → FINAL-REVIEW → CLOSURE） |
| PR | 人工写描述 + 补测试 + 找 reviewer | `gh pr create` 自动带双语标题 + reporter attribution + 8 Gate artifact |
| 成本 | $$$ 人天 | **0 API 成本**（走 Claude Code 订阅而非 API billing） |

**真实运行证据**：PR [NatureBlueee/Towow#90](https://github.com/NatureBlueee/Towow/pull/90) 是这条流水线处理的第一条真用户反馈，从"飞书 @bot 发消息"到"GitHub PR OPEN"总耗时约 15 分钟，triage 2 分钟 + fixer 6.7 分钟 + bookkeeping。

---

## 架构一眼图

```
┌─────────────────┐
│ 飞书群 @bot 发 bug │
└────────┬────────┘
         │ WebSocket long-poll
         ▼
┌──────────────────────┐      ┌─────────────────────────┐
│ bug_daemon.py        │      │  ~/.towow/queue/        │
│ (LaunchAgent #1)     │─────▶│  <ts>-lark-im-*.json    │
│ lark-oapi 长连接      │      │  (JSONL 队列)            │
└──────────────────────┘      └───────────┬─────────────┘
                                          │
                                          │ 30s poll
                                          ▼
                              ┌─────────────────────────┐
                              │ bug_worker.py           │
                              │ (LaunchAgent #2)        │
                              │ batch + bundle + route  │
                              └───────────┬─────────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
         ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
         │ claude -p        │  │ git worktree add │  │ claude -p        │
         │ --skill          │  │ + stage issues   │  │ --skill          │
         │   lark-triage    │  │                  │  │   guardian-fixer │
         │ (opus-4-6)       │  │                  │  │ (opus-4-6)       │
         └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘
                  │ state file          │                     │ PR url
                  ▼                     ▼                     ▼
           docs/issues/*.md       isolated branch         gh pr create
```

三个关键隔离：
1. **daemon 和 worker 进程隔离** — daemon 挂了不影响已排队的 bug 继续修
2. **worker spawn 子进程跑 `claude -p` 而不是 inline 调 API** — 用 Claude Code 订阅，不烧 API tokens
3. **每个 bundle 一个 git worktree** — 多个 bug 可以并行修互不踩，失败也不污染主工作区

---

## 和 harness 的关系

这个 skill **不是**一个独立可跑的东西。它假设你背后已经有一套 Claude Code harness：

| 依赖 | 是什么 | 哪里拿 |
|---|---|---|
| `claude` CLI | Claude Code 命令行，headless 模式 `claude -p ... --permission-mode bypassPermissions` | <https://claude.com/claude-code> |
| Claude 订阅 | 别用 API key —— 两条修复走 API 要 $$$，走订阅是 flat rate | Claude Max 订阅 |
| `lark-triage` skill | 把用户一句话翻译成结构化 issue 草稿 + escalation 判定 | 本 skill 的 `runtime/lark-triage.md`，install.sh 会拷到 `.claude/skills/lark-triage/` |
| `guardian-fixer` skill | 8 Gate 修复流程（规划/审查/实现/测试/closure） | **你自己要有**。本 skill 只依赖它的 slash-command 接口；参考 Towow 主仓的 `.claude/skills/guardian-fixer/SKILL.md` |

`guardian-fixer` 是 harness 里最重的一块，本 skill 没把它包进来的原因：它是一套跟整个仓库工程规范深度耦合的东西（测试基线、review 规范、ADR-030 不可降级要求、commit 双语规则等等），强行打包没意义。你自己的仓库有自己的 guardian-fixer 实现，或者参考 Towow 的版本改一个。

---

## 安装

### 0. 前置条件

- macOS 14+
- Python 3.9+，`pip install lark-oapi`
- `claude` CLI 已登录，`claude -p "hello"` 能跑通
- 你的仓库里已经有 `.claude/skills/guardian-fixer/SKILL.md`

### 1. 飞书侧（详见 `docs/feishu-setup.md`）

1. 在 <https://open.feishu.cn/app> 创建「自建应用」
2. 申请 scope：`im:message`, `im:message.group_at_msg`, `bitable:app`
3. 开启「事件订阅」→ 消息与群组 → `im.message.receive_v1`
4. 新建多维表格「Bug 反馈」，字段见 `docs/feishu-setup.md`
5. 把应用加到你想收 bug 的群，给机器人一个名字

### 2. 环境变量

```bash
mkdir -p ~/.towow
cp .claude/skills/lark-bug-pipeline/templates/env.lark.example ~/.towow/.env.lark
# 按注释填实际值（App ID, Secret, Table Token, Bot Open ID 等）
```

### 3. 一键装

```bash
cd <你的项目根目录>
bash .claude/skills/lark-bug-pipeline/install.sh
```

install.sh 会：
- 检查 macOS / python3 / claude CLI / lark-oapi
- 拷贝 `bug_daemon.py` 和 `bug_worker.py` 到你仓库的 `scripts/lark/`
- 拷贝 `lark-triage` 子 skill 到 `.claude/skills/lark-triage/`
- 渲染两个 LaunchAgent plist 到 `~/Library/LaunchAgents/`（替换 `{{PYTHON}}` / `{{REPO_ROOT}}` / `{{HOME}}` / `{{PATH_PREFIX}}`）
- `launchctl bootstrap` 启动两个服务
- 验证 state=running，打印运维命令

### 4. 验证

```bash
# 在飞书群里 @bot 发一条 bug
# 然后看日志
tail -f ~/.towow/logs/lark-worker.log
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Towow-ai/lark-bug-pipeline](https://github.com/Towow-ai/lark-bug-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
