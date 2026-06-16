---
trigger: always_on
description: Generates a full application package for one job. LLM calls run sequentially.
---

# findajob — CLAUDE.md

Read by Claude Code at the start of every session. Authoritative context for this codebase.
Personal identifiers (name, targets, API topic, form URLs) live in `CLAUDE.local.md` (gitignored).

---

## Self-Governance — Check Before Every Command

Before writing any command, path, binary call, or file location:

- [ ] All binary paths come from `findajob.paths` — `PANDOC`, `BASE`. Never hardcode.
- [ ] For subprocess calls to other pipeline scripts, use `sys.executable` (never a hardcoded Python path).
- [ ] LLM calls go through `findajob.llm.openrouter.complete()`. Never re-introduce a subprocess transport.

**If uncertain about any value: say so. Do not guess.**

---

## PII and Domain-Neutrality

The repo is public. Tracked files must not contain personal identifiers (real names, emails, API keys) or content that locks the pipeline to one career field. The actual enforcement layer is `.git/hooks/pre-commit` — see `docs/operations/config-reference.md` for setup. The hook is not tracked; each clone installs its own and extends `PATTERNS` when new identifiers appear.

Two categories the hook can't fully catch — be deliberate about these:

- **Operator topology** — hostnames, deployment paths (`/opt/stacks/...`), backup destinations, consumer infra brand names (hypervisor / NAS / VPN mesh products), per-stack port numbers, the operator's domain. Setup docs use placeholders: `<deployment-host>`, `<operator-handle>`, `<operator-domain>`.
- **Field-locked content** — hardcoded company lists, single-field title patterns, industry vocabulary in role prompts. Belong in gitignored config (`config/target_companies.md`, `config/prefilter_rules.yaml`) or referenced from the candidate profile, not enumerated in tracked files. Tracking doc: [`docs/maintainers/generalization.md`](docs/maintainers/generalization.md).

Plans, specs, and experiment notes under `docs/superpowers/` are gitignored (#430). Stay off the index even for "just this PR." Plan-content conventions are documented in [`## Plan Structure`](#plan-structure) below; the *storage* is operator-private.

If you find yourself wanting to put a real name, real employer, real city, or a tech-only example into a tracked file: move it to `CLAUDE.local.md` or a gitignored config and reference it instead.

---

## Pipeline Context

The pipeline is Docker-only: image `ghcr.io/brockamer/findajob`, supercronic + uvicorn co-process inside one container, paths under `/app/...` (override via `JSP_BASE`). All scripts use `findajob.paths.BASE` — never hardcode `/home/...` or `/app/`. All LLM calls go through `findajob.llm.openrouter.complete()`. Per-role model assignments, plumbing, and container path shifts are in [`## Per-Role Model Assignments`](#per-role-model-assignments) below.

---

## Per-Role Model Assignments

<!-- Absorbed from docs/maintainers/pipeline-context.md, 2026-05-27 -->

Model assignment for the core pipeline and interview-materials roles, plus the canonical paths and conventions the pipeline depends on. Read when working on a specific role, fetcher, or path question.

### Models per role

| Role | Model | Notes |
|------|-------|-------|
| Default | `openrouter:google/gemini-3-flash-preview` | |
| `job_scorer` | `openrouter:deepseek/deepseek-v3.2` | profile.md injected directly; `--rag` NEVER used |
| `resume_tailor` / `cover_letter_writer` | `openrouter:anthropic/claude-opus-4.8` | `max_tokens: 4096` |
| `briefing_writer` | `openrouter:anthropic/claude-opus-4.8` | cascades into `resume_tailor` + `cover_letter_writer` |
| `outreach_drafter` | `openrouter:anthropic/claude-opus-4.8` | profile + voice samples injected directly |
| `recruiter_critic` | `openrouter:anthropic/claude-opus-4.8` | `max_tokens: 1024`; sees company, title, JD, tailored resume, cover; NOT profile/briefing/fit |
| `interview_prep` | `openrouter:anthropic/claude-opus-4.8` | `max_tokens: 4096`; fires on `applied → interview` |
| `study_guide_generator` / `flashcard_generator` | `openrouter:anthropic/claude-sonnet-4.6` | on-demand interview-study artifacts from `/materials/{fp}/` (#873/#1029) |
| `podcast_scriptwriter` | `openrouter:anthropic/claude-opus-4.8` | interview-prep podcast script artifact |
| `company_discoverer` | `openrouter:perplexity/sonar-reasoning-pro` | weekly Sun 02:00; emits `candidate_context/discovered_companies.md` + `.json`; field-agnostic, augments static `## Target Companies` |
| `company_researcher` | `openrouter:perplexity/sonar-reasoning-pro` | |
| `fit_analyst` | `openrouter:perplexity/sonar-reasoning-pro` | appended to company briefing |
| `candidate_led_briefing` | `openrouter:perplexity/sonar-deep-research` | async (1–5 min); drives speculative briefing pass |
| `speculative_roles_synth` | `openrouter:anthropic/claude-sonnet-4.6` | `max_tokens: 4096`; synthesizes 1–5 candidate-tailored role cards |
| `resume_change_reviewer` / `network_analyst` | `openrouter:google/gemini-3-flash-preview` | |

> Auxiliary/meta roles (`onboarding_interviewer`, `voice_processor`, `recall_auditor`, `loose_ends_*`) carry their own `model:` frontmatter in `config/roles/` and are intentionally not enumerated here.

### Pipeline plumbing

| Item | Value |
|------|-------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brockamer/findajob](https://github.com/brockamer/findajob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
