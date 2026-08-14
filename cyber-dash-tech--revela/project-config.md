---
trigger: always_on
description: > Current working guide for AI agents and developers in this repository.
---

# AGENTS.md - Revela Agent Guide

> Current working guide for AI agents and developers in this repository.
> Historical implementation notes belong in `docs/AGENTS.archive.md`.
> Last updated: 2026-06-12 for Codex-first maintenance policy.

## 0.18.1 Migration Override

0.18.0 is a breaking deck-first target. For current development, the default workflow is:

```text
Init -> Research -> Plan --deck -> Make --deck -> Review --deck -> Export
```

- Do not generate `revela-narrative/` during init, research, plan, make, review, or export.
- `/revela story` and `/revela make --brief` are removed from the public workflow.
- Research saves source-linked findings under `researches/`; it does not bind findings into vault evidence.
- `deck-plan.md` is the canonical execution plan. Legacy `deck-plan/index.md` and `deck-plan/slides/*.md` remain read-compatible only.
- Deck-plan slide blocks use `sourceLinks` for materials, findings, assets, URLs, and caveats. Legacy `narrativeLinks` inputs are compatibility-only.
- Design inventory must expose layout slots and component nesting hints; planned component slots must belong to the selected layout.
- Component plans support `children`; use `box.children` for semantic groups that contain text, media, charts, tables, stats, quotes, or steps.
- Review is Artifact QA plus Comment/Apply Fix. Insight/Inspect is removed from the public Review path.
- Export supports PDF, PPTX, and per-slide PNG.

Older 0.17 Narrative Vault guidance below remains release archaeology until it is fully deleted; when it conflicts with this override, follow the 0.18.1 rules above.

## Codex-First Maintenance Policy

Revela is now Codex-first. OpenCode support is legacy compatibility only.

- New features must target the Codex surface first: CLI/MCP runtime, `plugins/revela/`, Codex skills, Codex hooks, and Codex Review UI.
- Do not add new OpenCode-only slash commands, prompt transforms, subagents, or tool surfaces unless the user explicitly asks for legacy OpenCode maintenance.
- Preserve legacy OpenCode behavior when practical, but do not expand OpenCode parity for new Codex features.
- Shared deterministic logic should live in `lib/runtime/` or shared `lib/*`; adapters may wrap it.
- When a task mentions "Revela" without specifying a platform, assume Codex unless local context clearly indicates an OpenCode legacy issue.

## Product Baseline

Revela is a narrative artifact workspace for high-stakes communication. It is not a generic AI slide maker.

Product promise:

**Turn source materials, research, data, and user intent into trusted, traceable, presentation-ready decision artifacts.**

Current baseline: `0.17.24 release baseline`.

User-facing workflow:

```text
Init -> Research -> Story -> Make -> Review -> Export
System surface: Design
```

Decks are render targets. The durable core is source trust, canonical narrative state, evidence traceability, artifact coverage, and post-artifact reading/refinement.

## Non-Negotiable Product Rules

- `revela-narrative/` is the editable source of truth for communication meaning when present.
- Canonical narrative state (`NarrativeStateV1`) is the compiled internal interface for communication meaning.
- Target architecture is file-native: `revela-narrative/` for meaning, `deck-plan/` for render planning, `decks/*.html` for artifacts, `researches/` for findings, and `assets/` for media. `DECKS.json` should be removed as a product state center, not preserved as workflow authority.
- `deck-plan/`, when present, is the render-layer execution-plan workspace for slide order, chapter writing batches, visual intent, and evidence trace; it is not the source of canonical meaning.
- `DECKS.json.slides[]` must not be treated as the authoritative HTML slide-count contract. Artifact identity is self-consistent positive 1-based slide indexes, unique indexes, DOM order, and canvas validity; plan completeness belongs to `deck-plan/` projection Markdown when present.
- Vault workspaces must not persist top-level `DECKS.json.narrative`; runtime `state.narrative` may still be hydrated as a compatibility projection.
- `.revela/narrative-cache/` contains regenerable compiled projections and diagnostics, not editable source.
- Saved research findings are not evidence support until explicit evidence nodes or bindings preserve source, quote/snippet, support scope, unsupported scope, caveat, and strength.
- Do not invent quotes, source paths, URLs, page references, caveats, claim ids, evidence ids, or artifact coverage.
- Missing evidence must stay visible as a gap instead of being filled by the model.
- Workflow permission gates should be removed. Users decide whether to continue; Revela reports diagnostics, risks, missing information, and technical validity.
- Hard blockers are limited to technical artifact validity, data-safety/integrity protections, and executable preconditions such as missing files or ambiguous paths.
- Meaning changes update canonical narrative first; artifact or deck-plan alignment gaps should be reported as diagnostics, not approvals or workflow blockers.
- Pure artifact polish may stay artifact-level: layout, typography, spacing, crop, visual hierarchy, export mechanics, deck contract fixes, and similar non-meaning changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyber-dash-tech/revela](https://github.com/cyber-dash-tech/revela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
