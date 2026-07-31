---
trigger: always_on
description: This repository uses `Quiet Pro` as the only long-term UI design baseline.
---

# AGENTS.md

This repository uses `Quiet Pro` as the only long-term UI design baseline.

This file is the top-level collaboration entry point for repository-aware agents.

These instructions apply to all UI work unless the user gives an explicit task-specific override.

## Always Read First

- Product direction and scope must follow `docs/product-principles-and-scope.md`.
- Roadmap and priority decisions must follow `docs/roadmap-and-prioritization.md`.
- Engineering quality direction should follow `docs/engineering-quality.md`.
- UI work must follow `docs/quiet-pro-component-guidelines.md`.
- Architecture refactors, boundary decisions, and new modules must align with `docs/architecture.md`.
- Stable-period issue fixes and boundary triage must follow `docs/issue-fix-boundary-guardrails.md`.
- Versioning, changelog, and release work must follow `docs/versioning-and-release-policy.md`.
- Treat the top-level long-lived docs under `docs/` as the current source of truth.

## Quiet Pro Baseline

- Build calm, professional, restrained desktop-product UI.
- Prefer typography, spacing, alignment, and hierarchy over decoration.
- Keep the interface neutral and durable rather than flashy or brand-heavy.
- New UI should feel native to the existing Dashboard, History, App Mapping, and Settings surfaces.

## Hard Rules

- Do not introduce glassmorphism, blur-heavy panels, neon glow, or large gradient backgrounds.
- Do not hardcode new colors, radii, shadows, or border styles when a token or semantic variable should exist.
- Do not add one-off visual treatments that only work on a single page.
- Do not make components louder than the information they present.
- Do not trade readability or efficiency for "design feeling".

## Token And Styling Rules

- Reuse existing semantic tokens first.
- If a new visual role is needed, add or extend a token instead of hardcoding a value in a component.
- Keep radius, border, elevation, and motion within the existing Quiet Pro scale.
- Category or status colors may vary by feature, but surrounding chrome must stay within the Quiet Pro system.

## Component Rules

- New components must define clear `default`, `hover`, `active`, `focus`, `disabled`, and where relevant `loading` and `empty` states.
- Prefer existing component archetypes: `panel`, `control`, `chip`, `status`.
- Icons support recognition; they should not become the main visual focus.
- Dense pages may be efficient, but they must still scan cleanly at a glance.

## Implementation Preference

- Extend the design system before inventing a page-local workaround.
- Preserve existing product behavior unless the user explicitly asks for interaction changes.
- If a proposed UI change conflicts with Quiet Pro or requires a new visual direction, pause and confirm before proceeding.

## Architecture Direction

- Follow `docs/architecture.md` as the architecture mother document.
- Frontend long-term structure is `app / features / shared / platform`.
- Rust long-term structure is `lib.rs + app / commands / platform / engine / data / domain`.
- Keep Tauri command handlers thin; do not let `commands/*` or `lib.rs` regrow thick business logic.
- Prefer owner-first placement: decide the real owner before deciding the file or layer.
- `shared/*` is only for stable shared capability, not a temporary bucket.
- `platform/*` is for explicit external-environment boundaries, not a generic dump for hard problems.
- Do not reintroduce exited root layers such as `src/lib/` or `src/types/`.
- Treat compatibility shells and forwarding layers as explicit exceptions that should stay thin.
- Treat files under `docs/archive/` as historical context, not the default source of truth.

## Product And Priority Direction

- Keep the product centered on personal, local-first, Windows desktop time tracking.
- Prioritize trust, readability, control, and long-term usability over feature count.
- Do not quietly expand the product toward team SaaS, cloud-first workflows, mobile-first usage, or gamified productivity unless the user explicitly changes product direction.
- When multiple directions compete, prefer correctness, data safety, and high-frequency core flows before expansion work.

## GitHub Project Active Maintenance

- When the current request clearly maps to an existing Project item, proactively read the live Project before implementation even if the user does not separately mention Project maintenance.
- For a newly approved item, create one complete draft item at the bottom of the Project, fill its long-form scope and fields, set its confirmed initial status, verify the live result, and report the recommended manual position. Do not move or reorder it after creation.
- During implementation of an existing item, do not mutate its Project status on the maintainer's behalf. Tell the maintainer exactly which item should move from its current status to `In progress`, `Blocked`, `Done`, `Next`, or `Queued`, and explain the triggering event.
- After every start, completion, block, or unblock event, recompute the `Next` window according to `docs/roadmap-and-prioritization.md` and report all required drag operations together. The maintainer performs the actual status changes in the Project `Board` view.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ceceliaee/time-tracking](https://github.com/Ceceliaee/time-tracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
