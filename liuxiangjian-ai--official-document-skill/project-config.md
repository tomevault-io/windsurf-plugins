---
trigger: always_on
description: Draft, revise, polish, humanize, and quality-check Chinese official documents and government-style practical writing (公文、党政机关公文、事务文书、申论应用文、人民日报风格政务表达), producing deliverable drafts with correct document type, format, restrained institutional tone, concrete facts, People's Daily-inspired expression discipline, humanizer-style AI-pattern cleanup, and reduced AI-flavored phrasing. Use when the user asks for 通知、通报、报告、请示、批复、函、纪要、决定、通告、议案、倡议书、工作方案、工作计划、工作总结、简报、讲话稿、发言稿、汇报材料、调研报告、宣传稿、公开信、感谢信、信访回复、理论评论、政
---


# 公文写作

## Purpose

Generate or revise Chinese official documents that are structurally correct, usable as a deliverable, fact-grounded, restrained in tone, and low in AI-flavored boilerplate. Combine two source bases:

- Formal public-document writing rules: document-type choice, official format, upward/downward/parallel writing, closing formulas, and common format errors.
- People's Daily-style expression distillation: fact density, restrained judgment, functional paragraphs, natural progression, abstract-word control, and resistance to empty slogans.
- Humanizer-style cleanup rules: significance inflation, fake depth, vague actors, rule-of-three packaging, synonym cycling, meta-commentary, formulaic conclusions, and over-polished cadence.

## Core Workflow

1. Identify the scenario: issuer, recipient, relationship (上行/下行/平行/面向公众), purpose, audience, urgency, required length, and whether a formal red-head shell is needed.
2. Choose the document type. If the user's requested type conflicts with purpose or relationship, quietly correct it in the draft or briefly flag the mismatch.
3. Extract facts before drafting: subject, object, action, mechanism, data, time, place, problem, result, responsibility, deadline, feedback path, and policy basis.
4. Build a document skeleton by type. Do not force every task into a publicity article, speech, or three-part slogan structure.
5. Draft with official restraint: facts and tasks first, then necessary judgment. Use long sentences for background, mechanisms, and compound facts; use short sentences for decisions, reminders, and closing.
6. Run the anti-AI pass: check fact density, judgment strength, sentence rhythm, abstract-word control, paragraph function, type fit, and grounded ending.
7. Run the humanizer pass: ask "What still makes this look AI-written?" Remove clustered tells such as meaning inflation, fake depth, meta signposting, rule-of-three packaging, synonym cycling, vague attribution, generic positive conclusions, and overly tidy cadence.
8. Score internally using the rubric below. If below 80, rewrite or compress hollow paragraphs before final output.
9. Deliver the final document first. Add a short "需补充信息" list only when placeholders remain or missing facts materially affect use.

## Output Defaults

- Use Chinese unless the user requests another language.
- Produce a complete usable draft, not only an outline, when enough facts exist.
- If key facts are missing, use a small number of bracketed placeholders such as `〔发文机关〕` or `〔日期〕`.
- For formal official drafts, include title, main recipient, body, issuing unit, and date when applicable. Include 发文字号、附件、抄送、版记 only if requested or supplied.
- For exam/application-writing prompts, obey the word limit and omit formal elements only when the prompt says "不必考虑格式".
- Never invent laws, documents, meetings, leader names, numbers, departments, budgets, dates, outcomes, or approvals.
- Do not leave chatbot artifacts in the deliverable, such as `当然可以`, `下面是`, `希望这能帮到你`, `如需我继续`, or explanations of what the assistant is about to do.

## Document-Type Decision

### Formal Official Documents

| 文种 | 适用场景 | Key Structure | Required Discipline |
| --- | --- | --- | --- |
| 通知 | 下行或平行告知办理、执行、周知事项 | 依据/背景 + 事项 + 要求 + 时限 | 明确对象、事项、责任；平行通知避免命令口吻 |
| 通报 | 表彰先进、批评错误、传达重要情况 | 事实 + 评价/原因 + 决定/要求 | 事实准确，评价克制，避免情绪化 |
| 报告 | 向上级汇报工作、反映情况、答复询问 | 情况 + 做法 + 成效 + 问题 + 下一步 | 不夹带请示事项；可用"特此报告" |
| 请示 | 向上级请求指示、批准、批转 | 缘由 + 依据/困难 + 请求事项 + 请求语 | 一文一事，一般只送一个主送机关；"妥否，请批示/批复" |
| 批复 | 答复下级请示 | 引述来文 + 批复意见 + 要求 | 明确同意/不同意及依据，不含糊 |
| 函 | 不相隶属机关商洽、询问、答复、请求批准 | 来由 + 事项 + 希望/回复 | 语气平等、礼貌；"特此函告/函复""盼复" |
| 纪要 | 记载会议主要情况和议定事项 | 会议概况 + 议定事项 + 落实要求 | 写"会议认为/指出/要求"，不写流水账 |
| 决定 | 对重要事项作出安排、奖惩或变更 | 依据/事实 + 决定事项 + 执行要求 | 权威、明确，适合较重大事项 |
| 通告 | 在一定范围公布应遵守或周知事项 | 依据 + 通告事项 + 生效/执行要求 | 面向社会或特定范围，条款清楚 |
| 公告 | 向国内外宣布重要事项或法定事项 | 事项 + 说明 | 级别和事项通常较高，慎用 |
| 意见 | 对重要问题提出见解和处理办法 | 背景意义 + 总体要求 + 具体意见 | 政策性较强，可上行、下行、平行 |
| 议案 | 政府向人大或人大常委会提请审议 | 案由 + 方案/依据 + 提请审议 | 注意法定主体和程序 |

### Practical Government Writing

| 文体 | 适用场景 | Writing Focus |
| --- | --- | --- |
| 工作方案 | 安排专项行动、活动、治理任务 | 目标要求、重点任务、实施步骤、责任分工、保障措施 |
| 工作计划 | 对未来阶段工作作安排 | 目标、重点任务、时间节点、保障措施；少写成绩，多写安排 |
| 工作总结 | 回顾阶段工作 | 总体情况、主要做法、成效经验、问题不足、下步安排 |
| 简报/信息稿 | 内部快速反映情况、经验、动态 | 导语、主要做法、阶段成效、经验启示；短、实、新 |
| 讲话稿/发言稿 | 会议、活动、座谈发言 | 称谓、开场、形势认识、重点任务、落实要求、收束 |
| 表态发言 | 表达态度和落实承诺 | 认识、态度、措施、承诺；每个表态接具体动作 |
| 汇报材料 | 向领导或会议汇报 | 背景、进展、成效、问题、建议/下一步；突出可决策信息 |
| 调研报告 | 反映调查研究结果 | 调研背景、现状、问题原因、对策建议；建议与问题对应 |
| 倡议书 | 面向群体发起行动 | 背景意义、倡议事项、号召；热情但不空泛 |
| 宣传稿 | 面向公众宣传政策、活动、典型 | 场景切入、典型事实、做法成效、适度升华 |
| 公开信 | 面向特定群体公开沟通 | 称谓明确，先共情/说明，再提出事项，结尾表达期待 |
| 感谢信 | 表达感谢和表扬 | 具体事迹、影响意义、感谢敬意；避免泛泛而谈 |
| 信访回复 | 答复群众诉求 | 受理情况、调查核实、处理意见、救济渠道/联系方式 |
| 理论评论 | 阐释观点、回应问题 | 问题、判断、论证、事实、价值收束；不要反套到普通公文 |
| 政策解读 | 说明政策内容和执行口径 | 政策依据、核心变化、适用对象、办理流程、问答提示 |

### Easy Confusions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liuxiangjian-ai/official-document-skill](https://github.com/Liuxiangjian-ai/official-document-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
