---
trigger: always_on
description: This repository stores Martin's self-built, reusable skills for cross-PC and cross-project sharing.
---

# AGENTS.md

## Project Purpose

This repository stores Martin's self-built, reusable skills for cross-PC and cross-project sharing.

The goal is portability, reuse, and safe publication. It is not a dumping ground for private workspace state.

## Working Principles

- Make the smallest necessary change
- Do not revert unrelated work
- Prefer reusable structure over project-specific hacks
- Sanitize before publishing
- Keep the repository easy to copy into other projects

## Scope

Only include skills that Martin built or substantially shaped by hand.

Current in-scope skills:

- `audio-transcriber`
- `brand-guidelines`
- `cloudflare-r2-publisher`
- `codex-fluent`
- `codex-retrospective`
- `conference-video-deck-transcript`
- `create-cover-illustration`
- `ds-citations`
- `doc-intelligent-summary`
- `grill-me`
- `life-os-reflect-cover-orchestrator`
- `life-os-deep-research`
- `martin-agent-roster`
- `martin-outcome-package`
- `martin-pptx-skill`
- `publish-assets`
- `slide-renamer`
- `prompt-crafter`
- `visual-mail`

Out of scope:

- temporary experiments
- one-off project scripts
- unsanitized real examples
- secrets, tokens, internal URLs, private glossary content

## Repository Layout

- top-level reusable documentation lives in the repo root
- each skill lives under `skills/<skill-name>/`
- each skill should be portable on its own

Prefer to keep:

- `SKILL.md`
- scripts
- prompts
- generic references
- minimal tests

Prefer to remove:

- caches
- generated outputs
- `.venv`
- private examples
- local machine artifacts

## Privacy Rules

Before publishing any change, review for:

- real names, internal project names, customer context
- private glossary entries
- raw meeting transcripts or journals
- secrets or `.env` values
- private infrastructure references
- workstation-specific paths unless they are generic installation examples

If content is useful structurally but not safe to publish:

- replace it with a template
- replace it with a sanitized demo
- or remove it entirely

## Skill-Specific Notes

### brand-guidelines

- keep Martin's personal Life identity portable and self-contained
- public Work examples must use the generic `Organization Brand` adapter; never publish private organization names, Logos, templates, or customer context
- keep the HTML visual reference free of remote dependencies and validate all packaged relative asset paths
- preserve the single-primary-brand rule and keep personal identity subordinate on organization-authorized artifacts

### cloudflare-r2-publisher

- keep Bucket names, account IDs, public URLs, credentials, and machine-specific Hermes paths out of the public package
- preserve explicit publish authorization, immutable version keys, public readback, append-only lifecycle records, and exact plan-ID cleanup confirmation
- keep configuration examples placeholder-only and require credentials to remain external in a mode-`600` environment file

### audio-transcriber

- keep the script and public prompts
- glossary must remain a starter template, not a private working glossary
- reference examples must stay sanitized

### codex-fluent

- keep maintenance references generic and report-first
- do not encode private local session paths or machine-specific archive locations
- preserve handoff-first safety language before any cleanup or archive action

### codex-retrospective

- keep examples focused on reusable behavior updates, not private session details
- require evidence-backed, minimal AGENTS.md or skill changes
- avoid committing raw conversation history, transcripts, or private retrospectives

### conference-video-deck-transcript

- keep examples generic; do not commit real conference videos, transcripts, extracted frames, or generated decks
- keep paths repo-relative and avoid hardcoded vault or media archive locations
- preserve the final-deliverables-vs-artifacts separation in docs and scripts

### create-cover-illustration

- keep prompts and journal examples generic
- do not hardcode private workspace paths, personal vault names, or local upload pipeline paths
- document Gemini fallback configuration through placeholders only
- do not commit generated images or E2E artifacts unless they are sanitized fixtures

### ds-citations

- keep script, tests, and docs aligned
- prefer path examples that match this repository layout

### doc-intelligent-summary

- keep the workflow portable and repo-relative
- do not commit generated output folders or private source documents
- preserve the audit-oriented structure: source chunks, summary chunks, and run log

### grill-me

- keep the workflow session-scoped and repo-relative
- do not hardcode private Life-OS or Work-PKM absolute paths
- preserve the one-question-at-a-time interview pattern and recommended-answer structure
- keep upstream attribution generic without copying private task examples

### life-os-reflect-cover-orchestrator

- keep terminal orchestration rules generic; do not commit real journal paths, dates, screenshots, or worker transcripts
- avoid hardcoded hosted image domains, asset folders, or local upload pipeline paths
- preserve the strict ordering: finish reflection before assigning cover generation

### life-os-deep-research


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xwbcl123/Martin-brew-skills](https://github.com/xwbcl123/Martin-brew-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
