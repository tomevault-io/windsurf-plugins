---
trigger: always_on
description: This file is the shared operating manual for AI contributors working in the
---

# AI Skill: HiperHealth Contributor Guide

This file is the shared operating manual for AI contributors working in the
`hiperhealth` repository. Use it to keep implementation style, review standards,
privacy expectations, and delivery quality consistent across agents.

## Recommended AI Configuration

For non-trivial work in this repository, especially changes touching clinical
logic, privacy, LLM prompts, schemas, or release automation, use Codex with the
latest model available and set the reasoning level to `xhigh`.

## When To Use This Skill

Use this guidance for any change inside the HiperHealth library/SDK repository:

- pipeline, stage runner, session, or skill registry changes
- built-in skill updates: diagnostics, extraction, privacy, or future skills
- LLM provider, prompt, structured-output, or configuration changes
- Pydantic schema, FHIR, SQLAlchemy model, or generated model changes
- CLI behavior updates
- documentation, examples, or MkDocs/Quarto updates
- tests, typing, linting, release, and CI-related maintenance

## Core Objectives

1. Protect patient privacy and avoid committing real PHI, secrets, or API keys.
2. Preserve public API and serialized data compatibility unless the task
   explicitly changes them.
3. Keep clinical outputs structured, validated, and clearly scoped as software
   assistance rather than medical advice.
4. Keep source, schemas, generated models, docs, and tests aligned.
5. Keep quality gates green: tests, coverage, mypy, ruff, douki, bandit,
   vulture, and pre-commit.
6. Make minimal, targeted edits with clear intent.

## Project Snapshot

- Package: `hiperhealth`
- Purpose: core Python library/SDK for HiperHealth clinical AI workflows; this
  repository is not the web application.
- Runtime: Python `>=3.10,<4` with CI currently covering Python 3.10 to 3.13 on
  Ubuntu and macOS.
- Packaging: `setuptools` with `src` layout and `uv`/`pip` installation.
- CLI entry point: `hiperhealth = hiperhealth.cli:main`
- Main architecture:
  `clinical data -> PipelineContext/Session -> StageRunner -> Skills -> typed outputs`
- Core concepts:
  - independently executable stages: screening, intake, diagnosis, exam,
    treatment, prescription
  - composable skills with `pre()`, `execute()`, and `post()` hooks
  - parquet-backed sessions for resumable workflows
  - Pydantic schemas for clinical outputs and FHIR-facing data models
  - LiteLLM-backed structured generation for provider-agnostic LLM access
  - Presidio-backed privacy/de-identification utilities
  - PDF/image/wearable extraction helpers
- Docs stack: MkDocs Material, mkdocstrings, and Quarto for rendered examples.
- Release: semantic-release with conventional commit / PR title expectations.

## Repository Layout

- `src/hiperhealth/`: package implementation
- `src/hiperhealth/agents/`: lower-level agent-style workflows and extraction
  helpers
- `src/hiperhealth/pipeline/`: stage, context, session, runner, registry, and
  skill abstractions
- `src/hiperhealth/skills/`: built-in installable skills and skill metadata
- `src/hiperhealth/privacy/`: de-identification implementation
- `src/hiperhealth/schema/`: Pydantic schema definitions
- `src/hiperhealth/models/sqla/`: SQLAlchemy models, including generated FHIR
  mappings
- `tests/`: pytest coverage and test data
- `docs/`: user-facing documentation and rendered examples
- `scripts/`: install, build, publishing, docs navigation, and model generation
  helpers
- `conda/dev.yaml`: development environment definition
- `.makim.yaml`: task runner definitions
- `.github/workflows/`: CI, docs, and release workflows
- `pyproject.toml`: package metadata, dependencies, and tool configuration

## Architecture And Responsibilities

### Pipeline and Skills

- `PipelineContext` carries patient data, language, session identifiers,
  accumulated results, and intermediate metadata.
- `Stage` defines the supported clinical workflow stages.
- `StageRunner` runs all skills registered for a stage in registration order.
- Skills should declare immutable `SkillMetadata` and implement only the hooks
  they need.
- `check_requirements()` should return structured `Inquiry` objects rather than
  raising for missing optional or collectable data.
- Keep stage execution deterministic except for explicitly injected external
  dependencies such as LLM clients.
- Do not add hidden global state to skills or runners; prefer explicit context,
  settings, and dependency injection.

### Sessions

- Sessions persist interaction history to parquet and must remain usable from
  notebooks and data tools.
- Preserve event log semantics and backward compatibility where possible.
- Treat session files as potentially sensitive clinical artifacts. Do not add
  logging or debug output that dumps patient records by default.

### Skill Registry and Channels

- The CLI manages skill channels and installs external skills from local paths
  or Git sources.
- Canonical external skill ids use `<local_channel_name>.<skill_name>`.
- Keep registry operations explicit, testable, and safe for repeated runs.
- Avoid network access in unit tests; mock Git or filesystem operations when
  needed.

### LLM Integration

- Use `hiperhealth.llm.LLMSettings` and the `StructuredLLM` protocol for LLM
  workflows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiperhealth/hiperhealth](https://github.com/hiperhealth/hiperhealth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
