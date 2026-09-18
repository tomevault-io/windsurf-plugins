---
trigger: always_on
description: Documentation map for RubyLens. Start with [README.md](README.md) for usage and development setup.
---

# AGENTS.md

Documentation map for RubyLens. Start with [README.md](README.md) for usage and development setup.

## Working conventions

Maintainer preferences for agents working in this repository.

Code:

- Do not introduce attributes that duplicate or are derivable from data that is already present — a stored copy can drift out of sync with its source. Derive from the single authoritative representation instead (for example, a package's declaration count is its `declarations.length`, never a separate field).
- No one-line delegator methods — inline the underlying call at each call site, even when it is wordier.
- Cheap identity markers (like the payload `schema` fields) are worth keeping, but only with their purpose documented in a correctly named home (see [docs/SCHEMAS.md](docs/SCHEMAS.md)).

Workflow:

- Never post GitHub comments, review replies, or reactions on the maintainer's behalf. Respond to review feedback with code changes and report conclusions in the conversation.
- Communicate in plain English, and be concise.
- Verification is empirical, not just green tests: prove behavior-preserving changes by generating artifacts from both versions and comparing the embedded data and rendered pixels, and prove features by running them end to end.
- Diagnose before assuming a regression — reproduce first; missing output is often environmental (for example, dependency gems not installed in the generating bundle).
- Adversarially review any complexity you add: every new guard, field, helper, or layer must survive the question of whether it needs to exist at all before it ships.

## Contracts

- [PRODUCT.md](PRODUCT.md) — the product contract: surfaces, meaning and scale, privacy, and non-goals.
- [DESIGN.md](DESIGN.md) — the design contract: stellar identity, Explorer interaction, motion, and Showcase rules.

## Engineering references

- [docs/SCHEMAS.md](docs/SCHEMAS.md) — the payload contract: snapshot, art, and showcase schema shapes, and when to bump their versions.
- [docs/PERFORMANCE.md](docs/PERFORMANCE.md) — scale instrumentation, complete-row artifact evidence, and the dependency-aggregation benchmark.
- [docs/EXPLORER_SHOWCASE_RENDERING.md](docs/EXPLORER_SHOWCASE_RENDERING.md) — shared renderer data and intentional Explorer/Showcase presentation differences.

## Visual design

- [docs/STELLAR_DESIGN_RESEARCH.md](docs/STELLAR_DESIGN_RESEARCH.md) — the astrophysical visual grammar: morphology, light, and performance rules the renderer follows.
- [docs/specs/2026-07-14-galaxy-morphology-design.md](docs/specs/2026-07-14-galaxy-morphology-design.md) — the accepted deterministic galaxy morphology design.

## Renderer geometry practices

Rules distilled from iterating on the deterministic galaxy recipes in
`assets/runtime/report.js`. They exist because each one was violated once and
produced a visual regression that unit tests missed.

- Judge geometry changes by renders, not code review: top-down scatter
  small-multiples for structure plus real Explorer renders, before and after,
  including dense realistic star counts — several defects only appear at
  scale or on real projects.
- Ground visual tuning in observed galaxy structure (see
  [docs/STELLAR_DESIGN_RESEARCH.md](docs/STELLAR_DESIGN_RESEARCH.md)) rather
  than iterating by taste, and prefer physics laws with closed-form
  inverse-CDF samplers; observation wins over theory when they disagree.
- Never let draws pile onto a bound: clamping radii or flattening a sweep to
  a constant concentrates stars into arcs, rings, or spokes. Respread the
  mass instead, and rely on `test/js/position_distribution.test.mjs` — when
  adding a recipe, extend it and calibrate thresholds against a known-bad
  build so the guard demonstrably fails on the defect.
- A predicate the renderer uses must be the same function any test measures;
  never let a test assert a proxy gate while the renderer adds conditions.
- Geometry shared between the project galaxy and dependency clouds must live
  in one helper; hand-copied variants have diverged before. Verify intended
  no-op refactors seed-for-seed against the previous runtime.
- `unit(seed, channel)` draws must keep channels disjoint per population, and
  `normal(seed, channel)` consumes both `channel` and `channel + 1`;
  collisions silently bias distributions. Name tuned constants and their
  channels in a frozen recipe block (see `ARM_RECIPE`) instead of inlining
  them, keeping only formula-local ratios inline.
- Ruby tests never assert on frontend asset content: everything the browser
  consumes — the runtime, shells, and stylesheets — is tested in the vitest
  suite (`test/js/`). Ruby asserts assembly and write mechanics, packaging,
  and Ruby-side constants only. Classifier knob changes still need
  `REGENERATE_FIXTURES=1 bundle exec rake` and the pinned knob rows updated.

## Indexing pipeline

`lib/rubylens/index/` splits one Rubydex run into collaborators with separate
concerns. Keep new indexing work inside whichever one already owns the concern.

- `Manifest` decides *what* is indexed: the Git-selected workspace plus the
  packages resolved from `Gemfile.lock`.
- `GitPackageSource` owns git checkouts specifically, because their gemspecs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [st0012/rubylens](https://github.com/st0012/rubylens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
