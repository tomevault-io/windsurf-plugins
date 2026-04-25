---
trigger: always_on
description: Guidance for humans and AI agents working in `jalco-ui`.
---

# AGENTS.md

Guidance for humans and AI agents working in `jalco-ui`.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Project intent

jalco ui is a React component library by Justin Levine. It ships polished, composable components for React and Next.js projects, distributed via a shadcn-compatible registry so users can install with a single command or copy the source directly.

Goals:
- Keep the codebase polished, readable, and consistent.
- Optimize for public/open-source maintainability.
- Prefer simple, composable patterns over clever abstractions.
- Make examples, docs, and components production-quality.
- Use shadcn registry infrastructure for distribution, not as an identity constraint.

## Core principles

- Favor clarity over novelty.
- Keep diffs focused and reviewable.
- Match existing patterns before introducing new ones.
- Make documentation part of the feature, not an afterthought.
- Build for accessibility, composability, and copy-paste ergonomics.
- Avoid unnecessary dependencies.

## Repository standards

### General

- All files MUST use consistent naming across files, exports, components, and docs.
- Folder structure SHOULD be predictable and shallow where possible.
- Small helpers SHOULD be colocated with the feature that uses them.
- TypeScript MUST be used for all source files.
- Committed code MUST NOT contain dead code, commented-out code, or placeholder implementations.

### Components

- Components SHOULD be small and composable.
- Accessible primitives and interactions MUST be the default.
- Prop names MUST be clear with sensible defaults.
- APIs MUST NOT be overengineered before real usage justifies it.
- Registry items MUST be installable, readable, and easy to adapt.
- Public registry code and docs-facing source files MUST follow the comment and file header rules in `.pi/references/docs-component-format-spec.md`.
- When building or refactoring public jalco ui components, agents MUST use `.pi/skills/jalco-component-builder/SKILL.md` as the primary workflow skill.
- The `components-build` skill (`.pi/skills/components-build/SKILL.md`) is the canonical standard for component engineering — composition, accessibility, typing, styling, state, polymorphism, asChild, and data attributes. Agents MUST treat it as authoritative for all component architecture decisions.
- Agents SHOULD treat `shadcn-ui`, `.pi/skills/tailwind-design-system/SKILL.md`, `.pi/skills/vercel-composition-patterns/SKILL.md`, and `.pi/skills/vercel-react-best-practices/SKILL.md` as supporting references during component work.
- When building or maintaining registry infrastructure or items, agents MUST consult `.pi/skills/jalco-shadcn-registry/SKILL.md`.
- When creating or revising component documentation, agents MUST consult `.pi/skills/jalco-writing-component-docs/SKILL.md`.
- The Vercel React best-practices skill SHOULD be used for performance, rendering, data fetching, and Next.js architecture decisions.
- The Vercel composition patterns skill SHOULD be used for component API design, composition, compound components, and avoiding boolean-prop-heavy interfaces.
- The Tailwind design-system skill SHOULD be used for Tailwind v4 tokens, semantic styling, variant systems, theming, and design-system consistency.
- The Jalco shadcn registry skill SHOULD be used for item typing, `registry.json`, namespacing, authentication planning, MCP compatibility, Open in v0 considerations, and registry structure decisions.

### Component quality bar

Public jalco ui components MUST feel intentional, production-ready, and visually complete in their default state.

When designing or reviewing a component:
- MUST start from a clear use case, not from a styling trick or a variant list
- SHOULD prefer one strong layout idea over stacked decorative treatments
- SHOULD use spacing, typography, grouping, and alignment to create hierarchy before reaching for extra color, borders, shadows, or icons
- MUST keep APIs smaller and more semantic than the first draft
- SHOULD favor fewer, stronger variants over many shallow permutations
- MUST ensure the default example is the most compelling and broadly useful version
- MUST make demos feel like real product UI, not component laboratory output
- MUST NOT ship a public component unless it is visually distinct enough to justify its existence in the registry

A public variant MUST:
- represent a real use case or semantic difference
- have meaningful preview and docs coverage
- remain understandable from its API name alone

MUST NOT include:
- decorative-only variants
- prop-heavy APIs that expose internal styling decisions
- generic card wrappers with little opinion
- demos padded with badges, icons, or fake complexity to compensate for weak structure
- components whose examples look unfinished without consumer customization

### Component creation workflow

When building a new public component, agents MUST NOT jump straight to implementation. Every new component MUST be developed on a feature branch and merged to main via a PR that passes the component checklist.

#### Branch and PR requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jal-co/ui](https://github.com/jal-co/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
