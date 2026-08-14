---
trigger: always_on
description: Instructions for agents creating or editing resumes in this repository.
---

# AGENTS.md

Instructions for agents creating or editing resumes in this repository.

## Project Context

- Resume content lives in `resumes/*.yml`, one file per resume.
- The example resumes (`resumes/*.example.yml`) are the canonical reference for valid shape.
- The schema that validates resumes lives in `lib/src/schema.ts`; `lib/schema.json` is generated from it for editor autocomplete.
- Templates are single Typst files: `templates/<name>.typ`. Pick one with `meta.template`.

Your job in this repo is resume **content**: strong, honest, well-structured bullets that survive a recruiter and a hiring manager. The build tooling is documented in `README.md`.

## YAML Rules

- Use the JSON Resume-compatible shape from `resumes/*.example.yml`.
- Keep top-level content keys aligned with JSON Resume: `basics`, `work`, `volunteer`, `education`, `awards`, `certificates`, `publications`, `skills`, `languages`, `interests`, `references`, `projects`.
- Put build and presentation settings under `meta`.
- Use `meta.template` for the template name without `.typ`; default is `default`.
- Use a real Typst font name in `meta.font`; default examples use `New Computer Modern`.
- Use only letters, digits, `_`, and `-` in `meta.output_filename`.
- Use `meta.sections` to control section order and titles: key order sets render order, a blank value (`work:`) keeps the default title, and unlisted sections render last in the built-in order. Keys must be JSON Resume section names.
- Set `meta.locale` to the BCP 47 locale of the resume language (`en`, `pt-BR`, `es`, etc.). This controls date formatting (`2024-09` → `Sep 2024` in EN, `Set 2024` in PT-BR) and the default present-role label (`Present`, `Atual`, `Actualidad`). Always set it for non-English resumes.
- Use `meta.present_label` only when the locale's default label is wrong for the context.
- Use JSON Resume date strings: `YYYY`, `YYYY-MM`, or `YYYY-MM-DD`. Omit `endDate` for current roles.
- Use `[]` to hide any list-backed section; keep required keys present unless the schema supplies a default.
- Do not add unsupported fields unless `lib/src/schema.ts` and the relevant template are updated together.
- Preserve Markdown-style emphasis only where useful: `**bold**` and `_italic_`.
- Keep the `# yaml-language-server: $schema=../lib/schema.json` comment in example resumes.

## STAR Method

Every meaningful experience or project bullet must be grounded in STAR:

- Situation: what problem, product, team, system, customer, or constraint existed.
- Task: what the candidate owned or was expected to solve.
- Action: what the candidate personally did, including tools, decisions, and methods.
- Result: what changed after the work.

Final bullets should usually follow this pattern:

```text
Action verb + specific work + context or scope + measurable or observable result.
```

Good:

```yaml
- Refactored payment reconciliation jobs in **Python** and **PostgreSQL**, reducing daily manual review time from 3 hours to 40 minutes.
```

Weak:

```yaml
- Worked on backend improvements and helped the team become more efficient.
```

If the source material does not contain a real result, do not invent one. Ask for the missing evidence or write an honest scope-based bullet.

## Evidence Standards

- Prefer outcomes over responsibilities.
- Prefer concrete scope over generic seniority claims.
- Use metrics when they are real: percentages, time saved, revenue, cost, latency, adoption, volume, team size, tickets, users, requests, or error reduction.
- If exact metrics are unavailable, ask for a defensible approximation.
- If no metric exists, use an observable result without pretending it is quantified.
- Never fabricate employers, dates, titles, tools, projects, metrics, or credentials.

Acceptable non-metric result:

```yaml
- Standardized onboarding documentation for the support team, replacing scattered notes with a single process used for new analyst training.
```

## Anti-AI Writing Rules

Resume prose must sound specific, human, and confirmable.

Avoid:

- Generic summaries such as "results-driven professional" or "proven track record".
- Inflated language such as "visionary", "dynamic", "world-class", or "best-in-class".
- Corporate filler such as "leveraged synergies", "stakeholder ecosystem", or "cross-functional excellence".
- Passive responsibility bullets starting with "Responsible for", "Tasked with", or "Involved in".
- Vague verbs such as "helped", "supported", "handled", or "worked on" unless the contribution was truly secondary.
- AI-style constructions such as "not only X but also Y", "in today's fast-paced environment", "the ability to", and "the ever-evolving world of".

Prefer:

- Direct verbs: built, shipped, led, migrated, automated, reduced, increased, designed, launched, consolidated, analyzed, mentored.
- Plain language that a former manager would recognize.
- Specific nouns: product name, system type, customer segment, process, team, market, repository, service, dashboard, workflow.
- Short bullets with one clear claim each.

## Resume Content Workflow

1. Identify the target role, language, geography, seniority, and resume length.
2. Extract evidence from the user-provided material before rewriting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gustavo-ferreira03/resume-ci](https://github.com/gustavo-ferreira03/resume-ci) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
