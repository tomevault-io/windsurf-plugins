---
trigger: always_on
description: **@office-kit/pptx** is a TypeScript library that generates `.pptx` (PowerPoint /
---

# CLAUDE.md

## Project overview

**@office-kit/pptx** is a TypeScript library that generates `.pptx` (PowerPoint /
Office Open XML Presentation) files from a typed object model. It runs in
**both Node.js and the browser** from a single ESM bundle, and produces files
that open cleanly in PowerPoint, Keynote, Google Slides, and LibreOffice
Impress.

The library has two complementary entry paths:

1. **From scratch** — build a presentation programmatically and emit a `.pptx`.
2. **From a template** — load an existing `.pptx`, mutate slides / placeholders
   / shapes, and emit the result.

The long-term goal is to support the full **OOXML PresentationML** spec
([ECMA-376 Part 1, §19](https://www.ecma-international.org/publications-and-standards/standards/ecma-376/))
plus the supporting DrawingML / SpreadsheetML pieces a presentation actually
references (charts, embedded media metadata, transitions, animations, theme,
slide masters / layouts, comments, notes).

## Tech stack

- **Language**: TypeScript (strict mode), targeting ES2022.
- **Runtimes**: Node.js >= 22.18 (Node 22 and 24 LTS lines) and modern browsers (Chrome / Firefox / Safari
  current-2). One ESM bundle, no Node-only built-ins on the hot path — use
  Web-standard `Uint8Array`, `TextEncoder/Decoder`, `CompressionStream` where
  available.
- **Package manager**: pnpm.
- **Build**: `tsdown` → ESM + `.d.ts`.
- **Test**: `vitest` (unit + golden-file fixture comparisons against real
  PPTX bytes).
- **Lint / format**: oxlint + oxfmt.
- **Release**: changesets + npm OIDC trusted publisher (see
  `.github/workflows/release.yml`).
- **Dependencies**: keep the runtime dep list **tiny**. ZIP and XML are the
  only two real third-party needs; everything else should be hand-rolled or
  not done at all. Every new runtime dependency must be justified in the PR.

## Domain-specific rules

### OOXML is the source of truth, not our types

The ECMA-376 spec defines the wire format. Our TypeScript model is a
convenience layer over it.

- When the spec and our types disagree, **the spec wins**. Fix the types.
- When PowerPoint and the spec disagree (and they do, often), **PowerPoint
  wins** — but leave a comment naming the specific behavior, because the next
  reader will not know.
- Round-trip safety matters: `parse(serialize(x))` must be structurally equal
  to `x` for everything we claim to support. Add a fixture test the first time
  a round-trip case comes up; do not rely on visual inspection.

### Output must be valid PPTX, not "valid enough"

A `.pptx` that opens in PowerPoint but crashes Keynote is a bug. A `.pptx`
that opens everywhere but is rejected by Microsoft's
[Open XML SDK Productivity Tool](https://github.com/dotnet/Open-XML-SDK) is
also a bug.

- All XML output must be schema-valid against the official XSDs.
- Required relationships (`_rels/*.rels`, `[Content_Types].xml`) are not
  optional. There is no shortcut where you "just skip the rels."
- IDs (`rId*`, shape IDs, drawing IDs) must be **unique and stable** within
  their scope. Generate them through the central ID allocator; never
  hand-roll a string concatenation.

### Cross-runtime correctness

This library runs in Node and the browser. That constrains what you can
import.

- No `fs`, `path`, `buffer`, `stream`, `zlib` on the hot path. Use Web
  primitives (`Uint8Array`, `Blob`, `CompressionStream`).
- If a feature genuinely needs the Node `fs` (e.g., reading a template from
  disk), put it behind a `node:` subpath export so the browser bundle does
  not pull it in.
- Do not assume a `Buffer` global. Use `Uint8Array` and convert at the
  boundary if a Node caller passes a `Buffer`.

### Public API is small and explicit

The library will be **the** way to author / edit PPTX in the JS ecosystem,
so the public surface needs to be defended.

- One canonical path per capability (see "One way to do one thing" below).
- Builder functions return new objects; mutation is reserved for the
  template-editing path, and it is explicit (`slide.addShape(...)`).
- All inputs are typed by discriminated unions when the OOXML schema has a
  choice (`a:srgbClr` vs `a:schemeClr` etc.). No "color is a string and we
  parse it." Parse, don't validate (see below).
- `internal/` is internal. If users import from it, the public API failed —
  fix the public API, do not stabilize the internal symbol.

## Operating context

- **Audience**: OSS — code that external contributors and end users will read.
- **Language**: TypeScript. Principles below are language-agnostic, but the
  examples in this repo are TS.
- **Readers**: yourself in 6 months, a first-time contributor, an AI agent.

OSS is not the same as "code only I touch." Optimize for **a stranger not getting
confused**, not for your own convenience.

## Core principles

Every line of code must be justified.

| Principle               | Meaning                                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------- |
| Simplicity              | No premature abstraction, no features for hypothetical futures. YAGNI.                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [office-kit/pptx](https://github.com/office-kit/pptx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
