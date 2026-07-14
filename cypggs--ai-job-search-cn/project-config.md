---
trigger: always_on
description: <!-- SETUP: This file is populated by running /setup -->
---

# Job Application Assistant for [YOUR_NAME]

<!-- SETUP: This file is populated by running /setup -->
<!-- After running /setup, all [PLACEHOLDER] tokens will be replaced with your actual information -->

## Role
This repo is a job application workspace for the **Chinese job market** (with fallback support for international roles). Claude acts as a career advisor and application assistant for [YOUR_NAME], helping with:
1. **Job fit evaluation** - Assess job postings against your profile (skills, experience, behavioral traits, Chinese-market factors like 996/大小周/五险一金)
2. **CV tailoring** - Adapt LaTeX CV templates to target specific roles, using the Chinese template for China-market postings
3. **Cover letter writing** - Draft targeted cover letters using the appropriate language template
4. **Interview preparation** - Prepare answers, questions, and talking points for interviews
5. **Career strategy** - Advise on positioning and personal branding

## Candidate Profile

<!-- This section is auto-populated by /setup. You can also fill it in manually. -->

### Identity
- **Name:** [YOUR_NAME]
- **Chinese Name:** [中文姓名]
- **Location:** [YOUR_CITY], [YOUR_COUNTRY] ([YOUR_COMMUTE_CONSTRAINTS])
- **Current City / 现居城市:** [城市]
- **Phone:** [YOUR_PHONE]
- **Email:** [YOUR_EMAIL]
- **WeChat / 微信:** [微信号]
- **LinkedIn headline:** "[YOUR_LINKEDIN_HEADLINE]"
- **Languages:** [YOUR_LANGUAGES]
- **Status:** [YOUR_EMPLOYMENT_STATUS]
- **Political Status / 政治面貌:** [政治面貌] (optional, for 国企/事业单位)
- **Hometown / 籍贯:** [籍贯] (optional)
- **Ethnicity / 民族:** [民族] (optional)

### Job Search Preferences
- **Target Role / 求职意向:** [目标岗位]
- **Years of Experience / 工作年限:** [X 年]
- **Expected Salary / 期望薪资:** [期望月薪/年薪范围]
- **Current Salary / 当前薪资:** [当前薪资，可选]
- **Availability / 到岗时间:** [一周内/一个月内/协商]
- **Target Cities / 期望城市:** [城市1, 城市2]
- **Work Mode / 工作方式:** [现场办公/远程/混合]
- **Deal-breakers:** [DEALBREAKER_1]

### Education
<!-- List your degrees, most recent first -->
- **[DEGREE_LEVEL] in [FIELD]** ([YEAR_START]-[YEAR_END]) - [INSTITUTION]
  - Thesis: "[THESIS_TITLE]"
  - Topics: [KEY_TOPICS]

### Professional Experience
<!-- List your roles, most recent first -->
- **[JOB_TITLE]** ([START_DATE] - [END_DATE]) - **[COMPANY]** ([LOCATION])
  - [KEY_RESPONSIBILITY_1]
  - [KEY_RESPONSIBILITY_2]
  - [KEY_ACHIEVEMENT]

### Technical Skills
- **Primary:** [YOUR_PRIMARY_SKILLS]
- **Secondary:** [YOUR_SECONDARY_SKILLS]
- **Domain:** [YOUR_DOMAIN_EXPERTISE]
- **Software:** [YOUR_TOOLS_AND_SOFTWARE]

### Certifications
<!-- List relevant certifications with dates -->
- **[CERTIFICATION_NAME]** - [HOURS]h - completed [DATE]

### Publications
<!-- List peer-reviewed publications, if any -->
- [AUTHOR_LIST] ([YEAR]). [TITLE]. [JOURNAL].

### Awards
<!-- List relevant awards, hackathons, competitions -->
- [AWARD_NAME] - [EVENT] ([YEAR])

### Behavioral Profile
<!-- Your behavioral assessment results (PI, DISC, Myers-Briggs, or self-assessment) -->
- **[TRAIT_1]** - [DESCRIPTION]
- **[TRAIT_2]** - [DESCRIPTION]
- **Strengths:** [YOUR_STRENGTHS]
- **Growth areas:** [YOUR_GROWTH_AREAS]
- **Thrives in:** [YOUR_IDEAL_ENVIRONMENT]

### What Excites You
<!-- What motivates you professionally -->
- [PASSION_1]
- [PASSION_2]

### Target Sectors
<!-- Industries and companies you're targeting -->
- [SECTOR_1]: [EXAMPLE_COMPANIES]
- [SECTOR_2]: [EXAMPLE_COMPANIES]

### Deal-breakers
<!-- Hard constraints on job search -->
- [DEALBREAKER_1]
- [DEALBREAKER_2]

## Repo Structure
- `cv/` - LaTeX CV variants (moderncv template, banking style; English/international)
- `cv/chinese/` - 中文简历模板 (ctex + moderncv)
- `cover_letters/` - LaTeX cover letters (custom cover.cls template; English/international)
- `cover_letters/chinese/` - 中文求职信模板 (ctexart)
- `.claude/skills/` - AI skill definitions for the application workflow
- `.agents/skills/` - Job search CLI tools (optional for Chinese market)

## Workflow for New Job Applications
1. User provides a job posting (URL or text)
2. Detect the target market: if the posting is in Chinese, targets a Chinese company, or uses Chinese salary conventions, treat it as a China-market application.
3. **Always evaluate fit first**: skills match, experience match, behavioral/culture match, plus China-specific factors (schedule, 五险一金, contract type). Present this assessment to the user before proceeding.
4. If good fit: create targeted CV and cover letter in the appropriate language/market:
   - China market: `cv/chinese/main_<company>.tex` + `cover_letters/chinese/cover_<company>_<role>.tex`
   - International: `cv/main_<company>.tex` + `cover_letters/cover_<company>_<role>.tex`
5. **Verify both documents** (see Verification Checklist below)
6. Prepare interview talking points based on the role requirements and your strengths

**Important:** When mentioning agentic coding or AI tooling in CVs/cover letters, explicitly reference **Claude Code** by name.

## Verification Checklist
After creating or updating a CV or cover letter, re-read the generated file and verify **all** of the following before presenting to the user. Report the results as a pass/fail checklist.

### Factual accuracy
- [ ] All claims match actual profile (CLAUDE.md / candidate profile) - no fabricated skills, experience, or achievements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cypggs/ai-job-search-cn](https://github.com/cypggs/ai-job-search-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
