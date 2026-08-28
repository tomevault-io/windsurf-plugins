---
trigger: always_on
description: Build reproducible public-microdata analyses from official, locally validated source releases. The system must acquire data without manual downloads and must fail visibly rather than fabricate or silently substitute inputs.
---

# Microdata Lab agent contract

## Purpose

Build reproducible public-microdata analyses from official, locally validated source releases. The system must acquire data without manual downloads and must fail visibly rather than fabricate or silently substitute inputs.

## Article-response projects

When the user feeds an article link and wants charts that respond to it, follow the `article-response-pipeline` skill (read → verified source matrix → position questionnaire → `responses/<slug>/position-brief.md` → data discovery → calculation → charts). The position brief is the contract: every chart serves a brief claim, and every chart note respects the brief's honest-framing constraints. Never skip the questionnaire — the user's stance is what makes the output a *response*.

## Acquisition

1. Use `microdata` source adapters. Never ask the user to browse to an agency page and download a file.
2. Official agency sources and explicitly configured licensed APIs are authoritative. Do not use mirrors.
3. Treat `$MICRODATA_ROOT/raw` and `$MICRODATA_ROOT/releases` as immutable.
4. Download into a unique incoming run, calculate SHA-256 hashes, validate required artifacts, and promote atomically only after every gate passes.
5. Preserve upstream revisions. A changed file at the same URL is a new release revision, never an overwrite.
6. Store credentials only in environment variables or an approved secret store. Never commit them.
7. Retain original documents and generate Markdown sidecars with source checksum and page markers.

## Source selection

Before selecting a survey, search the local catalog and write a source-selection note that compares:

- concept and variable definition;
- record unit and universe;
- geography and years;
- weight, design variables, replicate weights, and imputations;
- known breaks and limitations;
- local release status and provenance.

Do not treat a vaguely related measure as an equivalent concept. `not available`, `not found`, and `not comparable` are different states.

## Analysis contract

Create `analyses/<survey>-<year>-<slug>/` with:

- `question.md`: estimand, universe, variables, design, assumptions, release IDs;
- executable calculation code;
- `data.csv`: exact values supplied to renderers;
- `diagnostics.json`: row counts, weighted population, missingness, design treatment, uncertainty, and benchmark results;
- `chart.yaml`: semantic chart specification;
- `figure.png` when static output is requested;
- `interactive.html` when interactive output is requested;
- `README.md`: methods, results, limitations, and source citations.

Never infer variable meaning from its name. Read and cite the codebook or curated variable catalog entry.

## Statistical correctness

- State the record unit, analysis universe, denominator, and exclusions.
- Apply the documented weight and complex-survey design.
- Use weighted quantiles for weighted bins and document tie handling.
- Handle multiple imputations/implicates and replicate weights according to the official methodology.
- Distinguish structural zero, missing, imputed, and out-of-universe values.
- Report standard errors or confidence intervals when supported.
- Record nominal versus real dollars, inflation base, and top-coding treatment.
- Reproduce an official benchmark before accepting a new survey/year pipeline.
- Keep descriptive findings separate from causal claims.

For SCF, all five implicates and the supplied replicate-weight design are mandatory for publishable inference.

## Visualization

Static and interactive charts share `data.csv` and `chart.yaml`; they do not need identical geometry.

- Static output uses the R renderer and must remain legible at its intended publication size.
- Interactive output uses the web renderer and must provide keyboard access, reduced-motion behavior, responsive reflow, tooltips that are supplementary rather than essential, and an accessible tabular fallback.
- Prefer direct labels over legends when the series count permits.
- Titles make a factual claim; subtitles define population, measure, and period.
- Include source, release, and notes in the exported artifact.
- Do not use dual axes, truncated quantitative axes without explicit justification, decorative 3D, or color as the only encoding.
- Inspect every output at 375, 768, 1280, and 1920 pixels. Check clipping, collision, contrast, ordering, empty space, misleading scales, mobile behavior, and source-note legibility.
- Never invent data to make a graphic look complete. Test fixtures must be labeled as fixtures and must not ship as findings.

## Diagrams

For architecture, process, and data-flow visuals, load the `diagram-design` skill and follow `viz/diagrams/README.md` plus `viz/diagrams/style-guide.md`.

- Write an explicit diagram brief before drawing: type, audience, target preset, detail level, focal step, nodes retained, and detail removed.
- Use one accessible static HTML/SVG as the source of truth. Do not add a live rendering dependency or runtime animation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melon-xf/microdata-lab](https://github.com/melon-xf/microdata-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
