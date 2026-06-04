---
trigger: always_on
description: Operating manual for AI coding agents working in `spine-benchmark`. Read this before opening files. The rules below are not stylistic preferences - they are load-bearing constraints that protect properties the project relies on.
---

# AGENTS.md

Operating manual for AI coding agents working in `spine-benchmark`. Read this before opening files. The rules below are not stylistic preferences - they are load-bearing constraints that protect properties the project relies on.

## What this repo is

A Spine animation profiling toolkit. Two views of the same data:

1. **Offline benchmark** (`apps/benchmark`) - parses a `.skel`/`.json` + atlas, computes per-animation impact scores from the static data, renders the result as the public site. The numbers it shows are the canonical "this is how expensive this skeleton is" verdict.
2. **Live crawler** (`packages/pixi-crawler`, demo in `apps/crawler`) - attaches to a running PixiJS `Application`, walks the live scene graph each frame, and computes the same scores against whatever the GPU is actually drawing right now.

The crawler also analyzes non-Spine objects (sprites, masks, filters, blend breaks, draw call counts) - it is the eye in the scene graph, the spider in the web. Its job is to look at any PixiJS scene and answer "what is going wrong here, and what would the offline benchmark say if it could see it?".

## Core values (do not violate)

### 1. Single source of truth for scoring math

Every impact score in this repo - per-frame heatmap, offline animation score, runtime crawler budget, summary tab, mesh tab, physics tab - flows through one file:

```
packages/metrics-impact-formula/src/index.ts
```

That package exports `renderingImpactCost`, `computationalImpactCost`, `impactFromCost`, `classifyImpactLevel`, `DEFAULT_IMPACT_BRACKETS`, and the `ImpactLevel` union. It has zero dependencies so even the crawler (which is published to npm and must stay tiny) can pull it in.

**Never inline the weights.** If you find yourself writing `* 0.7` next to `* 0.55` next to `* 0.35`, or `Math.min(0.5, x / 500)`, or `/ 200` for a vertex divisor, you are reimplementing the formula. Stop and import it instead.

There is a CI guard at `scripts/check-no-duplicate-impact-formulas.mjs` that fails the build if the canonical numbers appear outside the leaf package. The guard runs as part of `npm test`. Do NOT add files to its allowlist to make it shut up - fix the duplication.

### 2. Heatmap / crawler / offline benchmark must be 1:1

If the same skeleton is observed by the offline benchmark, the in-app heatmap, and the live crawler at the same instant, all three must produce the same RI/CI numbers. Identical formulas alone are not enough - the *inputs* to the formulas must also match:

- **Visibility** is `slot.color.a > 0 && (slot.bone?.active !== false)`. The crawler used to read `slot.data.visible`, which does not exist on the real `spine-core` `SlotData` shape; that read returned `undefined` and silently zeroed every score in production. If you need to test slot visibility, use the `isSlotActive` predicate in `spine-analyzer.ts` (or the equivalent inline check in `useAnimationHeatmap.ts`).
- **Constraints** must be filtered by `constraint.active`. Spine flips this on/off via skin overrides and constraint controllers. The crawler used to count `skeleton.ikConstraints.length` unconditionally and overshoot CI on skeletons with deactivated constraints.
- **Blend mode RI input** is the count of currently visible slots whose blend mode is non-normal. It is NOT the count of blend mode transitions. Transitions are used separately for draw-call estimation; do not feed them into `renderingImpactCost`.

The crawler test suite includes an explicit parity check that asserts these inputs match. If you change the analyzer, run that test and add a new one if you discover another parity gap.

### 3. The crawler is a published package

`@spine-benchmark/pixi-crawler` ships to npm. That means:

- Public types are part of the API. Renaming, removing, or changing the union of `ImpactLevel`, `CrawlerConfig`, `SpineAnalysis`, etc. is a breaking change. Bump the major (or 0.x minor) accordingly.
- Dependencies on workspace packages must be on **published** versions, not `file:..` paths, otherwise `npm publish` will rewrite the path and break the consumer install. The leaf package `@spine-benchmark/metrics-impact-formula` is also published for this reason.
- Any new runtime dependency added to the crawler is paid for by every consumer. Prefer duck typing over importing optional runtimes (the analyzer never imports `@esotericsoftware/spine-core` directly; it uses a `SpineLike` interface).

### 4. House style: ASCII only for arrows and dashes

This repo forbids "fancy" Unicode punctuation in source, docs, locales, and CI files:

| Forbidden | Use instead |
|-----------|-------------|
| em-dash `U+2014` | `-` (or `--` between code sections) |
| en-dash `U+2013` | `-` |
| right arrow `U+2192` | `->` |
| left arrow `U+2190` | `<-` |
| double arrow `U+21D2` | `=>` |
| any other Unicode arrow | the closest ASCII equivalent |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schmooky/spine-benchmark](https://github.com/schmooky/spine-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
