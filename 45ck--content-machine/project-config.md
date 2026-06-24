---
trigger: always_on
description: **content-machine** — local-first short-form video skill pack and runtime for coding-agent CLIs.
---

# AGENTS.md

**content-machine** — local-first short-form video skill pack and runtime for coding-agent CLIs.

> **Version:** 0.2.x | **License:** MIT | **Direction:** skills + `45ck/prompt-language` flows + deterministic runtime, with the legacy CLI demoted

This file provides context for AI coding agents (Copilot, Claude Code, Cursor, etc.). For human docs, see [README.md](README.md) and [docs/](docs/).

---

## Preferred Surfaces

Use these first when working as Claude Code, Codex CLI, or similar
coding-agent CLIs:

- `skills/*/SKILL.md` — skill docs
- `flows/*.flow` — executable flow manifests
- `scripts/harness/*.ts` — optional repo-side execution surfaces
- `src/harness/*` — reusable logic behind those surfaces
- `src/*` runtime modules — direct imports only when a runtime script
  does not exist yet

The legacy `cm` surface still exists, but new agent-facing work should
prefer skills and flows over adding more control-plane logic to
`src/cli/`. Runtime scripts exist to support the skills, not define
them.

## Installed Pack Context

When this repo is installed inside another project, the materialized
pack should live under `.content-machine/`. Agents should read
`.content-machine/README.md`, `.content-machine/AGENTS.md`, and the
relevant `.content-machine/skills/*/SKILL.md` before running tools.
Packaged runtime calls should use:

```bash
npx --no-install cm-agent <tool>
```

For installed flows, pass `"flowsDir": ".content-machine/flows"` to
`run-flow` or `flow-catalog`.

## Agent Entry Points

If Content Machine is installed into another project as
`.content-machine/`, do not use the source-checkout
`scripts/harness/*` commands below. Use `npx --no-install cm-agent <tool>`
from that project and pass `.content-machine/skills` or
`.content-machine/flows` explicitly.

```
node --import tsx scripts/harness/doctor-report.ts
node --import tsx scripts/harness/flow-catalog.ts
node --import tsx scripts/harness/run-flow.ts
node --import tsx scripts/harness/skill-catalog.ts
node --import tsx scripts/harness/generate-short.ts
node --import tsx scripts/harness/asset-ledger.ts
node --import tsx scripts/harness/brief-to-script.ts
node --import tsx scripts/harness/ingest.ts
node --import tsx scripts/harness/reverse-engineer-winner.ts
node --import tsx scripts/harness/longform-to-shorts.ts
node --import tsx scripts/harness/longform-clip-extract.ts
node --import tsx scripts/harness/longform-highlight-select.ts
node --import tsx scripts/harness/highlight-approval.ts
node --import tsx scripts/harness/boundary-snap.ts
node --import tsx scripts/harness/source-media-analyze.ts
node --import tsx scripts/harness/media-index.ts
node --import tsx scripts/harness/style-profile-library.ts
node --import tsx scripts/harness/script-to-audio.ts
node --import tsx scripts/harness/timestamps-to-visuals.ts
node --import tsx scripts/harness/video-render.ts
node --import tsx scripts/harness/caption-export.ts
node --import tsx scripts/harness/publish-prep.ts
node --import tsx scripts/harness/publish-prep-review.ts
node --import tsx scripts/harness/reddit-story-assets.ts
node --import tsx scripts/harness/install-skill-pack.ts
```

Discover the live skill and flow surface instead of relying on a static
list:

```bash
cat <<'JSON' | node --import tsx scripts/harness/skill-catalog.ts
{}
JSON

cat <<'JSON' | node --import tsx scripts/harness/flow-catalog.ts
{}
JSON
```

## Current Short-Form Path

The active agent path is skill and harness driven:

```text
source-media-analyze
  -> longform-highlight-select
  -> boundary-snap
  -> highlight-approval
  -> longform-clip-extract
  -> video-render
  -> publish-prep-review
```

For topic-to-video generation, use `generate-short` or the
`generate-short` flow. For longform-to-short planning, use
`longform-to-shorts` or the `longform-to-shorts` flow, then use
`longform-clip-extract` to materialize approved source ranges before
`video-render`.

---

## Key Concepts

- **Archetype**: script format — data files in `assets/archetypes/`, overrides in `.cm/archetypes/`
- **Template**: render preset — Remotion composition + render defaults
- **Workflow**: pipeline orchestration preset

Full glossary: [`docs/reference/GLOSSARY.md`](docs/reference/GLOSSARY.md)

---

## Repository Structure

```
src/
├── cli/          # Commander.js CLI entry points and commands
├── script/       # Stage 1: LLM script generation
├── audio/        # Stage 2: TTS + ASR pipeline
├── visuals/      # Stage 3: Visual asset matching
├── render/       # Stage 4: Remotion video rendering
├── core/         # Shared infrastructure (config, LLM, logger, errors)
├── score/        # Quality scoring (audio, caption, engagement, pacing)
├── validate/     # Validation systems
├── media/        # Media synthesis (Veo, Nanobanana, DepthFlow)
├── research/     # Research orchestration
├── feedback/     # Human feedback model + JSONL store
├── lab/          # Experiment Lab (review UI)
└── test/stubs/   # Test fakes (FakeLLMProvider, etc.)
```

---

## Architecture Principles

1. **Skill Pack First** — prefer skill docs and flow docs for agent-facing work; runtime scripts back execution when needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [45ck/content-machine](https://github.com/45ck/content-machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
