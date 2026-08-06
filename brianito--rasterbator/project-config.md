---
trigger: always_on
description: Rasterbator is a private, offline, macOS-first desktop application for splitting one local image into a dimensionally accurate, multi-page A4 mosaic PDF. Version 1 exports the original image only, with configurable blank cutting margins, vector cutting guides, optional overlap, and page labels. The source aspect ratio is immutable in MVP: fitting may crop or add unused poster area, but it must never stretch or squash the image.
---

# Rasterbator Agent Guide

## Introduction

Rasterbator is a private, offline, macOS-first desktop application for splitting one local image into a dimensionally accurate, multi-page A4 mosaic PDF. Version 1 exports the original image only, with configurable blank cutting margins, vector cutting guides, optional overlap, and page labels. The source aspect ratio is immutable in MVP: fitting may crop or add unused poster area, but it must never stretch or squash the image.

The planned stack is Tauri 2, Rust, React, TypeScript, Vite, and Tailwind CSS. The Rust geometry engine is the canonical source of truth for both preview and PDF export. Do not duplicate output geometry in the frontend.

## Source of Truth and References

`AGENTS.md` is the source of truth for agent and contributor instructions. Supporting documents provide detailed requirements but do not override this guide. When an intentional decision changes these instructions, update `AGENTS.md` and every affected supporting document in the same change.

### Project specifications

Load only the context required for the current task. Search for the relevant section before reading a whole supporting document, and do not reread an unchanged document in the same session.

- **Scope or architecture changes:** read [PLAN.md](./PLAN.md), then every supporting document affected by the decision.
- **Rust geometry, validation, or page mapping:** read the relevant geometry and command sections in [PLAN.md](./PLAN.md) plus the Rust test layers in [TESTS.md](./TESTS.md).
- **Image decoding, rendering, or PDF export:** read the relevant processing/PDF sections in [PLAN.md](./PLAN.md), the export sequence in [USER_FLOW.md](./USER_FLOW.md), and the applicable integration-test sections in [TESTS.md](./TESTS.md).
- **React UI, styling, accessibility, or interaction:** read the relevant sections in [DESIGN.md](./DESIGN.md) and [USER_FLOW.md](./USER_FLOW.md), plus applicable frontend test sections in [TESTS.md](./TESTS.md).
- **Packaging, repository, documentation, or release mechanics:** read the files being changed and the release/manual-gate sections in [TESTS.md](./TESTS.md). Read product documents only when release scope or claims change.
- **Small isolated fixes:** inspect the affected code, its nearest tests, and only the specification section governing that behavior.

Reference documents:

- [Implementation plan](./PLAN.md) — product scope, architecture, geometry, milestones, and definition of done.
- [UI design plan](./DESIGN.md) — layout, components, styling, accessibility, motion, and design acceptance criteria.
- [User flow](./USER_FLOW.md) — complete import-to-export journey, failure paths, keyboard flow, and flow acceptance criteria.
- [Test strategy](./TESTS.md) — canonical automated test layers, fixtures, milestone coverage, CI gates, and manual release checks.

When supporting documents appear to conflict, use `PLAN.md` for product and technical scope, `DESIGN.md` for presentation, `USER_FLOW.md` for interaction sequencing, and `TESTS.md` for testing implementation. `AGENTS.md` takes precedence for agent and contributor behavior. Record unresolved conflicts before implementing assumptions.

### External documentation

- [Tauri 2](https://v2.tauri.app/)
- [React](https://react.dev/)
- [TypeScript](https://www.typescriptlang.org/docs/)
- [Vite](https://vite.dev/guide/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Rust `image` crate](https://docs.rs/image/)
- [Rust `printpdf` crate](https://docs.rs/printpdf/)
- [Rust `pdf-writer` crate](https://docs.rs/pdf-writer/)
- [Motion for React](https://motion.dev/docs/react)
- [Remix Icon React](https://github.com/Remix-Design/RemixIcon)

## Workflow

1. **Review the specifications**
   - Identify the relevant milestone and acceptance criteria.
   - Keep MVP work limited to macOS, A4, offline processing, and original-image output.
   - Update the documents when an intentional product or architecture decision changes.

2. **Work milestone by milestone**
   - Complete Milestone 0 technical spikes before committing to a PDF crate.
   - Build and test the Rust geometry engine before reproducing its results in the UI.
   - Implement import and settings before preview, then implement export and hardening.
   - Do not begin deferred effects, other paper sizes, or other platforms during MVP work.

3. **Implement in small vertical slices**
   - Define or update canonical Rust types and validation.
   - Add geometry/processing tests before wiring Tauri commands.
   - Expose a small, typed command boundary.
   - Mirror or generate TypeScript contracts without copying geometry formulas.
   - Add UI behavior, translated English strings, validation, loading, and error states.

4. **Validate every change**
   - Follow the test layers, fixture policy, milestone coverage, and gates in [TESTS.md](./TESTS.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrianIto/rasterbator](https://github.com/BrianIto/rasterbator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
