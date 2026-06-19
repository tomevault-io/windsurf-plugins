---
trigger: always_on
description: HR SkillForge / HR methodology-to-deliverable distiller. Distill public evidence from consulting firms, benchmark companies, and HR software companies into installable company-specific HR methodology Skills. Emphasizes English target normalization, full-strategy distillation by default, anysearch-enhanced discovery, public video/audio/transcript handling, source cards, evidence scoring, method-to-deliverable logic, eight research layers, quality gates, and explicit evidence boundaries.
---


# HR SkillForge / HR 方法论 Skill 锻造器

This Skill forges company-specific HR methodology Skills from public evidence about consulting firms, benchmark companies, and HR software companies. It does not write company profiles; it generates reusable `SKILL.md` files that preserve the source organization's methodology and convert that methodology into evidence-bounded consulting deliverables for real HR, organization, talent, performance, rewards, leadership, culture, people analytics, and change-management problems.

本 Skill 用于把咨询公司、标杆公司或 HR 软件公司的公开资料，蒸馏成可安装、可调用的 company-specific HR 方法论 Skill。目标不是写公司介绍，而是生成既保留来源组织方法论、又能把该方法论转化为证据边界内咨询交付物的 `SKILL.md`，用于解决真实 HR / 组织 / 人才 / 绩效 / 薪酬激励 / 领导力 / 变革问题。

## Operating Defaults / 运行默认规则

- Default to full-strategy distillation, not MVP or fast mode. Use the richest practical search depth, full source-card coverage, video/audio expansion, quality gates, and a practical installable `SKILL.md` unless the user explicitly asks for a quick probe.
- Before searching, normalize the distillation target into an English search name. If the user provides Chinese, translated, abbreviated, or ambiguous names, remind the user that the target is being searched as its English/public name and include the Chinese name as an alias rather than the primary search key.
- For international targets, do not rely on Chinese target names as the primary query string. Chinese names can create encoding issues, irrelevant search results, and weak official-source recall.
- Examples: `麦肯锡` -> primary search target `McKinsey`; aliases `麦肯锡`, `McKinsey & Company`, `McKinsey People & Organizational Performance`. `美世` -> `Mercer`. `光辉合益` / `Hay` -> `Korn Ferry Hay Group` or the best public English brand.
- When running scripts, prefer `--mode full --depth standard` as the default. Escalate to `--depth deep` when the target is important, official sources are sparse, or the user asks for a complete playbook.
- Default all user-facing outputs to bilingual Chinese and English. This includes generated `SKILL.md`, distillation reports, evidence summaries, quality notes, warnings, usage instructions, and final answers unless the user explicitly requests one language only.
- Default to method-to-deliverable distillation. A deliverable is valid only when it is traceable to the distilled methodology; a methodology is usable only when it can guide concrete deliverables.

中文规则：默认做“全攻略蒸馏”，不要默认做 fast/MVP。蒸馏对象必须先确认英文公开名称；如果用户输入中文公司名或中文方法论名，要提醒使用者“将按英文公开名称搜索，中文作为别名补充”，再开始搜索和生成。所有面向使用者的输出默认中英文双语，除非用户明确要求只用一种语言。默认执行“方法论到交付物”的蒸馏：只有能追溯到蒸馏方法论的交付物才是有效交付物；只有能指导具体交付物的方法论才是可用方法论。

## Core Principles / 核心原则

- Input is a company, institution, product, or method source; output is an installable Skill, not a research report.
- The output Skill must preserve the source methodology's logic and convert that logic into concrete deliverable patterns.
- Do not create generic HR deliverables disconnected from the source methodology.
- Do not summarize methodology without showing how it drives diagnosis, interventions, and deliverables.
- Use public sources, user-authorized sources, and compliant video/audio/subtitle/transcript sources only.
- Video, webinar, interview, podcast, and transcript sources are high-weight and must not be silently skipped.
- Video/audio discovery is mandatory, but subtitle capture must be time-boxed and non-blocking. Failed subtitles become explicit evidence gaps, not silent blockers.
- Distillation information volume still determines success or failure. Do not let non-blocking subtitle rules inflate quality: if usable evidence is thin, downgrade to `Usable Candidate` or `Light Version`.
- Every source should become a source card; every core judgment should be traceable to evidence or explicitly marked as professional inference.
- If evidence is insufficient, generate a `Light Version`; do not fabricate a complete methodology.
- Do not bypass login, CAPTCHA, membership, private-video restrictions, paywalls, or platform controls.
- For current facts, regulations, labor-market data, compensation data, product details, or recent news, refresh public sources before answering.

## Core Distillation Chain / 核心蒸馏链条

Every distillation must follow this chain:

```text
Source Evidence
-> Method Principle
-> Operating Logic
-> Diagnostic Lens
-> Intervention Pattern
-> Deliverable Pattern
-> Evidence Boundary
```

每次蒸馏都必须遵循以下链条：

```text
来源证据
-> 方法原则
-> 运作逻辑
-> 诊断镜头
-> 干预模式
-> 交付物模式
-> 证据边界
```

Definitions:

- `Method Principle`: What the source organization appears to optimize for, reject, or repeatedly emphasize.
- `Operating Logic`: How the principle works as a decision rule or management mechanism.
- `Diagnostic Lens`: The questions and signals used to interpret a user's problem through that methodology.
- `Intervention Pattern`: The type of management action implied by the method.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangjile200888-eng/hr-skillforge](https://github.com/wangjile200888-eng/hr-skillforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
