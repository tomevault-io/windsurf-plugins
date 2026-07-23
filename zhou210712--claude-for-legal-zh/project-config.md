---
trigger: always_on
description: CONFIGURATION LOCATION
---

<!--
CONFIGURATION LOCATION

User-specific configuration for this plugin lives at a version-independent path that survives plugin updates:

  ~/.claude/plugins/config/claude-for-legal/ai-governance-legal/CLAUDE.md

Rules for every skill, command, and agent in this plugin:
1. READ configuration from that path. Not from this file.
2. If that file does not exist or still contains [PLACEHOLDER] markers, STOP before doing substantive work. Say: "This plugin needs setup before it can give you useful output. Run /ai-governance-legal:cold-start-interview — it takes about 10-15 minutes and every command in this plugin depends on it. Without it, outputs will be generic and may not match how your practice actually works." Do NOT proceed with placeholder or default configuration. The only skills that run without setup are /ai-governance-legal:cold-start-interview itself and any --check-integrations flag.
3. Setup and cold-start-interview WRITE to that path, creating parent directories as needed.
4. On first run after a plugin update, if a populated CLAUDE.md exists at the old cache path
   (~/.claude/plugins/cache/claude-for-legal/ai-governance-legal/<version>/CLAUDE.md for any version)
   but not at the config path, copy it forward to the config path before proceeding.
5. This file (the one you are reading) is the TEMPLATE. It ships with the plugin and shows the
   structure the config should have. It is replaced on every plugin update. Never write user data here.

**Shared company profile.** Company-level facts (who you are, what you do, where you operate, your risk posture, key people) live in `~/.claude/plugins/config/claude-for-legal/company-profile.md` — one level above this file, shared by all 12 plugins. Read it before this plugin's practice profile. If it doesn't exist, this plugin's setup will create it.
-->

# AI 治理实务画像

*由冷启动访谈编写。在此之前，这是模板——如看到
`[PLACEHOLDER]`，运行 `/ai-governance-legal:cold-start-interview`。*

---

## 公司画像

[公司] 是一家 [描述——公司做什么、客户是谁]。 *(来自 company-profile.md——编辑那里以跨所有插件修改)*

**AI 角色：** *不在公司层面设定。* 在中国 AI 监管框架下，角色（AI 服务提供者、部署者、分发者等）**按 AI 系统评估**——参见下方 `## AI 系统清单`。一个组织可以是一个系统的服务提供者、又是另一个系统的部署者；单一公司层面的标签产生错误答案。

**AI 活动摘要：** [PLACEHOLDER —— 概述 AI 如何触及公司的段落：你是否构建、部署、消费供应商 AI、训练模型或某种组合。这仅是方向性描述。权威的逐系统分类位于 `ai-systems.yaml`。]

**监管覆盖范围：** [PLACEHOLDER —— 仅列实际适用的。生成式人工智能服务管理办法 / 科技伦理审查办法 / 算法推荐管理规定 / 行业特定要求 / 仅合同要求。如尚无适用的，说明。] *(来自 company-profile.md——编辑那里以跨所有插件修改)*

**未结监管事项：** [PLACEHOLDER]

**外部承诺：** [PLACEHOLDER —— 自愿 AI 承诺、公开 AI 原则页面、透明度报告——或无]

**执业场景：** [PLACEHOLDER — 独立执业/小型律所 | 中型/大型律所 | 企业法务 | 政府/法律援助/诊所] *(来自 company-profile.md——编辑那里以跨所有插件修改)*

---

## 谁在使用

**角色：** [PLACEHOLDER — 律师 / 法律专业人士 | 非律师有律师对接 | 非律师无律师对接]
**律师联系人：** [PLACEHOLDER — 姓名 / 团队 / 外部律所 / N/A]

---

## 可用集成

| 集成 | 状态 | 不可用时的回退 |
|---|---|---|
| 文档存储（Google Drive / SharePoint / 飞书文档） | [✓ / ✗] | 手动文件路径；输出本地保存 |
| 定时任务 | [✓ / ✗] | 政策监测扫描仅在按需模式下运行 |
| 飞书/Slack | [✓ / ✗] | 升级和通知通过邮件发送 |

*重新检查：`/ai-governance-legal:cold-start-interview --check-integrations`*

---

## 应用场景登记册

*从访谈中提取。随新应用场景的出现添加。*

| 应用场景 | 已批准 | 条件 / 要求 | 不可——原因 |
|---|---|---|---|
| [PLACEHOLDER] | | | |

### 红线

以下为自动拒绝，不论请求如何包装：

- [PLACEHOLDER — 红线 1 及原因]
- [PLACEHOLDER — 红线 2 及原因]

### 治理层级

| 风险层级 | 审批路径 | 示例应用场景 |
|---|---|---|
| 标准 | [PLACEHOLDER] | 内部生产力工具、辅助性起草 |
| 升级 | [PLACEHOLDER — 需法律/个人信息保护审查] | 面向客户的 AI、人力资源应用 |
| 高风险 | [PLACEHOLDER — 高管层或董事会] | 重大自动化决策、生物识别 |

---

## AI 系统清单

**清单文件：** `~/.claude/plugins/config/claude-for-legal/ai-governance-legal/ai-systems.yaml`

在中国 AI 监管框架下，**角色和风险层级按 AI 系统评估，不按公司。** 一个组织可以是系统A的提供者、系统B的部署者——每个组合触发不同的义务集合。本清单每条记录对应一个系统。

每条记录承载：
- `role` —— 服务提供者 / 部署者 / 分发者 / 其他角色
- `role_basis` —— 该角色适用原因的一句话解释，标注 `[verify against current regulatory text]`
- `tier` —— 高风险 / 有限风险 / 低风险 / 通用人工智能 / 通用人工智能系统级
- `tier_basis` —— 匹配到的监管风险分类依据，标注 `[verify against current regulatory text]`
- `cn_nexus` —— 系统是否具有中国关联（部署、提供或影响中国境内人员）
- `obligations_note` —— 关于需评估哪些义务的简短说明；非衍生表格
- `next_review` —— 重新分类的日期和触发条件

**清单不自动推导义务。** 当用户问"系统X我的义务是什么？"时，答案在对话中生成，标注 `[verify]`。这是有意为之——监管映射复杂，制度在持续完善中，硬编码的角色 × 层级 → 义务表格正是那种出现在董事会备忘录中的自信但错误的产物。清单是律师的登记册；律师拥有义务分析。

管理清单使用 `/ai-governance-legal:ai-inventory` —— `list | add | edit <id> | classify <id> | show <id>`。

---

## 影响评估内部风格

**触发条件：** [PLACEHOLDER —— 什么需要影响评估。参照生成式人工智能服务管理办法、科技伦理审查办法等要求]

**格式：** [PLACEHOLDER —— 来自种子影响评估的结构，或如未提供使用基线结构]

**深度：** [PLACEHOLDER —— 典型长度和详细程度]

**签批：** [PLACEHOLDER —— 谁批准]

**模板结构：**

[PLACEHOLDER —— 从种子影响评估提取的章节标题。如未提供种子文件，在完成首次评估后替换本节。]

---

## 供应商 AI 治理

### 我们对 AI 供应商的要求

| 条款 | 我们的标准 | 可接受的回退 | 决不 |
|---|---|---|---|
| 数据使用 | [PLACEHOLDER] | | |
| 可审计性 | [PLACEHOLDER] | | |
| AI 输出责任 | [PLACEHOLDER] | | |
| 事件通知 | [PLACEHOLDER] | | |
| 人工审查权 | [PLACEHOLDER] | | |
| 模型变更通知 | [PLACEHOLDER] | | |

### 底线条款

[PLACEHOLDER —— 自动拒绝的 AI 供应商条款]

---

## AI 政策承诺

*从 [政策名称 / URL] 于 [日期] 提取*

**已述明禁止用途：** [PLACEHOLDER]
**已述明必要保障措施：** [PLACEHOLDER]
**披露义务：** [PLACEHOLDER —— 政策关于向客户、员工或受影响方披露AI使用的规定]
**已批准供应商/工具：** [PLACEHOLDER —— 列表或"维护在许可清单中"]
**已禁止供应商/工具：** [PLACEHOLDER —— 列表或"维护在禁止清单中"]

---

## 治理团队和升级

**团队：** [PLACEHOLDER —— N人，AI治理在组织中的位置]
**供应商关系负责人：** [PLACEHOLDER]
**AI 风险负责人：** [PLACEHOLDER —— CISO / 个人信息保护负责人 / 总法顾问 / 专职角色]

| 事项 | 处理层级 | 升级至 | 何时 |
|---|---|---|---|
| 新应用场景——标准 | [PLACEHOLDER] | | 风险层级模糊 |
| 新应用场景——升级 | | [总法顾问] | 超出已批准类别 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhou210712/claude-for-legal-ZH](https://github.com/zhou210712/claude-for-legal-ZH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
