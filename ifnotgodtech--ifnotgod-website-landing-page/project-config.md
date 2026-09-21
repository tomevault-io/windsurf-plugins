---
trigger: always_on
description: Engineering standards for the IFNOTGOD TECH LTD marketing website: a single-page, conversion-focused corporate site (see project brief in `CLAUDE.md`). No auth, no user accounts, no backend domain beyond capturing and delivering contact/lead submissions.
---

# AGENTS.md - IFNOTGOD TECH LTD

Engineering standards for the IFNOTGOD TECH LTD marketing website: a single-page, conversion-focused corporate site (see project brief in `CLAUDE.md`). No auth, no user accounts, no backend domain beyond capturing and delivering contact/lead submissions.

## Engineering Standards
- Use TypeScript for all application code.
- Use Next.js App Router (`src/app`) for all route definitions.
- Keep the page composed from section components matching the brief's page structure (Nav, Hero, Problem, Solutions, AI, How We Work, Why Us, Who We Help, About, Conversion CTA, Contact, Footer).
- Validate incoming data at boundary layers (client form and server route handler).
- Favor explicit types for domain entities (`ContactSubmission`).
- All modules must be thoroughly tested.
- Minimum expectation:
  - unit tests for form validation logic and edge cases
  - integration test for the contact submission API route
  - UI interaction tests for form states (idle, submitting, success, error)

## Suggested Folder Structure (Next.js Reference)
- `src/app` (App Router: root layout, the single page, `api/contact` route handler)
- `src/components/sections` (Hero, Problem, Solutions, AI, HowWeWork, WhyUs, WhoWeHelp, About, ConversionCta, Contact, Footer, Navigation)
- `src/components/ui` (shared reusable UI primitives: Button, Card, Input, TextArea, Badge)
- `src/lib` (contact-submission service, email/notification client, config)
- `src/shared` (constants, validators, shared helpers)
- `src/types` (`ContactSubmission` and other shared contracts)
- `src/styles` (global styles, theme tokens)

## Next.js Architecture Rules
- Keep route concerns in `src/app`; keep section composition and any submission logic in `src/components/sections` and `src/lib`.
- Prefer Server Components by default; use Client Components only where interactivity is required (contact form, nav hamburger, scroll/fade animations).
- Co-locate section-specific UI/hooks within `src/components/sections`.
- Use shared UI components from `src/components/ui` to maintain design consistency.

## Performance Standards
- Optimize for Core Web Vitals on the landing page:
  - LCP under 2.5s on standard broadband
  - INP under 200ms for core interactions (nav, form, CTAs)
  - CLS under 0.1
- Use lazy loading for below-the-fold media and non-critical sections.
- Minimize client bundle size:
  - avoid unnecessary client components
  - use dynamic imports for heavy optional UI (e.g. animation-heavy sections)
- Optimize images (hero visuals, icons) using Next.js image optimization patterns.
- Monitor performance regressions in staging before release.

## Delivery Priorities (MVP)
1. Page scaffold, layout, navigation (with sticky behavior + mobile hamburger)
2. Hero + Problem + Solutions sections
3. AI section + How We Work + Why Us + Who We Help
4. About + Conversion CTA section
5. Contact form + submission handling (API route, validation, delivery of leads)
6. SEO/meta tags, performance pass, full responsive QA

## Definition of Done (Feature Level)
A feature/section is done when:
- Functional acceptance criteria from the brief are met (copy, CTAs, layout intent)
- Client and server validation checks are in place for any data entry
- Error, empty, loading, and success states are handled where applicable
- Tests are implemented and passing for affected modules
- Documentation is updated if behavior or structure changes

## Change Management
- Keep the `ContactSubmission` shape synchronized between client form, validator, and API route.
- Any change to the contact/lead capture flow must consider what happens to in-flight or failed submissions (no silent data loss).
- Update this `AGENTS.md` when section structure, naming, or module boundaries change.

## Resource Cleanup Rules
- Always clean up subscriptions, timers, event listeners, observers, and custom browser integrations.
- Clean up side effects properly on unmount and dependency changes.
- Abort stale requests when needed.
- Prevent state updates after unmount.
- Do not leak scroll/intersection observers across re-renders.

## Heavy Work Rules
- Never block the browser main thread.
- Do not perform expensive transformations inside render paths.
- Prefer server-side rendering for static content; keep client-side JS limited to interactivity and animation.

## Network Rules
- Use a centralized HTTP/client abstraction for the contact submission call.
- Handle errors with typed exceptions or normalized failures.
- Normalize the contact form payload before it reaches the API route.
- Do not let raw request/response shapes leak into presentation code.

## Caching and Data Ownership Rules
- Static marketing content is source-of-truth in code (or CMS, if later added) — not duplicated in client state.
- Keep any filter/tab UI state (if introduced, e.g. solutions filtering) in local state; use URL state only if it needs to be shareable.
- Derived UI state should be computed, not redundantly stored.

## Data Parsing and Mapping Rules
- Do not perform expensive data shaping inside components.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ifnotGodTech/ifnotGod_website_landing_page](https://github.com/ifnotGodTech/ifnotGod_website_landing_page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
