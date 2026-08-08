---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目是什么

**考核驱动的个人学习工具**，工程内核是一个可观测、可恢复、可评测的 Agent Runtime（Python 3.12+）；
作者本人是用户 #1，同时作为 AI/Agent 工程师方向的简历项目。核心循环是"考核"：学完材料 → 被拷问
→ 暴露薄弱概念 → 记入记忆 → 下次优先考薄弱点。

**当前状态（2026-08-04）**：可观测/可恢复/可评测的 Agent Runtime 已落地——`kernel/`（events/runner/tools/
hooks/context/clock/recovery/trace/db）+ `providers/`（OpenAI 兼容 + Record/Replay）+ `domain/learning/`
（考核竖切 ingest→深读→出题→判卷→薄弱记账）+ `interfaces/cli/`（ingest/quiz/react/report/trace 子命令）
+ `evals/`（17 条 Tier-1 规则用例 + case15 校准优先 Tier-2 质量门）。**最小 ReAct 对话核（R1）与全局 KB 重构均已落地**（`grandquiz react`
可真机跑：自然语言选材料 + 定题型的持久全局知识库考核）；上下文压缩、真实网络抓取、跨会话去重与
自适应难度第一阶段也已完成。[稳定性加固](docs/devrecords/03-stability-hardening-closeout.md) 已收口；其上的
[修订化文档结构](docs/devrecords/04-revisioned-document-search-foundation.md) DS-S1–S4 代码也已落地：不可变 revision/tree、精确
Evidence、自然节点 Reader、FTS5 与有界 Agentic Search。生产 DB 已备份并迁移到 schema v11；新增真实材料后现为
4 resources / 122 items / 4 revisions / 1723 nodes / 1723 FTS rows / 169 evidence（117 resolved / 52 unresolved）。
Reader、ReAct case14/case15/case17 与 Tier-2 judge 已用真实模型录制；Web Acquisition WA-S1–S5 已落地
（Trafilatura、质量门、可选 Tavily / SearXNG、Search/Fetch Replay、case16/case17），免信用卡 Key、
loopback-only 单容器、两种 provider 真实连通与 search → 用户选择 → ingest ReAct dogfood 均已验收。
Local Web 的 LW-S1–S5 与 Web Runtime WR-O1–O4 也已落地：FastAPI 资源/大纲/有界节点、
GroundedDocumentAnswer run、稳定 SSE、取消、精确 citation 与 loopback 启动，“墨迹星图”亮/暗 React
Article/Assessment Workspace，复用 `AssessmentSession` 的逐题考核、可审计 Evidence reveal、幂等提交/下一题，
以及 exact 当前材料、跨轮 Chat cursor、安全实时 `TraceObservatory`，并提供 Markdown/Text 上传、公开 URL
导入、持久状态与跨重启候选审批。v0.1.0 功能 RC 已完成安全 Markdown、Assessment trace 终态、Chat 并发拒绝、
Provider 原生 delta 流式 Chat、turn-scoped 真取消、版本化首次引导、稳定观测投影与确定性 Web Scenario Bot
收口；当前收口为 LW-S7 发布门，LW-S6 完整资源/知识点管理进入 v0.1.0 后 backlog。Learning Model v2
基础闭环也已落地：长期白名单 Journal/outbox、可重建 Attempt、判决纠正/reconciliation、受控词表与分类审核、
LearnerProjection 和稳定审查导出；自动 Demand Judge、Diagnosis/Misconception 仍受 Eval gate 限制。
v0.2 RC 又补齐非代码 Markdown 节点中 CommonMark 可见 Evidence 到 raw source 的唯一映射，以及 Acquisition
`code / stage / reason` 安全错误信封；领域失败会进入 Trace error 统计，并在 CLI/Web 管理态可见。多题考核由
`AssessmentPlan` 统一 CLI/Web/FastAPI 有序题型意图，开放题由 `QuestionSpec` 统一评分点、Evidence、参考答案
和逐点评判；v0.2 功能 RC 已关闭。发布回归又修复了考核导航覆盖 Chat 回复，并为输入框增加空白态 `↑` 恢复
上一问题；默认 Eval/HTML 只做离线 Replay，Rule/Quality 与 execution/judge 成本分列；v0.3 代码 RC 已接通 Web
approved-only 分类筛选、生产判卷人工盲标 calibration gate 与判决纠正到本地 Eval 候选；v0.4 进一步补齐
人工授权的材料发现 inbox、Acquisition 桥接、Eval 隐私审核与内容哈希不可变快照。首批 19 条盲标的生产
校准与固定 10 条开发样本的 Flash/Pro × thinking 2×2 pilot 已完成；Report v2、预注册核心评分点和代码三值
聚合已落地；后续语义判卷收口将已见开发集原始口径提升至 87.50% 逐点准确率 / 66.67% 三值一致率，
人工残差裁决 overlay 后为 89.58% / 75.00%；合成挑战 12/12 通过但强制为 exploratory。v3.1 使用同一
Pro/Thinking Off cohort 真实复测后保持全部逐点决定不变，总 Token 降低 15.41%；真实 cassette 已重录并
通过离线 replay。随后 12 条全新真实 blind holdout 的正式 gate 仅为 68.75% 逐点准确率 / 58.33% 三值
一致率，并有 3 个 serious FN 与 1 个结构化判卷失败，因此自动策略继续关闭；该 Snapshot 已揭盲，只能
转为开发回归集。针对结构失败的 v3.2 补丁已移除 Evidence 的 80 字诱导，明确连续原文且禁止省略号/
拼接，重试会回显非法片段并要求改选；Calibration Report v4 将合法输出率与合法输出上的语义质量分列，
gate 仍要求 100% 合法输出。真实开发回归中最终合法输出率从 91.67% 升至 100%、重试从 2 降至 1，
但首轮 H10 仍使用省略号；统一合法输出口径后新旧逐点准确率均为 75%，不能改变 holdout 失败结论。
随后将自由复制答案 Evidence 收窄为代码生成的唯一句子单元 ID：真实开发实验首轮合法输出 12/12、
重试归零、Token 下降 10.73%，语义指标不变；生产 Grader 已只接受 ID 并由代码解析精确原文。
进一步的 nested rubric prototype 虽修复 H02/H10 并表达 H07 的 OR 边界，却仅 3/4 合法、4 次重试且
Token 比 flat baseline 高 282.77%，因此拒绝引入 Boolean rubric schema，继续使用 flat atomic
ExpectedPoint。该实验还推动 Cassette 对同 replay key 保存有序响应序列，旧单条 fixture 保持可读。
随后补齐答卷 provenance 契约：只有 `unassisted_human` 可进入 release gate，模型/辅助/合成答案在
Compilation 与 Report 中均强制为 exploratory。已在 12 道揭盲 Development Gold 题上用
DeepSeek V4 Pro / Thinking Off 生成 30 条模型答卷（12 完整 / 12 部分 / 6 合理误区），仅作为新的
Synthetic Challenge；13 个录制响应合计 7,665 Token，不污染 Holdout 03 新题源。30 条 assistant
screening 已完成（对 6 / 勉强 12 / 错 12），等待 owner 复核 6 组真正的 rubric 边界；Holdout 03 首批
10 个新 QuestionSpec / 40 个原子评分点已冻结并通过固定源码 Evidence 逐字校验，owner 的 10 条独立
闭卷答卷也已锁定；owner 接受 Codex 的对 3 / 勉强 5 / 错 2 初筛，确定性编译得到 10 条 eligible /
0 excluded，全部为 `unassisted_human`。生产 Grader 尚未运行，且该批仍不足以单独构成 release gate，
第二批 GQ4-H11–H20 的 10 个新 QuestionSpec / 40 个原子评分点也已独立冻结并通过 Evidence、排重和
防泄漏校验；owner 的第二批 10 条闭卷答卷已锁定并接受 Codex 的对 5 / 勉强 5 / 错 0 初筛，第二批
编译同样为 10 eligible / 0 excluded；两批合计为 20 个新
QuestionSpec、80 个评分点、20 条 eligible（对 8 / 勉强 10 / 错 2），排除 0 条。Compilation 已拆开
`question_id` 与独立答卷 `sample_id`；两位朋友又独立完成 10 条自然答卷，owner 终审后正式 cohort 达到
30 条人类答卷 / 20 个 unique QuestionSpec / 120 个逐点评判（对 17 / 勉强 11 / 错 2）。本地隐私审核
冻结的 30 eligible / 0 exploratory Dataset Snapshot
`71a504b0725e41e9992e217de1daf89429f1b126faaa281c7d8822558d306743` 已用 DeepSeek V4 Pro / Thinking Off
运行正式 gate：合法输出 30/30、逐点准确率 90.83%、严重 FN/FP 0/0 均通过，但三值一致率 25/30 =
83.33% 低于 85%，因此 gate 失败。31 个真实响应已录制并通过 30/30 离线语义 replay；该 cohort 已揭盲，
只能作为 Development Gold。随后受限 Required Claims seam 已实现并在 12 条已揭盲 Development Gold 上
真实验证：输出 12/12 合法，但三值仅 8/12、逐点 37/48，新增六个逐点分歧且 Token 比 flat baseline
增加 50.68%，预注册实验失败；因此暂停新 holdout，seam 只作为可审计实验能力，不得描述成已通过的
默认判卷策略。owner rubric audit 后的紧凑 claim 真实实验虽以 18,561 Token 在首阶段解决 4/4 个高影响
目标，但 5 次聚焦复核没有修复错误且新增一个 false positive，使 aligned point 从 37/43 降为 36/43、
三值从 9/12 降为 8/12；按预注册退出条件，Required Claims 默认路线已否决，不再叠加 Judge 或消耗新

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hyr1sky/TheGrandQuiz](https://github.com/Hyr1sky/TheGrandQuiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
