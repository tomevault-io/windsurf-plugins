---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose
The Summer EBT (SUN Bucks) Self-Service Portal allows parents/guardians to manage Summer EBT benefits for eligible children. The portal supports multiple states (DC, CO) via a plugin architecture. See [README.md](./README.md) for full background and setup instructions.

## Interaction norms
We're colleagues working together. Neither of us is afraid to admit we don't know something or are in over our head. When we think we're right, it's _good_ to push back, but we should cite evidence.

## Code-Authoring norms
- We prefer simple, clean, maintainable solutions over clever or complex ones, even if the latter are more concise or performant. Readability and maintainability are primary concerns.
- Doing it right is better than doing it fast. You are not in a rush. NEVER skip steps or take shortcuts.
- Stay focused. Fix only what relates to your current task. Notice something else that needs work? Document it separately rather than fixing it now.
- Preserve comments. They're documentation, not clutter.
- Write evergreen code. Describe what code does, not when it was written. (i.e. avoid "newFunction")
- All user-facing strings must go through i18next. Never hardcode display text in components — reference keys via the translation functions.
- **Locale JSON files are generated — NEVER hand-edit them.** They are produced by `packages/design-system/content/scripts/generate-locales.js` from CSV exports in `packages/design-system/content/states/`. To add or change content: update the source Google Sheet, re-export the CSV, and re-run the generator (run `pnpm copy:generate` from within `src/SEBT.Portal.Web/` or `src/SEBT.EnrollmentChecker.Web/`). This also runs automatically via the `predev` and `prebuild` hooks. If a key is missing, note it as a content gap to resolve in the spreadsheet — do not add it directly to the JSON.

### Code style
- C#: 4-space indent, Allman brace style (braces on own line), nullable reference types enabled (see `.editorconfig`)
- Frontend: TypeScript (not JavaScript). ESLint + Prettier with organize-imports plugin
- Unix line endings (LF) enforced project-wide

### Frontend styling
- **Avoid inline `style={...}` props.** They bypass the design-token system, can't be themed per state, are hard to override, and don't compose with USWDS modifier classes.
- **Reach for shared design-system components first** (`Button`, `InputField`, `Alert`, … from `@sebt/design-system`) before composing your own from raw HTML + USWDS classes. They already encapsulate ARIA wiring, USWDS class composition, and per-state theming — re-using them keeps behavior consistent and prevents accessibility drift. Use a `<button>` with `usa-button` only when no shared component fits, and consider whether the design system should be extended instead.
- **When no shared component fits, prefer USWDS component classes** (`usa-button`, `usa-input`, `usa-form-group`, `usa-combo-box__list`, …) before writing custom CSS.
- **Use USWDS utility classes** (`position-relative`, `margin-bottom-2`, `text-center`, `display-flex`, …) for layout, spacing, and one-off style needs. The full utility set is generated from our design tokens, so utilities stay in sync with the per-state theme.
- When none of the above fits, add SCSS in a co-located `.scss` file that references USWDS tokens (`@use 'uswds-core' as *;` and the `units()` / `color()` helpers). Don't hardcode colors, spacing, or font sizes.

## Getting help
- If you're confused or having trouble with something, you are strongly encouraged to stop and ask for help. Especially if it's something your human might be better at.

## Decision-Making Framework
### 🟢 Proceed Immediately
- Fix tests, linting errors, type errors
- Implement single functions with clear specs
- Correct typos, formatting, documentation
- Refactor within a single file to improve clarity
- Add missing imports or dependencies

### 🟡 Propose First
- Changes spanning multiple files
- New features or significant functionality
- API or interface changes

### 🔴 Always Explicitly Ask a Human First!
- Rewriting working code from scratch
- Changing core business logic or removing functionality
- Architectural changes. Architectural decisions are recorded as ADRs in [docs/adr/](./docs/adr/). Consult existing ADRs before proposing changes that affect architecture. Note: some ADR numbers are accidentally reused (e.g., 0007) and may be cleaned up later — always reference ADRs by filename, not number.
- Security modifications

## Designing Solutions
### 1. Build for composition
- Each service delivers one focused capability.
- When proposing major new functionality, ask: "should this be a separate service?". Be pragmatic, but default to yes if the capability is clearly independently useful. Still, always request confirmation from a human, and if the human declines a separate service, build in a highly modular way that will allow for easy extraction of a composable service later.
### 2. API-first design
- Services expose documented REST APIs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codeforamerica/sebt-self-service-portal](https://github.com/codeforamerica/sebt-self-service-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
