---
trigger: always_on
description: <!-- Generated for the public repository by the "public-document-set" recipe. -->
---

<!-- Generated for the public repository by the "public-document-set" recipe. -->
# Agent Instructions

This repo treats product strategy, architecture, and roadmap docs as live system state. Keep them current with the code.

## Product Canon

- The product is **Exawatt**.
- Exawatt is an Electron desktop app and future hosted interface layer for commanding agents from any compatible source: local OpenClaw, hosted OpenClaw, Codex, Claude Code, custom harnesses, and Demo Mode.
- Demo Mode is first-class forever. It should exercise the same UI and command layers through a lower data-source layer.
- Helios/Godot is a retired direction. Do not reintroduce it as active architecture unless the roadmap and decision records explicitly change.
- Architecture rule: architect 10 miles ahead, build one mile at a time.

## Canonical Concepts

Use the vocabulary in `docs/product/concepts.md` and keep `/architecture` aligned with it:

- Workspace
- Initiative
- Project / Context Group
- Agent
- Session
- Event
- Artifact
- Consumption
- Decision
- Attention Scheduling
- Context Signal
- Secret / Credential
- Agent Source / Harness
- Launch Configuration
- Gateway
- Policy / Budget
- Approval

## Planning Canon

Plans are live product state only when they amend canonical docs.

- Planning is first-class work (operator, 2026-08-02): invest time and inference in researching, shaping, and grooming roadmap items independently of executing them, so many parallel agents can pick up well-fed, executable items. Grooming sessions promote conclusions into canon as they happen, not only at the end.
- Keep executable plans centralized in `docs/engineering/roadmap.md`, linked engineering project docs, architecture docs, or decision records.
- Do not turn each chat session, brainstorm, or implementation attempt into a competing standalone plan.
- If a draft plan contains useful thinking, promote durable conclusions into the canonical docs and archive or mark the draft stale.
- Make canonical plans agent-readable: explicit scope, sequence, assumptions, ownership boundaries, and acceptance criteria.
- Keep `docs/engineering/roadmap.md` as the singular engineering roadmap. It may point to deeper project docs, but those docs must remain execution detail for a roadmap item, not independent roadmaps.
- Keep milestone NARRATIVES out of the roadmap. The roadmap carries a status index, each item's contract (status, concise scope, exit criteria, links), and a one-line-per-milestone list; the full narrative, root causes, and evidence belong in the item's project doc under its dated `Roadmap milestone log` / progress log section.
- Later milestones frequently supersede earlier ones. Record every supersession in the roadmap's `Amendment chain` table so an agent reading only the roadmap cannot act on a superseded decision.
- Preserve stable roadmap IDs such as `ENG-004` when reordering or editing roadmap items.
- Use only roadmap statuses defined in `docs/engineering/roadmap.md`.
- Prefer adapting the single roadmap and its linked project docs over creating parallel roadmap layers.
- Treat canonical plans as living hypotheses, not commands to obey blindly. Agents should flag contradictions, weak assumptions, or stale milestones and update the canonical plan statefully when direction changes.

## Documentation Contract

When changing product concepts, update:

- `docs/product/concepts.md`
- any affected customer-facing guide source in `docs/product/guides/`
- any affected public-safe reference source in `docs/product/reference/`

When changing architecture, update:

- `docs/engineering/architecture.md`
- `src/lib/architecture/manifest.ts`, which powers `/architecture`

When changing roadmap scope, sequence, or status, update:

- `docs/engineering/roadmap.md`

When making a durable tradeoff, choosing a vendor, reversing a direction, or retiring an approach, add or update:

- `docs/engineering/decisions/`

When a failure in a dogfood or released build costs real diagnosis time and the cause is not obvious from the code — an external cause (macOS, Electron, a dependency, another app), a symptom that pointed at the wrong subsystem, or a remedy that is only a workaround — add:

- `docs/engineering/incidents/`

Check that directory BEFORE re-diagnosing a familiar symptom. Records carry the falsified hypotheses and the diagnostic method, so the expensive part is not repeated. Its `README.md` holds the contract and template.

Do not create competing plans. Replace, archive, or explicitly mark stale docs. If old thinking is worth preserving, move it to `docs/archive/` with a note explaining why it is no longer active.

## Engineering Guidance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Full-Vibe/exawatt](https://github.com/Full-Vibe/exawatt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
