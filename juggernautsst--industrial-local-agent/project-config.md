---
trigger: always_on
description: 本文件适用于 `Industrial_Local_Agent` superproject 及从本目录进入的所有组件。更深目录中的 `AGENTS.md` 可以增加组件规则；发生冲突时，遵循更具体且更保守的规则。
---

# AGENTS.md

## 1. 适用范围 / Scope

本文件适用于 `Industrial_Local_Agent` superproject 及从本目录进入的所有组件。更深目录中的 `AGENTS.md` 可以增加组件规则；发生冲突时，遵循更具体且更保守的规则。

This file governs the `Industrial_Local_Agent` superproject and components reached from this checkout. A deeper `AGENTS.md` may add component-specific rules; when rules conflict, follow the more specific and more conservative rule.

根文件在子仓库被独立克隆时不会自动生效。对子仓库进行独立工作前，应读取该仓库自己的 `AGENTS.md`；若不存在，应通过该子仓库的 Issue 单独同步核心治理规则。

This root file does not automatically govern a child repository cloned on its own. Before standalone child work, read that repository's own `AGENTS.md`; if none exists, propagate the core governance rules through a separate Issue in that child repository.

所有面向用户的沟通、Issue 摘要、PR 摘要和持久性项目文档使用中文和英文对照。代码标识符、命令、日志片段和第三方原文无需机械翻译。

All user-facing communication, Issue summaries, PR summaries, and durable project documentation must be bilingual in Chinese and English. Code identifiers, commands, log excerpts, and quoted third-party text do not require mechanical translation.

## 2. 核心原则 / Governing Principles

1. 一项可独立验收的工作对应一个 canonical GitHub Issue；不要按命令、文件或每次测试创建 Issue。
   One independently acceptable deliverable maps to one canonical GitHub Issue; do not create Issues per command, file, or test run.
2. Issue/PR 保存工作历史，README、架构和路线文档保存当前事实。重要决策不得只存在于已关闭 Issue 中。
   Issues and PRs preserve work history, while README, architecture, and roadmap documents preserve current truth. Durable decisions must not exist only in closed Issues.
3. 远端留痕必须有信息价值：记录目标、范围变化、关键决策、阻塞、外部副作用和验收证据，不记录每条本地命令或普通迭代失败。
   Remote traceability must carry information: record objectives, scope changes, material decisions, blockers, external effects, and acceptance evidence, not every local command or routine iteration failure.
4. Issue 不是授权。创建或关联 Issue 不自动授权 branch、commit、push、PR、评论、label、Release、仓库设置、云任务、付费任务或数据传输。
   An Issue is not authorization. Creating or linking an Issue does not automatically authorize branches, commits, pushes, PRs, comments, labels, releases, repository settings, cloud jobs, paid work, or data transfer.
5. private GitHub 不是加密存储、主机隔离或高价值科研数据传输通道。
   Private GitHub is not encrypted storage, host isolation, or a transfer channel for high-value research data.

## 3. 何时必须有 Issue / When an Issue Is Required

在产生持久修改或外部副作用之前，下列工作必须关联一个处于 open 状态且归属正确仓库的 Issue：

Before any durable mutation or external side effect, the following work must reference an open Issue in the correct owning repository:

经当前请求明确授权创建本任务的 canonical Issue，本身就是初始远端留痕，不需要另一个前置 Issue。此例外仅覆盖该 Issue 的首次创建；后续评论、label、PR、push 和其他远端动作仍以它为记录，并分别遵守授权边界。批量创建治理 Issue 或自动拆分 Issue 必须由已有 Issue 跟踪。

Creating the canonical Issue for the current task, when explicitly authorized by the current request, is itself the initial remote record and requires no prior Issue. This exemption covers only that initial creation; later comments, labels, PRs, pushes, and other remote actions use it as their record and remain separately authorization-bound. Bulk governance-Issue creation or automatic Issue splitting requires an existing tracking Issue.

- 源码、测试、文档、配置、依赖、接口或数据结构变更 / source, test, documentation, configuration, dependency, interface, or data-structure changes;
- submodule 增删、URL、branch 或 gitlink 指针变化 / submodule additions, removals, URLs, branches, or gitlink-pin changes;
- 会保留结果、影响研究判断或产生对外主张的科研实验 / research experiments whose results persist, affect decisions, or support external claims;
- Issue 的后续编辑、评论、label、milestone、PR、Release、repository settings 或 Actions 变更 / later Issue edits, comments, labels, milestones, PRs, releases, repository-setting, or Actions changes;
- 云端执行、付费 API、FlexCredits、外部协作方消息或数据传输 / cloud execution, paid APIs, FlexCredits, messages to external collaborators, or data transfer;
- 安全策略、权限、密钥生命周期、部署或迁移工作 / security policy, permissions, key lifecycle, deployment, or migration work.

以下活动不要求为每次执行新建 Issue：

The following activities do not require a new Issue for every execution:

- 回答问题、解释现有代码或纯只读调查 / answering questions, explaining existing code, or purely read-only investigation;
- 不保留产物、不改变决定且不产生外部副作用的本地临时实验 / disposable local experiments that retain no artifact, change no decision, and cause no external effect;
- 已有 Issue 范围内的普通实现迭代和重复测试 / routine implementation iterations and repeated tests within an existing Issue.

只读发现一旦改变范围、设计、风险判断或下一步，必须先回写已有 Issue；若不存在合适 Issue，则在持久修改前创建新 Issue。若当前请求没有远端写授权，只能完成只读调查、起草 Issue 内容并请求授权。

Once a read-only finding changes scope, design, risk, or next steps, record it in the existing Issue first. If no suitable Issue exists, create one before durable changes. Without current authorization for remote writes, perform only read-only investigation, draft the Issue, and request authorization.

## 4. Issue 创建标准 / Issue Creation Standard

创建前必须搜索重复项，并核对 GitHub 账号、owner、仓库、默认分支和目标组件。一个 Issue 只容纳一个可独立验收的交付；可分别验收的内容应拆分并互链。

Before creation, search for duplicates and verify the GitHub account, owner, repository, default branch, and target component. One Issue contains one independently acceptable deliverable; separable deliverables must be split and cross-linked.

Issue 至少包含：

Every Issue must include:

- 问题与目标结果 / problem and intended outcome;
- 范围内与明确非目标 / in-scope work and explicit non-goals;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Juggernautsst/Industrial_Local_Agent](https://github.com/Juggernautsst/Industrial_Local_Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
