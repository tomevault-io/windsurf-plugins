---
trigger: always_on
description: A commercially-sold **animated component library for React and Next.js** — accessible, reduced-motion-safe, RSC-safe, sold as editable source through a shadcn-compatible registry. The moat is production-readiness and a coherent catalog, not component count.
---

# AGENTS.md

A commercially-sold **animated component library for React and Next.js** — accessible, reduced-motion-safe, RSC-safe, sold as editable source through a shadcn-compatible registry. The moat is production-readiness and a coherent catalog, not component count.

> **Phase:** 🟡 Phase 2 (MVP). 60 catalog items across `@scope/{tokens,motion,react,sections}` + registry + docs site + Storybook + CI.
> **Component inventory is FROZEN** (2026-07-16 cleanup): no new components, blocks, packs, themes, or features. Work is cleanup, consistency, and reliability only. Cleanup tracker: [`docs/57-library-production-cleanup.md`](docs/57-library-production-cleanup.md).

## Source of truth

Canonical docs (start here). Detailed/record docs are linked from these.

| Topic | Canonical doc |
| --- | --- |
| Index | [`docs/README.md`](docs/README.md) |
| Architecture & boundaries | [`docs/architecture.md`](docs/architecture.md) |
| Component authoring | [`docs/component-authoring.md`](docs/component-authoring.md) |
| Registry authoring | [`docs/registry-authoring.md`](docs/registry-authoring.md) |
| Preview authoring | [`docs/preview-authoring.md`](docs/preview-authoring.md) |
| Accessibility | [`docs/accessibility-standard.md`](docs/accessibility-standard.md) |
| Motion | [`docs/motion-standard.md`](docs/motion-standard.md) |
| Responsive | [`docs/responsive-standard.md`](docs/responsive-standard.md) |
| Code quality | [`docs/code-quality-standard.md`](docs/code-quality-standard.md) |
| Release | [`docs/release-checklist.md`](docs/release-checklist.md) |
| Commercial delivery | [`docs/commercial-delivery.md`](docs/commercial-delivery.md) |
| Security | [`docs/security-model.md`](docs/security-model.md) |
| ADRs (durable decisions) | [`docs/adrs/`](docs/adrs/) |

## Precedence (when guidance conflicts)

1. Accepted ADRs & architectural boundaries.
2. Accessibility & performance standards.
3. Component API & design-token standards.
4. Project skills ([`.Codex/skills/`](.Codex/skills/)).
5. Taste skills (`design-taste-frontend`, `redesign-existing-projects` — third-party, visual guidance only; **do not modify** them). Project rules take precedence.
6. Task-specific instructions.

## Permanent invariants

- **Boundaries:** core React packages must not import Remotion, Node built-ins, or `next/*`. Remotion is video-only and isolated. Enforced by `pnpm lint`.
- **Packages** are ESM-only with `"use client"` preserved in client entry points ([ADR-0006](docs/adrs/0006-library-bundler.md), [ADR-0007](docs/adrs/0007-package-format.md)).
- **Accessibility (WCAG 2.2 AA) and reduced motion are release-blocking.**
- **Semantic tokens** over one-off values; never hardcode the brand or a placeholder `@scope/*` in visitor-facing output — brand comes from [`product.config.json`](product.config.json).
- **Motion for React** is the default engine; CSS for simple effects; heavy engines stay component-local and lazy. Motion is not in every component.
- **Registry source is customer-editable:** exact files/deps, no docs/preview/test/internal imports, no secrets. See [`docs/registry-authoring.md`](docs/registry-authoring.md).
- **Free source is public; Pro source is protected** and must never appear in public build output ([`docs/security-model.md`](docs/security-model.md)). No runtime license checks; no secrets in client code.
- **Clean-room only:** no competitor source, names, effects, or APIs copied. Renaming is not remediation.
- Documentation has one canonical owner per topic.

## Code quality (summary — full: [`docs/code-quality-standard.md`](docs/code-quality-standard.md))

- No `any`, no unsafe non-null assertions, no swallowed errors, no unhandled promises.
- Clean up every listener/timer/observer/animation loop. Pause continuous animation offscreen. No browser globals during render.
- Controlled/uncontrolled via shared `useControllableState`; stable `useId`s across SSR/CSR.
- Comments explain **why**, not what. One or two direct sentences. No marketing words, no commented-out code, no vague TODOs.
- Prefer readable local code over a generic helper used once.

## Operating model — two tracks

- **Default track:** [`registry-release`](.Codex/skills/registry-release/SKILL.md) — one-page brief → build → wire registry/preview/docs → lightweight release gate → mark Released/Experimental in [`docs/39-catalog-production-board.md`](docs/39-catalog-production-board.md). Status set: `Idea · Building · Preview-ready · Registry-ready · Released · Experimental · Deprecated`. No subjective scores.
- **Signature track:** heavier review is reserved for homepage centerpieces and complex Pro creative effects only ([`docs/36-premium-creative-component-strategy.md`](docs/36-premium-creative-component-strategy.md)). It is not the default.
- One component's polish does not block unrelated work. Free and Pro share the same accessibility/usability baseline.

### Shared-file editing rule (parallel work)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RMahammad/motiq](https://github.com/RMahammad/motiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
