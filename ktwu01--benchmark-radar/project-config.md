---
trigger: always_on
description: Read `principle.md` before changing a benchmark-facing surface. Its full-corpus
---

# Repository Instructions

Read `principle.md` before changing a benchmark-facing surface. Its full-corpus
coverage rule applies to charts, search, tables, counts, and exports: start from
1,259+ benchmark records across 4+ sources, and investigate any unexplained
reduction to a few dozen. Missing measurements must not remove corpus records.
Benchmark Frontier and its linked score ranking explicitly exclude records
without numeric reported scores, as specified in `principle.md`.

## Reference guides

This file holds the rules that apply to every change. Detailed procedure lives
in the guides below. Each line states when the guide is required reading, and
the constraint that holds whether or not you open it.

| Guide | Read it before | Constraint that always holds |
| --- | --- | --- |
| [`docs/pipeline-and-data-map.md`](docs/pipeline-and-data-map.md) | Touching a source input, a generator, a generated artifact, or the technical report | Never reconstruct the system from a report, the deployed site, or leftover generated files. Never patch a derived file to fix a source-data problem. |
| [`docs/sop-add-model-cards.md`](docs/sop-add-model-cards.md) | Adding a model card, a benchmark, or a score | `data/model_cards.yml` and `data/benchmark_scores.yml` move together. Every value is read out of the cited document, never from memory. |
| [`docs/query-surfaces.md`](docs/query-surfaces.md) | Changing search, detail lookup, the CLI or HTTP query surface, or the consumer Skill | `QueryService` is the single source of truth. No interface-specific ranking, and no silent network fallback. |
| [`principle.md`](principle.md) | Changing any benchmark-facing surface | Start from the full corpus across all sources. |

## Glob rule: showcase and UI communication

Applies to `README*`, `docs/**`, `.github/ISSUE_TEMPLATE/**`, `site/**`, and
any report, launch note, TLDR, screenshot, GIF, demo, dashboard, or UI surface.

- Treat what is shown as part of the work. What was done and what is displayed
  are both important; in many communication surfaces, what is displayed is more
  important because it is the receiver's entry point.
- Start from the receiver's perspective, not the implementer's. Ask what the
  reader most wants to know, what will help them decide quickly, and what is
  most worth remembering or sharing.
- Do not let engineering effort bury the message. Data work and implementation
  details often take most of the time, but reports and TLDRs should foreground
  the result, implication, and decision-useful signal before the process.
- Prefer strong information hierarchy, plain language, concrete examples,
  screenshots, short GIFs, and compact summaries that make the work easy to
  scan, review, forward, or explain upward.

### Example: simplify badge copy and keep its style

Before:

```html
<p align="center">
  <a href="https://koutian.is-a.dev/benchmark-radar/"><img alt="Benchmark records collected" src="https://img.shields.io/endpoint?url=https%3A%2F%2Fkoutian.is-a.dev%2Fbenchmark-radar%2Fdata%2Frecords-badge.json&amp;style=for-the-badge"></a>
  <a href="https://koutian.is-a.dev/benchmark-radar/data/radar.json"><img alt="Download dataset" src="https://img.shields.io/badge/Dataset-download%20JSON-2f81f7?style=for-the-badge&amp;logo=json&amp;logoColor=white"></a>
  <a href="https://x.com/ktwu01"><img alt="X" src="https://img.shields.io/badge/X-%40ktwu01-000000?style=for-the-badge&amp;logo=x&amp;logoColor=white"></a>
  <a href="https://www.linkedin.com/in/ktwu01"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-Koutian%20Wu-0A66C2?style=for-the-badge&amp;logo=linkedin&amp;logoColor=white"></a>
  <a href="https://scholar.google.com/citations?user=s9w1k-cAAAAJ&amp;hl=en"><img alt="Google Scholar" src="https://img.shields.io/badge/Google%20Scholar-Koutian%20Wu-4285F4?style=for-the-badge&amp;logo=googlescholar&amp;logoColor=white"></a>
</p>
```

After:

```html
<p align="center">
  <a href="https://koutian.is-a.dev/benchmark-radar/"><img alt="Benchmark records collected" src="https://img.shields.io/endpoint?url=https%3A%2F%2Fkoutian.is-a.dev%2Fbenchmark-radar%2Fdata%2Frecords-badge.json&amp;style=for-the-badge"></a>
  <a href="https://koutian.is-a.dev/benchmark-radar/data/radar.json"><img alt="Download dataset" src="https://img.shields.io/badge/Dataset-download%20JSON-2f81f7?style=for-the-badge&amp;logo=json&amp;logoColor=white"></a>
  <a href="https://x.com/ktwu01"><img alt="X" src="https://img.shields.io/badge/X-000000?style=for-the-badge&amp;logo=x&amp;logoColor=white"></a>
  <a href="https://www.linkedin.com/in/ktwu01"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&amp;logo=linkedin&amp;logoColor=white"></a>
  <a href="https://scholar.google.com/citations?user=s9w1k-cAAAAJ&amp;hl=en"><img alt="Google Scholar" src="https://img.shields.io/badge/Google%20Scholar-4285F4?style=for-the-badge&amp;logo=googlescholar&amp;logoColor=white"></a>
</p>
```

The after example removes the handle or name from three badge labels. It keeps
the five-badge layout, badge styles, logos, colors, and profile URLs.

## Glob rule: Benchmark Radar audience

Applies to `README*`, `docs/**`, `.github/ISSUE_TEMPLATE/**`, `site/**`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ktwu01/benchmark-radar](https://github.com/ktwu01/benchmark-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
