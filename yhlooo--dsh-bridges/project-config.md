---
trigger: always_on
description: 本文件是本仓库协作编码智能体的共享记忆（dsh、Claude Code、Codex、OpenCode、
---

# AGENTS.md

本文件是本仓库协作编码智能体的共享记忆（dsh、Claude Code、Codex、OpenCode、
CodeBuddy……）。在本仓库的所有工作中请遵守以下约定。

## 插件约定

### 布局

- 本仓库**本身就是插件**：仓库根目录就是 `dsh-bridges` dsh bundle，不是插件
  monorepo。`cordis.patch.yml` 恰好插入**一行**；每个受支持的 agent 工具都是
  `src/agents/<tool>/` 下的一个子系统，从 `src/index.ts` 注册。绝不为新 agent
  工具新增第二个 bundle、行或包。
- 新增一个 agent 工具意味着：一个 `src/agents/<tool>/` 目录、
  `registerBridgeSubsystems()` 里一行注册、`bridges` 行上一个配置段。共享代码
  放在 `src/util.ts` / `src/fs-adapter.ts`。
- 子系统注册的每一个副作用（provider、事件监听、watcher、spawn 的子进程）都
  必须属于插件 fiber，并且在 teardown 时可逆。

### 命名

- Patch 行 `name` = npm 包名（loader 导入所用的名字）；patch 行 `id` = 短语义
  名：包名去掉 `@scope/dsh-` 前缀，与随附 bundle 保持一致（`dsh-bridges` →
  `bridges`，如同 `@deepseek-ai/dsh-skill-filesystem` → `skill-filesystem`）。
  `id` 是后续 patch 层覆盖配置所用的稳定键——绝不要用完整包名作 `id`。
- 技能 provider：每个 agent 工具一个，以工具命名（`claude-code`、
  `codebuddy-code`、`opencode`、`codex`、`pi`、`gemini-cli`、`cursor`）。每个
  provider 独占一段 rank（claude 105–120、codebuddy 125–140、opencode 145–160、
  codex 165–175、pi 180–195、gemini-cli 205–220、cursor 225–240；段分配避开
  filesystem provider 的整百点，详见 `docs/development/dsh-integration-surface.md`）；
  同层内 rank 越小越优先，段内资产遵循上游工具的优先级（Claude Code：个人 >
  项目；CodeBuddy Code / OpenCode / Codex：项目 > 用户；Pi：个人 > 项目；
  gemini-cli：工作区 > 用户；cursor：项目 > 用户），且技能优先于同级同名命令。
- 每个桥接的技能 provider 都注册在**全局**技能层，因此 preset 层的 DeepSeek
  Harness 原生技能（`.dsh/skills`、`.agents/skills`、运行时技能）通过层序在
  同名冲突时遮蔽桥接资产。绝不要用 rank 数字论证这一优先级——同层内桥接段的
  数字其实高于运行时技能（250），保住该优先级的是层序。
- `bridges` 行上的配置段以工具命名（`claudeCode`、`codebuddyCode`、`opencode`、
  `codex`、`pi`、`geminiCli`、`cursor`），各自带 `enabled` 总开关和每桥接的
  具体参数。
- 注入消息的 `source.plugin` id 一律以 `dsh-bridges:` 开头，Web GUI 会把它
  原样显示在「上下文注入」标签旁。记忆注入用 `dsh-bridges:<资产名>`（如
  `dsh-bridges:CLAUDE.md`、`dsh-bridges:AGENTS.md`、
  `dsh-bridges:CODEBUDDY.md`、`dsh-bridges:GEMINI.md`、
  `dsh-bridges:.cursor/rules`、`dsh-bridges:references`）；hook 注入用
  `dsh-bridges:<tool>-hooks/<事件名>`（如
  `dsh-bridges:claude-code-hooks/UserPromptSubmit`，事件名取上游 hook 事件名，
  如 `UserPromptSubmit`、`PreToolUse`、`sessionStart`）。hook 的 `tool_name`
  载荷携带上游工具的名字（`Bash`、`Edit`……），绝不携带 dsh 的名字。
- DeepSeek Harness 核心自行加载的指令文件绝不重复注入：项目根到 cwd 每层
  目录的 `AGENTS.md` / `CLAUDE.md` / `AGENTS.local.md` / `CLAUDE.local.md`
  由核心负责（`~/.dsh/AGENTS.md` 同理）；桥接只注入上游工具特有、核心不
  覆盖的资产（`~/.claude/CLAUDE.md`、`.claude/CLAUDE.md`、`CODEBUDDY.md`、
  `GEMINI.md`、`.cursor/rules` 等）。去重边界用 `.git` 根（无 `.git` 时以
  cwd 为界），与核心默认 project-root 标记一致。

## 文档约定

### README

- 两个根 README 是**面向用户的入口**。保持简短（约一屏），开头用能展示收益的
  快速上手——安装、在已有的 agent 项目里运行、展示用户得到了什么——而不是功能
  列表。
- 详细使用说明（安装与验证、完整配置参考、各桥接 skills/memory/hooks 行为、
  限制）放在 `docs/guides/`；README 链接过去。开发细节（构建/测试命令、冒烟
  测试、目录结构）绝不进 README——链接到 `docs/development/`。
- `README.md`（英文）与 `README.zh.md`（中文）必须保持同步：任何改动两版都要
  做，且都以语言切换头开头（`English | [中文](README.zh.md)` /
  `[English](README.md) | 中文`），紧跟备注
  `> This project is implemented by DeepSeek Harness.`（中文版：
  `> 该项目由 DeepSeek Harness 实现。`）。
- 正文中一律写全称 **DeepSeek Harness**——绝不用 `dsh`/`DSH`。仅在标识符场合
  保留短写：CLI 命令（`dsh plugin`、`dsh --profile`）、包名 `dsh-bridges`、
  配置键（`dsh.profile.bundles`）、路径（`.dsh/skills`）。
- 上游工具名按官方写法大小写：**Claude Code**、**CodeBuddy Code**、
  **OpenCode**、**Codex**、**Pi**、**Gemini CLI**、**Cursor**（正文中指代工具
  时用此写法，依据各自官方文档正文：opencode.ai 文档用 OpenCode、pi 文档用
  Pi）。标识符保持小写：CLI 命令（`opencode`、`pi`）、路径（`.opencode/`、
  `.pi/`、`~/.pi/agent`）、provider 名（`opencode`、`pi`）、配置键
  （`opencode:`、`pi:`）、包名（`pi.skills`）。
- 文档描述的行为必须与代码一致。曾坑过我们的例子：dsh 的 todo 工具叫
  `todo_write`，所以 hook 名称映射表必须映射 `todo_write`→`TodoWrite`——
  `todo` 条目匹配不到任何东西。

### docs/ 布局

- `docs/guides/` — 面向用户的使用指南。入口索引在 `README.md`（英文）与
  `README.zh.md`（中文）：安装与验证、公共行为、逐工具索引；每个桥接一个独立
  页面 `<tool>.md`（中文版 `<tool>.zh.md`），页内结构统一为「配置块 →
  skills/commands → 记忆 → hooks → 权限 → MCP → 子代理 → 限制」，各桥接共有的
  规则（原生技能遮蔽、32 KiB 记忆预算、热更新、fail-open）只写在索引页的
  "Common behaviors / 公共行为"，工具页用一句话引用，不重复铺陈。
- `docs/reference/` — 各桥接目标的官方上游文档，保持原文不改动。
- `docs/development/` — 贡献者指南（中文），包括新增桥接的清单。
- `docs/development/todo.md` — **待办与特性补全计划**：所有"需要做但不立即做"的
  工作都追加到这里（按优先级分组，完成后勾选并注明 commit）。桥接审计、新增工具
  后发现的遗留事项一律写入，不要只留在会话记忆里。

### 新增桥接按此顺序更新文档

1. `docs/reference/<tool>/` — 先收集官方上游规范。
2. `docs/guides/` — 为该工具新增独立页面 `<tool>.md` / `<tool>.zh.md`
   （skills/commands、记忆、hooks、限制与配置块），并在 `README.md` /
   `README.zh.md` 索引页登记一行（一句话概述 + 链接）。
3. 根 README（两版语言）— 状态 callout、支持矩阵表行（✓/—，链接到该工具
   指南页）、guides/reference 链接。
4. 本轮未做 / 待做的遗留事项写入 `docs/development/todo.md` 并保持更新。

## Git 提交约定

本项目使用 [Conventional Commits](https://www.conventionalcommits.org/)。

提交消息格式：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

描述是简短的祈使句总结（例如 "add foo bar" 而不是 "added foo bar"），小写，
结尾不加句号。

### 类型

| 类型        | 用途                                                              |
| ----------- | ----------------------------------------------------------------- |
| `feat`      | 新功能                                                            |
| `fix`       | 缺陷修复                                                          |
| `docs`      | 仅文档改动                                                        |
| `style`     | 仅格式化；不改代码含义                                            |
| `refactor`  | 既不修 bug 也不加功能的代码改动                                   |
| `perf`      | 性能改进                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yhlooo/dsh-bridges](https://github.com/yhlooo/dsh-bridges) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
