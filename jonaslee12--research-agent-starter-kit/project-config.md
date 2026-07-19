---
trigger: always_on
description: <!-- Customise this file with your own project details -->
---

<!-- Customise this file with your own project details -->

# Research Project Agent Guide

Project: `PROJECT_TITLE`

This repository is a reusable research-agent starter kit. It can be adapted for dissertations, theses, journal articles, grants, fieldwork, policy reports, design research, evidence synthesis, and knowledge-base projects.

Replace placeholders with project-specific facts only after checking a source file or receiving direct user confirmation.

## Project Profile

Before serious work, choose a profile in `RESEARCH_PROJECT_BRIEF.md` using `PROJECT_TYPE_PROFILES.md`.

Default profile: `TO CONFIRM`

Examples:

- taught dissertation / thesis
- journal article / manuscript
- research proposal / grant
- qualitative fieldwork project
- quantitative or computational study
- design research / product research
- policy / practice report
- literature review / evidence synthesis
- knowledge-base / RAG project

## Project Skills

Project-level skills live in `.agents/skills/`.

Use these skills for research-project work. Some skill names still begin with `dissertation-*` for backwards compatibility; treat them as general research workflows unless the selected project profile is specifically a dissertation or thesis.

- `agent-orchestration`: route each task to the right skills and decide whether subagents are useful.
- `dissertation-source-first-gate`: check source files before drafting formal text or factual claims.
- `dissertation-document-quality-gate`: review formal outputs before delivery.
- `dissertation-learning-loop`: turn new reading and confirmed decisions into durable project memory.
- `dissertation-literature-review`: plan and synthesise literature review work.
- `dissertation-research-search-protocol`: structure literature, web, policy, LMS, and source searches.
- `cognitive-frameworks`: make section type, claim, gap/problem type, evidence, warrant, boundary, and rhetorical plan explicit before major writing.
- `academic-self-review-loop`: run a two-pass writing-quality review and revision loop before style and document-quality gates.
- `academic-integrity-preflight`: check prompt residue, placeholders, fake references, unsupported claims, unresolved compliance requirements, and AI-use disclosure boundaries before formal drafting or delivery.
- `authorial-voice-integrity`: handle AI-writing, de-AI, humanising, detector-framed, generic-AI-style, and AI-use disclosure writing requests by improving authorial voice and integrity without detector-evasion.
- `style-fingerprint-gate`: scan repeated binary negative-contrast templates such as `rather than`, `not...but`, `不是...而是`, and `而不是` before formal delivery.
- `material-passport`: package source readiness, compliance or requirement evidence, citation boundaries, and open confirmations before formal writing or delivery.
- `formal-delivery-guard`: create/check pre-delivery locks and final guard reports before presenting formal artifacts as usable.
- `dissertation-argument-spine`: build the controlling argument and section logic.
- `dissertation-chapter-plan`: plan chapters, section jobs, and writing schedules.
- `dissertation-research-review`: review research design, questions, methods, claims, and drafts.
- `dissertation-citation-audit`: verify citations and claim support.
- `uk-academic-writing-style`: check British-English academic style when relevant.
- `style-memory-and-revision-gate`: apply user style preferences and prohibited-phrase checks.
- `context-continuity`: keep task state and handoff notes usable across long work.
- `dissertation-knowledge-ops`: maintain research-wiki, knowledge-base, Obsidian, and source registers.
- `dissertation-agent-self-debug`: diagnose false runs, stale assumptions, or shallow checking.
- `dissertation-agent-architecture-audit`: audit rule stacks, memory layers, and skill routing.
- `dissertation-workspace-surface-audit`: audit local tools, files, rendering, connectors, and missing surfaces.
- `release-surface-verification`: verify GitHub Releases, About/sidebar, topics, rendered README/docs, and public links before claiming a public release or template update is complete.
- `dissertation-automation-audit`: audit scheduled checks, hooks, monitors, and automation safety.
- `dissertation-skill-stocktake`: review skills for overlap, stale rules, and trigger clarity.
- `brainstorming`: structure unclear research-project or agent-system ideas before drafting, implementation, or skill changes.
- `project-skill-creator-governance`: govern new or updated project skills and route SKILL.md authoring to the global `skill-creator` skill.
- `playwright-dissertation-browser`: safely route browser automation to the global `playwright` skill while preserving read-only and privacy boundaries.
- `markitdown`: guide file-to-Markdown conversion for source review, literature ingestion, Obsidian notes, or RAG-ready knowledge-base preparation.
- `research-project-adapter`: map the starter kit to the selected project profile and decide which dissertation-specific files are optional.
- `research-neural-network-figure`: plan or audit neural-network architecture figures and tool routes such as NN-SVG, PlotNeuralNet, draw_convnet, TikZ, or custom SVG.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JonasLee12/research-agent-starter-kit](https://github.com/JonasLee12/research-agent-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
