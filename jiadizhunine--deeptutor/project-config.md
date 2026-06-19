---
trigger: always_on
description: >
---


# DeepTutor — Academic Advisor Investigation System

> **Multi-platform skill.** This `SKILL.md` is the entry for Claude Code / agentskills.io. For Codex CLI, OpenCode, OpenClaw, Aider, Cline, Continue, and any other tool following the [agents.md spec](https://agents.md/), the equivalent entry point is the repo-root `AGENTS.md`. Cursor reads `.cursor/rules/deeptutor.mdc`. All three documents describe the same workflow; updates should be propagated to `AGENTS.md` when changing the workflow itself.

## Core Principle

> **Your ceiling = your seniors' ceiling.**

Student outcomes are the single most predictive signal for advisor quality. A professor with stellar publications but whose students consistently end up in unclear positions is a red flag. A professor with modest metrics but whose students thrive is gold. Always weight student trajectory evidence above all other dimensions.

## Language & Region Detection

### Language Rule
Respond in whatever language the user writes in. If the user writes in Chinese, the entire report — titles, analysis, recommendations — must be in Chinese. If in English, everything in English. If in Japanese, Korean, or any other language, follow that language throughout. Never mix languages within a report unless quoting original source text.

### Region Detection
Determine region from the institution name. This affects which search platforms to use and which evaluation criteria apply.

| Region | Institutions | Strategy |
|--------|-------------|----------|
| **Mainland China** | Any university/institute in 中国大陆 | Chinese strategy → `references/chinese_academic_system.md` |
| **International** | US, EU, UK, Japan, Korea, Australia, Singapore, etc. | International strategy → `references/international_academic_system.md` |
| **Hong Kong / Macau / Taiwan** | HKU, CUHK, HKUST, NTU, NTHU, etc. | Hybrid — use both Chinese social platforms AND international academic platforms |

When uncertain about region, ask the user.

## Input Requirements

**Minimum input**: Professor name + institution name.

If the user hasn't provided these, ask:

1. **Career goal** (shapes the Goal-Advisor Match scoring dimension)
   - **Chinese context**: 读博深造 / 考公考编 / 进大厂 / 药企CRO / 进医院 / 纯拿学位
   - **International context**: Academic career (tenure-track) / Industry R&D / Consulting & Finance / Government & Policy / Startup / Just get degree
2. **Risk tolerance**: Conservative / Moderate / Aggressive
3. **Specific concerns** (optional): e.g., "I heard the lab has high turnover"

If the user doesn't provide career goal or risk tolerance, proceed with a balanced evaluation and note that the Goal-Match dimension couldn't be fully scored.

---

## Model Capability Detection & Version Selection

DeepTutor has two investigation modes. The right mode depends on the model running it.

### Auto-Detection Rule

**Full Version (完整版)** — run without asking:
- Claude Opus 4.6+ / Claude Sonnet 4.6+
- GPT-5 / GPT-5-Codex (powering Codex CLI) and equivalent
- Gemini 2.5 Pro / Gemini 3 and equivalent
- Future flagship models of equivalent or higher capability

**Prompt user to choose** — for all other models (GPT-4o, Gemini Flash, GLM, MiniMax, Claude Haiku, smaller open models, etc.), display:

> ⚠️ **DeepTutor 模式选择**
> 检测到当前模型非旗舰级别。
> - **完整版**: 10阶段/11维度/18节报告（推荐高端模型）
> - **轻量版**: 6阶段/7维度/7节报告（Token约完整版40%，可能遗漏部分信息）
> 请选择：完整版 or 轻量版？

If unsure of the running model's class, default to prompting rather than silently running Full — a Lite report from a weaker model beats a hallucinated Full report.

### Lite Version: 6-Phase Workflow

If the user chooses Lite, read `references/lite_mode.md` for the full specification. Key differences:
- **6 phases** (skip co-author network, funding analysis, macro trend deep dive, retirement risk)
- **7 scoring dimensions** (merge and drop 4 dimensions, re-weight)
- **Simplified Sharp Critique** (5-line template instead of 7-question framework)
- **7-section report** (instead of 18)

### Report Generation (Both Versions)

Both Full and Lite versions should output structured JSON and use `scripts/generate_report.py` for HTML rendering:

```bash
# Model outputs investigation data as JSON → script renders HTML
python scripts/generate_report.py report_data.json -o report.html
```

This separates investigation (model's job) from rendering (script's job). Even Full version benefits from this — the model focuses on analysis, not wrestling with CSS.

---

## 10-Phase Investigation Workflow (Full Version)

### Phase 1: Identity Resolution

Establish the professor's verified identity across platforms. This prevents investigating the wrong person (especially common with Chinese names that have many romanization variants).

**For all regions:**
- Official faculty page (university website)
- Google Scholar profile
- Scopus Author ID / ORCID
- Semantic Scholar

**Chinese-specific additions:**
- Baidu Scholar (百度学术)
- ResearchGate
- X-MOL faculty profile
- NSFC funded project database (kd.nsfc.cn)
- ScholarMate

**International-specific additions:**
- DBLP (for CS)
- Web of Science ResearcherID
- Personal/lab website
- GitHub (for computational fields)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jiadizhunine/deeptutor](https://github.com/jiadizhunine/deeptutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
