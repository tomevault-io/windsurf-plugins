---
trigger: always_on
description: Project-specific instructions for Claude (and humans) working in this repository.
---

# CLAUDE.md

Project-specific instructions for Claude (and humans) working in this repository.
This file is the load-bearing source of truth for "how do we write code here." If
something in here conflicts with what you would do by default, this file wins.

## Project overview

**xlsx-kit** is a TypeScript library for reading and writing Excel `.xlsx`
workbooks from Node 22+ and modern browsers, with no runtime dependencies on
Python or Excel. Inspired by [openpyxl](https://openpyxl.readthedocs.io/).

- **Status**: pre-1.0 alpha. Core read / write / streaming pipeline is in place
  and round-trips real-world fixtures (including pivot tables and `.xlsm`),
  but APIs may shift before `1.0`.
- **License**: MIT, single tier, no Pro / paid features.
- **Audience**: OSS — code that external contributors and end users will read.

See `README.md` for the user-facing pitch and `CONTRIBUTING.md` for the
day-to-day contributor workflow.

## Tech stack

- TypeScript with strict mode + `exactOptionalPropertyTypes` +
  `noUncheckedIndexedAccess`. No `as any`, no `@ts-ignore`.
- Runtime: Node `>=22` (relies on built-in `Web Streams`, `Blob`, `fetch`).
  Modern browsers via the same APIs.
- Runtime dependencies: `fflate` (deflate / inflate), `saxes` (SAX XML),
  `fast-xml-parser`. Anything else is a build-time dependency.
- Tooling: pnpm, vitest, oxlint, knip, tsdown (bundler), size-limit, typedoc,
  changesets (release management), libxml2-utils (`xmllint`) for ECMA-376 XSD
  validation in CI.

## Repository layout

```
src/
  io/         loadWorkbook / saveWorkbook / workbookToBytes + Source/Sink
  node/       Node fs glue (fromFile / toFile / fromBuffer / ...)
  streaming/  read-only iter + write-only append
  workbook/   createWorkbook, addWorksheet, defined names
  worksheet/  setCell, getCell, mergeCells, tables, ...
  cell/       cell value-model + inline rich text
  styles/     fonts, fills, borders, alignment, number formats
  chart/      c: (legacy) + cx: (modern) chart kinds
  chartsheet/ standalone chartsheets
  drawing/    anchors, images, chart placement
  packaging/  OPC (Open Packaging Conventions)
  zip/        zip read/write, decompression-bomb defense
  xml/        XML reader/writer abstractions
  schema/     OOXML schema + types
  utils/      error hierarchy, shared helpers

tests/
  Mirrors src/ layout. Includes roundtrip tests against reference/openpyxl/
  fixtures (git submodule) and ECMA-376 conformance tests under conformance/.

reference/openpyxl/   git submodule — fixture corpus, do not edit
docs/                 documentation site source
.github/              CI workflows, issue/PR templates, template-compliance
.claude/skills/       workflow-specific guides Claude consults
```

## Operating context

- **Readers**: yourself in 6 months, a first-time contributor, an AI agent.
- OSS is not "code only I touch." Optimize for **a stranger not getting
  confused**, not for your own convenience.

## Core principles

Every line of code must be justified.

| Principle               | Meaning                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------ |
| Simplicity              | No premature abstraction, no features for hypothetical futures. YAGNI.               |
| Consistency             | Match existing patterns. New patterns require an explicit reason.                    |
| Performance             | Don't write N+1 / O(n²) in the first place. Defend with code shape, not profilers.   |
| Security                | Validate at boundaries (external input, file I/O, external APIs). Watch OWASP.       |
| Maintainability         | Write code that you in 6 months and a new contributor can read and understand.       |
| Backwards compatibility | Public API is preserved unless a breaking change is genuinely justified. Pre-1.0 we still try to call out breaks clearly. |

## "One way to do one thing"

> A capability has exactly one canonical path through the public API.

Do not add a **parallel API** — a second path that produces the same result as an
existing one — just because it's shorter, more discoverable, or "feels nicer." Reasons:

1. Every reader pays the "which one should I use?" cost on every code review and onboarding.
2. Two APIs = two of everything: docs, tests, types, bundle size, compatibility surface.
3. If the README says "use X for Y" but the library also accepts Z, the docs are lying.

**The rule bends** in two cases:

- An existing path is **misleadingly named** — rename it, don't parallelize.
  (Pre-1.0: rename outright. Post-1.0: deprecate + remove on next major.)
- A capability is reachable but only at an abstraction so low that every real user
  re-implements the same wrapper — graduate the wrapper into the library and
  hide the low-level path behind an "escape hatch" subpath. xlsx-kit's
  `xlsx-kit/xml`, `/zip`, `/packaging`, `/schema` subpaths exist as those
  escape hatches; new high-level features should extend `io`, `streaming`,
  `workbook`, `worksheet`, `cell`, `styles`, `chart`, `drawing` instead.

Decision flow when evaluating a feature request:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baseballyama/xlsx-kit](https://github.com/baseballyama/xlsx-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
