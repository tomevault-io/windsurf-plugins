---
trigger: always_on
description: Build and run a multi-agent workflow for Korean job applications, especially self-introduction essays and cover letters. Use when users ask to analyze a target company/role, extract and map personal experiences, draft application answers, rewrite tone for a specific company, detect AI-like writing patterns, and run final submission checks.
---


# Job Application Agent Suite

## Overview

Use this skill to execute a full writing pipeline for job applications.
Run eleven specialized agents in order so each output becomes the next agent's input.

You can also run any agent independently when the user requests partial work.

## Invocation UX

If user says only "agent를 불러줘" (or equivalent), use progressive intake in this order:

1. Ask only company/role (+ posting link/text if possible) and run company-role analysis first.
2. Ask application question(s) and character limits.
3. Ask candidate experience details last.

Do not ask for candidate experience in step 1.

Use these staged templates:

1) Stage-1 (company/role first)

```json
{
  "company_name": "네이버",
  "role_title": "백엔드 개발자",
  "job_posting_text_or_link": "https://careers.example.com/naver/backend"
}
```

2) Stage-2 (questions after baseline analysis)

```json
{
  "application_questions": [
    {
      "question_id": "Q1",
      "question_text": "지원 동기와 입사 후 기여 방안을 작성해 주세요.",
      "char_limit": 800
    }
  ]
}
```

3) Stage-3 (candidate evidence last)

```json
{
  "candidate_profile": "백엔드 3년차, Java/Spring, 대용량 트래픽 서비스 운영 경험",
  "experience_notes": "결제 API 병목 개선(응답속도 35% 단축), 장애 대응 자동화 경험",
  "forbidden_claims_or_constraints": ["확인되지 않은 수치 사용 금지", "기밀 정보 노출 금지"]
}
```

Optional shortcut when user already has a draft:

4) Existing draft path (`agent-06 -> agent-07`)

```json
{
  "question_text": "지원 동기와 입사 후 기여 방안을 작성해 주세요.",
  "char_limit": 800,
  "final_draft": "저는 사용자 문제를 기술로 해결하는 과정에서 동기를 얻습니다. 이전 회사에서 결제 API 병목을 분석해 응답 시간을 35% 단축한 경험이 있습니다..."
}
```

## Workflow

1. Run the intake-validator agent.
2. Run the analysis baseline in this order:
   - `agent-01a` industry analyst
   - `agent-01b` company analyst
   - `agent-01c` role analyst
   - `agent-01` synthesis analyst
3. Run the corpus-experience recommender agent.
   - must report "what was scanned" and "what will be used" to user
4. Run the experience miner agent.
5. Run the question-experience matcher agent.
6. Run the experience-refiner agent (tail-question/refinement gate).
7. Run the metric-evidence tracker agent.
8. Run the drafting assistant agent.
9. Run the tone-customizer agent.
10. Run the AI-style reviewer and final QA checker.
11. Collect external AI feedback and write it to `external_feedback_notes`.
12. Run the final-packager agent.

## Default Persistence

Persist analysis outputs by default.

Unless the user explicitly says not to save files, save analysis artifacts automatically.

Default save rules:

- company-only analysis:
  - `~/job_runs/<company_slug>/company_analysis.md`
- company + role analysis:
  - `~/job_runs/<company_slug>/<role_slug>/01_company_role_analysis.md`
- industry-only analysis:
  - `~/job_runs/<company_slug>/<role_slug>/01a_industry_analysis.md`
- company-only step inside a multi-step run:
  - `~/job_runs/<company_slug>/<role_slug>/01b_company_analysis.md`
- role-only step inside a multi-step run:
  - `~/job_runs/<company_slug>/<role_slug>/01c_role_analysis.md`

Persistence rules:

- If no run directory exists, create it automatically.
- Save the human-readable markdown result first.
- If the response contains structured lists/tables worth reusing, preserve them in the markdown file rather than dropping them.
- Tell the user where the file was saved.
- Only skip saving if the user explicitly asks for chat-only output.

Language rules for saved artifacts:

- Default saved analysis artifacts should be written in Korean.
- If the user explicitly requests English output, the saved artifact may be written in English.
- For Korean job-application workflows, prefer Korean headings, Korean summaries, and Korean writing hooks by default.

## Company Research Enforcement

When running `agent-01a`, `agent-01b`, `agent-01c`, or `agent-01`, treat the following company-research checks as mandatory, not optional.

Primary reference:

- `references/company-research-checklist.md`
- `references/company-research-checklist.json`

Always load and follow this file before or during company-analysis work.

Required company investigation scope:

- main business and core competitiveness
- industry position and strengths
- latest business focus, new business, and investment direction
- competitor comparison and differentiation
- company culture, work environment, and welfare/support systems

Verification rules (must enforce):

- Check the latest official homepage, careers page, newsroom, and IR/investor materials when available.
- If recent reporting is used, separate verified fact from interpretation.
- Do not state an ambiguous or weakly supported point as fact.
- If something is outdated, unclear, or cannot be confirmed, explicitly say it is not accurate enough to confirm.
- Prefer official and recent sources over generic summaries or stale blog content.
- If a user provides a custom company-research checklist later, merge it with this file instead of silently replacing this baseline.

Company-analysis output expectations:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HyunWoo9930/job-application-agent-suite](https://github.com/HyunWoo9930/job-application-agent-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
