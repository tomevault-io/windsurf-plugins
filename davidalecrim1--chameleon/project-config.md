---
trigger: always_on
description: Chameleon is a Claude Code project that tailors a master resume YAML to a specific job posting, then renders it to PDF using RenderCV. It uses two internal agents to analyze the job description and apply changes to the YAML.
---

# Chameleon — Resume Tailor

Chameleon is a Claude Code project that tailors a master resume YAML to a specific job posting, then renders it to PDF using RenderCV. It uses two internal agents to analyze the job description and apply changes to the YAML.

## Project Goal

Given a job posting URL or pasted job description, produce a tailored, ATS-optimized PDF resume derived from a master YAML file. Each tailored resume is saved as a separate YAML under `templates/` alongside the master.

## Directory Structure

```
chameleon/
├── .claude/
│   ├── skills/
│   │   ├── chameleon/
│   │   │   └── SKILL.md          # Command skill — /chameleon entrypoint
│   │   ├── init-cv/
│   │   │   └── SKILL.md          # Command skill — /init-cv entrypoint
│   │   └── render-cv/
│   │       └── SKILL.md          # Command skill — /render-cv entrypoint
│   └── agents/
│       ├── analyze-job-posting.md         # Subagent — job analysis (isolated context)
│       └── update-cv-with-job-posting.md  # Subagent — resume editor (isolated context)
├── templates/                    # Master and tailored CV YAMLs
│   ├── <name>_cv.yaml             # Master — source of truth, never mutated by tailor runs
│   └── <company>_<role>_cv.yaml   # Tailored — one per job application
├── output/              # RenderCV output — do not commit
└── CLAUDE.md                     # This file
```

## Technology Stack

- **RenderCV** — YAML → PDF renderer (Typst backend, no LaTeX required)
  - Install: `make install-tools`
  - Render: `make render FILE=<file>.yaml`
  - Output: PDF, Markdown, HTML, PNG in `output/`
- **Claude Code skills** — `/chameleon` and `/init-cv` are user-invocable command skills
- **Subagents** — `analyze-job-posting` and `update-cv-with-job-posting` run in isolated contexts

## Skills vs Agents

| Type | What it is | When it runs | Context |
|------|-----------|-------------|---------|
| **Command skill** | Instructions + `disable-model-invocation: true` | Only when user types `/skill-name` | Shared with main conversation |
| **Subagent** | Isolated Claude instance with own system prompt | Spawned by the skill | Own isolated context — returns summary to main |

**`/chameleon` and `/init-cv`** are command skills: user-triggered, not auto-invoked. They orchestrate the workflow and delegate work to subagents.

**`analyze-job-posting` and `update-cv-with-job-posting`** are subagents: spawned by the skill, run in isolation, return a summary. Isolation keeps large intermediate context (raw HTML, full YAML processing) out of the main thread.

## Codex Delegation

For Codex, keep the same two-agent split. When the user explicitly wants delegation or subagents, use `spawn_agent` so the raw JD text and YAML editing work stay out of the main thread.

- Reuse `.claude/agents/analyze-job-posting.md` as the prompt boundary for the analysis subagent. It should receive only the raw JD text and return the structured analysis fields documented below.
- Reuse `.claude/agents/update-cv-with-job-posting.md` as the prompt boundary for the editing subagent. It should receive only the structured analysis plus the resolved master YAML path.
- Do the orchestration, CV selection, rendering, and user-facing reporting in the main thread.
- Do not delegate if the user is only asking questions about the repo or workflow. Delegate when performing an actual tailoring run and isolation helps control context size.

## Skill Workflow (`/chameleon`)

1. **Fetch** the job posting URL or read pasted text
2. **Resolve** which master CV to use from `templates/`:
   - If `--cv` argument provided, use `templates/<name>_cv.yaml`
   - If only one YAML exists in `templates/`, use it automatically
   - If multiple YAMLs exist in `templates/` and none specified, list them and ask the user to choose
3. **Spawn** `analyze-job-posting` subagent with the raw JD text
4. **Receive** structured analysis output
5. **Spawn** `update-cv-with-job-posting` subagent with analysis + resolved master YAML path
6. Subagent saves `templates/<company>_<role>_cv.yaml` and reports the path
7. **Render** via `make render FILE=templates/<company>_<role>_cv.yaml`
8. **Report** the path to the generated PDF

## CV Initialization Workflow (`/init-cv`)

Used when setting up for the first time or when the user provides an updated source resume.

1. Accept a PDF or YAML as input
2. If PDF: parse the content and produce a valid `master_cv.yaml` conforming to RenderCV schema
3. If YAML: validate the structure against RenderCV entry types and save as `master_cv.yaml`
4. Confirm the file renders cleanly: `rendercv render master_cv.yaml`

## Editing Rules (apply to all agents)

These rules are absolute and must never be violated:

- **Never fabricate experience.** Only reword or reorder what already exists in the master YAML. Do not invent companies, roles, dates, metrics, or skills.
- **Preserve all facts.** Company names, job titles, locations, and dates are immutable. Only highlight bullets may be rewritten.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidalecrim1/chameleon](https://github.com/davidalecrim1/chameleon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
