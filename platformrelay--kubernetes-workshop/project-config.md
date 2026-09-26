---
trigger: always_on
description: Guidance for any agent or contributor working in this repository.
---

# AGENT.md

Guidance for any agent or contributor working in this repository.

This is an **open source, vendor-neutral, beginner-to-intermediate Kubernetes workshop**.
It ships as a Slidev slide deck plus a separate set of hands-on labs in Markdown. The
workshop is **50% presentation, 50% practice**.

Content is authored as a **superset** and boiled down per delivery: every section is a
self-contained, **toggleable** unit, and generated **root decks** compose Day 1, Day 2,
Day 3, and Optional / Appendix entries. `slides.md` and `slides-3day.md` remain
compatibility entries, not the default live-delivery path. See
[Deck architecture](#deck-architecture--compartmentalized-sections).

## Non-negotiable guardrails

These apply to **everything** — slides, labs, components, assets, planning docs,
filenames, and **commit messages**.

1. **No brand references.** Do not name or imply any specific employer, customer, or
   corporate brand anywhere. This workshop is vendor-neutral.
2. **No tooling or AI attribution.** Do not mention the editors, generators, or AI
   assistants used to produce any material — not in content and **not in commit
   messages**. Do **not** add `Co-Authored-By` or similar trailers.
3. **Label AI-generated imagery.** Any AI-generated image (e.g. the Mœbius-style
   section covers) must carry a visible `AI generated` footer on the slide.
4. **Commit messages: Conventional Commits + gitmoji.** See
   [Commit conventions](#commit-conventions).
5. **Stay current.** Track current Kubernetes behaviour, API versions, and CNCF
   ecosystem conventions. Legacy source material is inspiration only — update anything
   outdated.
6. **Alignment, not exam prep.** Coverage is aligned with CKAD/CKA domains as a design
   check, but certification prep is not the organizing principle. **Verify the current
   Kubernetes / CKx curriculum version at authoring time — do not hard-pin a version in the
   docs.**

## Where things live

| Path | Purpose | Tracked? |
| --- | --- | --- |
| `AGENT.md` | This file — contributor guidance. | yes |
| `scripts/deck-manifest.mjs` | Source of truth for section metadata and generated deck membership. | yes |
| `slides-day-*.md`, `slides-optional.md` | Generated live-delivery entries; never edit by hand. | yes |
| `slides.md`, `slides-3day.md` | Generated compatibility entries for the superset and legacy combined cut. | yes |
| `pages/SNN-topic/` | One self-contained, toggleable **section** per folder (`index.md`). | yes |
| `theme/` | **Local Slidev theme** (`slidev-theme-k8s-workshop`): master styles, layouts, UI components. Root decks use `theme: ./theme`. | yes |
| `components/` | Deck-level Vue components (animated teaching diagrams). | yes |
| `labs/day-*/` | Standalone Markdown labs (not embedded in the deck). | yes |
| `agent-context/` | Planning, roadmap, user stories, outline, image prompts, source analysis. **Local working material.** | no (gitignored) |
| `references/` | Vendored reference theme/pattern gallery and CNCF artwork, for rehearsal. | no (gitignored) |
| `.claude/` | Local tooling/skills. | no (gitignored) |

> **Deck location note:** the deck lives at the repo root. `slides-day-1.md`,
> `slides-day-2.md`, `slides-day-3.md`, and `slides-optional.md` compose the
> `pages/SNN-topic/` section library for live delivery;
> `slides-templates.md` is the design-system gallery (reusable layouts + the
> animation-technology spike) — reference material, not workshop content. All sections
> `S00`–`S27` exist as stubs; author content into them milestone by milestone (Day 1 first).

## Source of truth for scope

The plan lives in `agent-context/` (gitignored, local). Read it before authoring:

- `agent-context/roadmap.md` — milestones, delivery model, guardrails.
- `agent-context/user-stories.md` — the backlog. **US-0 comes first**: build reusable
  slide templates (using the Kubernetes/CNCF icons) and a pod-replacement animation
  spike before curriculum content.
- `agent-context/presentation-outline.md` — the full section-by-section outline as a
  **compartmentalized superset** (`S00`–`S27`), each section tagged with a **Tier**
  (`core`/`recommended`/`optional`) and **Suggested day**. The spine is the **red line**:
  `Pod → Deployment → Service → Ingress → Gateway API` (S05–S09). Also holds the **deck
  architecture**, the **lab authoring contract**, the CKAD/CKA alignment appendix, and the
  **canonical 3-day cut** (Appendix C).
- `agent-context/section-image-prompts.md` — Mœbius continuous-story covers (`S00`–`S27`).

## Teaching model

- Each day is ~50% slides / ~50% hands-on. Every concept block names the lab that
  follows it.
- Module rhythm: **problem → mental model → minimal YAML → run it → observe → break it
  → fix it → recap**.
- **Environments:** every lab must run in an assigned **namespace** on a shared
  cluster *or* a local **kind** cluster. Never require cluster-admin unless the topic
  needs it; then mark the lab **kind-only** and provide a namespace-safe read-only
  alternative.

## Deck architecture — compartmentalized sections

The deck is a **superset** of toggleable sections composed by one or more **root decks**.

- **One folder per section:** `pages/SNN-topic/index.md` (e.g. `pages/S01-containers/index.md`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlatformRelay/Kubernetes-Workshop](https://github.com/PlatformRelay/Kubernetes-Workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
