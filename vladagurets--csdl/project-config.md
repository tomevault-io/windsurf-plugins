---
trigger: always_on
description: These instructions apply to the entire repository. The canonical GitHub target is `vladagurets/csdl`.
---

# AGENTS.md — CSDL operating instructions

These instructions apply to the entire repository. The canonical GitHub target is `vladagurets/csdl`.

## Mission

Develop Constructive Signal Design Language as a versioned, machine-readable visual language for educational presentation slides about AI, software engineering, and economics. Optimize for clarity, landscape presentation readability, memorability, and reproducibility with GPT Image 2.

## Mandatory reading order

Before changing anything, read:

1. `STATUS.md`
2. `DECISIONS.md`
3. `specs/2026-07-17-csdl-v0.1-design.md`
4. `pilots/01-agentic-discipline/manifest.yaml`
5. the relevant task in `docs/superpowers/plans/2026-07-17-csdl-pilot-01.md`
6. `pilots/01-agentic-discipline/prompts/00-style-anchor.yaml`
7. `pilots/01-agentic-discipline/references/style-anchor-light.provenance.md`
8. `docs/handoff/CODEX_IMAGE_GENERATION.md` for raster tasks

Do not rely on memory or infer a new direction when these files are explicit.

## Locked design constraints

Do not change these without explicit user approval and a corresponding update to `DECISIONS.md`:

- direction: Constructive Signal;
- default expression: Quiet Modular;
- display direction: Modular Technical, with rare condensed editorial emphasis only;
- palette character: warm, muted, mineral, restrained;
- canonical canvas: 1920×1080, ratio 16:9, landscape;
- portrait masters and mobile-preview deliverables are out of scope;
- standard series rhythm: A, A, B, A, B, A, C;
- one main idea, one visual mechanism, one dominant signal per screen;
- every three-candidate review uses three materially different conceptual and compositional directions for the same approved content contract;
- 50–75% negative space depending on expression level;
- no political, Soviet, revolutionary-poster, or imitation-1920s styling;
- Markdown is the canonical specification.

## Current objective

Milestone 7 — Cookbook and Design Book v1.0 is complete under `cookbook/design-book-v1.0/` and D-033 through green integration PR #69 and merge commit `4c20829f4923c164b48985d06a49247ff372ed4f`. Preserve its additive publication contract, Prompt DSL v0.5, exactly fifteen public components, exactly 23 recipes, Analytical Mode v0.1 invariants, Night Mode and Accessibility v0.1, and all sixty accepted raster hashes. Do not generate, recolor, or replace CSDL raster evidence without separate explicit approval.

Milestone 8 has started only for the D-034 licensing slice. CSDL is source-available for noncommercial use: PolyForm Noncommercial 1.0.0 covers software and machine-readable materials, CC BY-NC-SA 4.0 covers original documentation and visuals, commercial use requires a separate written license, and CSDL branding is reserved. Preserve the exact scope in `LICENSE`, the single-owner contribution gate, and the statement that this is not OSI open source. Public-release versioning, tags, GitHub Releases, final font licensing, and long-term asset distribution remain out of scope without a new explicit objective.

## Work protocol

1. Use one branch and one pull request per independently reviewable task.
2. Preferred branch names: `codex/pilot-01-card-01`, `codex/pilot-01-card-02`, and so on.
3. Preserve canonical copy from `manifest.yaml` exactly. A copy change requires user approval before editing the manifest.
4. Before generation, define separate `v1`, `v2`, and `v3` direction briefs. All three preserve the approved topic, exact copy/data, evidence, expression level, canvas, reference hierarchy, and hard exclusions, but each pair must differ in conceptual framing, dominant visual mechanism, and composition topology.
5. Generate one candidate from each direction-specific prompt. Repeating one prompt three times is not a valid three-candidate pass. Mirroring, repositioning, color or shape swaps, minor scale/spacing changes, and decorative substitutions do not count as different directions.
6. Review the three candidates together before selection. If any pair reads as near-variants of the same representation, reject and regenerate the collapsed direction before applying the evaluation rubric. Do not silently promote the first candidate.
7. Keep drafts under `pilots/01-agentic-discipline/drafts/`; they are intentionally ignored by Git.
8. Commit approved card files only under `pilots/01-agentic-discipline/canonical/`.
   Canonical PNG filenames use the unpadded one-based card position only:
   `1.png`, `2.png`, and so on. Keep slugs in prompts, drafts, and previews.
9. Update `STATUS.md` and `CHANGELOG.md` whenever a task is completed or a decision changes.
10. Record the three direction briefs, all candidate filenames, observed concept/mechanism/composition differences, divergence result, rejection reasons, the selected filename, exact-copy review, dimensions/color-mode review, and selection rationale in `pilots/01-agentic-discipline/evaluation/review.md`. Draft-local `approved.txt` is only a convenience marker and is not persistent project evidence.
11. Record the accepted card score in `pilots/01-agentic-discipline/evaluation/scores.csv`.
12. Do not add decorative geometry that has no semantic role.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vladagurets/csdl](https://github.com/vladagurets/csdl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
