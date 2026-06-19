---
trigger: always_on
description: Use when setting up or improving an agent memory workflow for OpenClaw, Codex, or Obsidian with MEMORY.md, daily notes, session recovery, and optional OpenViking support.
---


# Agent 记忆系统搭建指南 Skill

> 🧠 从零搭建 Agent 长期记忆系统。基于 OpenClaw 实战，覆盖本地优先的 `MEMORY.md`、每日笔记、`SESSION-STATE.md`、`working-buffer.md`、Obsidian 原生笔记与可选召回后端全流程。

## 触发词

`记忆系统`、`memory-setup`、`搭建记忆`、`记忆架构`

## ⚡ 5 分钟快速上手

**如果你赶时间，只需做这几步：**

### 第一步：创建 `MEMORY.md`

```markdown
# MEMORY.md

> 长期记忆。只保留会持续影响协作的事实、偏好和决策。

## User

- Preferred name: K
- Timezone: Asia/Shanghai

## 当前任务

- [ ] 正在做的事

## 决策记录

- YYYY-MM-DD: 决策内容 + 原因

## 踩坑记录

- 问题 → 解决方案
```

### 第二步：创建每日笔记

```markdown
# memory/2026-03-20.md

## 完成
- 分析了网宿科技

## 决策
- 暂不加仓

## 踩坑
- （无）

## 待办
- [ ] 明天要做什么
```

### 第三步：从模板创建 `SESSION-STATE.md` 和 `working-buffer.md`

```markdown
# SESSION-STATE.md

## 当前任务
- 任务名称

## 已完成
- ...

## 卡点
- ...

## 下一步
- ...

## 恢复信息
- 最近一次有效上下文：...
```

```markdown
# working-buffer.md

## 进行中
- ...

## 临时决策
- ...

## 未完成
- ...
```

模板来源：

- `templates/SESSION-STATE.md`
- `templates/working-buffer.md`

### 恢复层契约先写死

- `SESSION-STATE.md` 只使用仓库模板提供的简洁结构：`当前任务`、`已完成`、`卡点`、`下一步`、`恢复信息`
- 不要写入 `Task`、`Status`、`Owner`、`Last Updated`、`Cleanup Rule` 这类另一套 schema 字段
- 如果外部 skill 产出的是详细版字段：
  - `Current Task` 合并到 `当前任务`
  - `Status` 合并到 `已完成`、`卡点` 或 `下一步`
  - `Owner`、`Last Updated`、`Cleanup Rule` 只在条目文本里按需保留，不扩展新标题
- `working-buffer.md` 是唯一的短期毛坯区
- 其他 skill 如果也有 working buffer 概念，应复用这个文件
- 不要再创建第二份并行写入的 WAL / buffer 文件
- `MEMORY.md` 用于启动时快速参考
- `memory/` 用于每日笔记和深度归档
- 两者允许主题重叠，但检索顺序不同：先 `SESSION-STATE.md`，再 recent notes，再 `MEMORY.md` / `memory_search`，最后再查归档

### 第四步：使用 Obsidian 原生笔记模板

```markdown
# templates/OBSIDIAN-NOTE.md

---
title: ""
aliases: []
tags: []
type: memory
status: draft
source: ""
source_url: ""
created: 2026-03-21
updated: 2026-03-21
related: []
---

## Summary
- 一句话摘要

## Key Points
- ...

## Evidence
> ...

## Related Notes
- [[SESSION-STATE]]
- [[working-buffer]]

## Attachments / Embeds
- `![[image.png]]`
- `![[note#^block-id]]`
```

### 第五步：每次对话开始时

```text
先读 SESSION-STATE.md，再读最近 1-3 天 daily notes，最后才做 memory_search。
memory_search(query="相关关键词")
```

**就这么简单。后续优化可以慢慢加。**

## 实战工作流示例

### 首次引导

首次引导工作区就当是开箱即用：复制 `templates/SESSION-STATE.md`、`templates/working-buffer.md` 和 `templates/memory-capture.md`，再跑一遍 `python3 scripts/memory_capture.py bootstrap --workspace /path/to/workspace`（`bootstrap` 可以省略，默认行为一致），此时 `SESSION-STATE.md`、`working-buffer.md` 和 `memory-capture.md` 都已经齐活了。`MEMORY.md` 仍然建议手动建立和维护，因为它属于长期记忆主文件，不应该被脚本悄悄代写。

### 会话启动初始化

如果你想从真实会话的第 1 轮就把恢复层准备好，可以运行 `python3 scripts/memory_capture.py session-start --workspace /path/to/workspace`。它会确保基础恢复文件存在，并给 `memory-capture.md` 补上结构化 capture metadata；需要时还可以传 `--session-id`、`--project` 和重复的 `--scope-tag`。

### 任务结束记忆捕获

在任务进行中写 `working-buffer.md` 的 `临时决策`/`新坑`/`待蒸馏`，任务结束前 30 秒用 `templates/memory-capture.md` 的 `候选决策`、`候选踩坑`、`候选长期记忆` 把最重要的内容整理出来，再决定哪些内容最终写入 `MEMORY.md`。这个环节让临时笔记和长期记忆的边界清晰且不会掉链子。

如果你想先做一层机器可读但仍然人工复核的整理，可以运行 `python3 scripts/memory_capture.py distill --workspace /path/to/workspace`。它会把候选内容整理成 `suggested_memory`、`recovery_only` 和 `follow_up`。

如果要把这一步交给别人复核，再加 `--output /path/to/distill-report.md`。输出的 Markdown 会带上 `candidate_document_id`，并按 `候选决策`、`候选踩坑`、`候选长期记忆` 分段展示建议写入长期记忆的内容。

如果你想把这条链路真正闭环，再运行 `python3 scripts/memory_capture.py apply --workspace /path/to/workspace`。它会把当前 distill 结果写进 `MEMORY.md`，并通过 `candidate_document_id` 保证重复执行不会重复写入。

### 每日笔记蒸馏

每日笔记蒸馏指的是定期从 `memory/` 下最新的 Markdown 文件里抽取事实和决策转换到 `MEMORY.md`，同时保留原笔记供深度回溯。把这个行为设定为每日或每周结束时的步骤，能确保 `MEMORY.md` 只包含真正长期有用的内容。

### 维护报告命令

`report command` 用来检查工作区当前状态，**永远不会**写入记忆文件。运行 `python3 scripts/memory_capture.py report --workspace /path/to/workspace`，它会输出四节：**Supported files**（`MEMORY.md`、`SESSION-STATE.md`、`working-buffer.md`、`memory-capture.md`）、**Directories**（递归扫描 `memory/` 和 `attachments/`，在 memory/ 里只统计形如 `YYYY-MM-DD.md` 的 daily notes，在 attachments/ 里数所有文件）、**Latest daily note**（选取 `memory/` 下字典序最新的匹配 daily note 路径）、**Warnings**（比如某个文件缺失或者权限异常）。像 `memory/index.md` 这样的参考页不会被当成 latest daily note。报告命令只在工作区目录不存在或无法读取时退出非 0；其它情况下即便有警告也返回 0。supported files、directories、latest daily note 以及 warnings 是报告里每条节的标题，方便快速对照命令输出。

`doctor` 用来做按作用域收敛的健康检查。运行 `python3 scripts/memory_capture.py doctor --workspace /path/to/workspace` 时，默认只检查当前启用的本地恢复层；如果这次工作流确实在用 Obsidian，再显式加上 `--obsidian-vault /path/to/vault`。

---

## 为什么需要记忆系统

Agent 每次会话醒来都是空白的。没有记忆文件，你就等于每次失忆重启。记忆系统是 Agent 的“大脑持久化”方案。它不让你变聪明，但让你不犯重复的错误。

这个 skill 的核心定位是本地优先的文件工作流和恢复约定，不是托管式 memory platform。外部检索或语义召回能力只能作为可选后端接在后面，不能替代本地恢复层。
即使 OpenClaw 新版已经提供 native memory、Active Memory、diary、dreaming、grounded recall，或更新的 import / wiki 流程，这个 skill 仍然负责可审计、可迁移的 local recovery layer，并与这些原生能力互补而不是竞争。

## 核心架构（本地优先分层）

```text
workspace/
├── SESSION-STATE.md
├── working-buffer.md
├── MEMORY.md
├── memory-capture.md
├── attachments/
├── templates/
│   ├── SESSION-STATE.md
│   ├── working-buffer.md
│   └── OBSIDIAN-NOTE.md
└── memory/
    ├── 2026-03-17.md
    ├── 2026-03-16.md
    └── ...
```

### 第一层：恢复层（`SESSION-STATE.md`）

**保存当前任务恢复所需的最小真相。**

**该记什么：**
- 当前任务和最近已完成项
- 卡点、风险、下一步
- 中断后恢复所需的上下文

**不该记什么：**
- 另一套项目管理 schema
- 大段长期背景资料
- 需要反复整理的毛坯草稿

### 第二层：毛坯层（`working-buffer.md`）

**临时决策、草稿和待蒸馏内容先落这里。**

**该记什么：**
- 临时判断
- 新坑
- 待蒸馏条目
- 未完成但还没整理成稳定表达的内容

**不该记什么：**
- 第二份并行 buffer
- 已经蒸馏好的长期事实

### 第三层：长期记忆层（`MEMORY.md`）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cjke84/agent-memory-system-guide](https://github.com/cjke84/agent-memory-system-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
