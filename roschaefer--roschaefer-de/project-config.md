---
trigger: always_on
description: This repository contains the current `roschaefer.de` portfolio site. Treat this file as durable project guidance for coding agents and automation. For human-facing setup and deployment details, start with `README.md`.
---

# Agent Notes

This repository contains the current `roschaefer.de` portfolio site. Treat this file as durable project guidance for coding agents and automation. For human-facing setup and deployment details, start with `README.md`.

## Product Intent

The site serves two purposes:

1. A web portfolio for recruiters, hiring managers, engineering leads, peers, and collaborators.
2. A high-quality CV/PDF derived from the same resume content source.

Core goals:

- Clarify Robert's positioning quickly for hiring and collaboration contexts.
- Keep the web portfolio and downloadable CV consistent without duplicating content by hand.
- Support German and English, with German treated as the primary audience language.
- Keep the site privacy-conscious, accessible, static-first, and easy to maintain.
- Prefer test-driven changes for data transformations, rendering logic, and critical UI behavior.

Content should stay focused on value and evidence:

- About/bio
- Experience with impact
- Featured projects
- Skills and technology experience
- Open source
- Talks/media where relevant
- Contact
- Resume PDF
- Legal pages

Each section should answer "why this matters" or "what was built". Prefer concrete outcomes over buzzwords.

## Information Architecture

The main portfolio page should remain scannable and direct:

- Hero with name, role, positioning, and primary calls to action.
- Experience with role impact, scope, and relevant stack.
- Featured projects with problem, solution, stack, result, and links.
- Skills and technology experience, including derived years of experience where defensible.
- Open source as a first-class section, not just a link list.
- Talks/media with privacy-friendly behavior.
- Contact options.
- Footer navigation to legal pages.

Optional content can be added when it has enough substance:

- Writing
- Testimonials
- Current-focus/"now" content
- Project detail pages

## Visual And UX Direction

Preserve the current site's visual identity unless a change improves clarity, accessibility, responsiveness, or PDF/print output.

Current direction:

- Dark portfolio identity with cool blue/cyan accents.
- Strong display typography with readable body copy.
- Generous spacing and clear section hierarchy.
- Subtle motion only where it improves orientation or polish.
- Motion must respect `prefers-reduced-motion`.
- Interactive enhancements must degrade to readable static content.

Do not add visual changes just to restyle the site. Make departures intentional and explain the user-facing reason.

## Content Source Of Truth

The authored resume source is:

- `resume.i18n.json`

There are no generated `resume.de.json`/`resume.en.json` files committed to the repo — they're gitignored and regenerated fresh by `pnpm build:json-resume` (`scripts/generate-resume-source.ts`):

1. Decrypt `resume.i18n.json` (`sops -d`) and mask redacted-client fields (see "Redacted Clients" below), writing the result to `.generated/resume-source.json` (locale-agnostic, never committed).
2. `deriveResume` (`src/lib/utils/derive-resume.ts`, a pure function) localizes that into a single locale's JSON Resume data, sorts dated sections, and computes skills. Written to `.generated/resume.de.json` / `.generated/resume.en.json`.

This avoids keeping derived copies in sync, and avoids stale generated output (e.g. duration text computed from "now") ever getting committed. `build:json-resume` is the only place `sops`/decryption is ever invoked — everything downstream (`src/lib/data/resume.ts`, `scripts/build-typst-data.ts`, tests) just reads the generated per-locale files as plain data, so none of it needs to be Node-only or server-only.

The pipeline is:

- `resume.i18n.json` -> (`pnpm build:json-resume`, decrypt + mask, one Node script) -> `.generated/resume-source.json`
- `.generated/resume-source.json` -> (`deriveResume`, pure, once per locale) -> `.generated/resume.de.json` / `.generated/resume.en.json`
- `.generated/resume.<locale>.json` -> web rendering (`src/lib/data/resume.ts` imports both generated files directly)
- `.generated/resume.<locale>.json` -> Typst-ready data (`scripts/build-typst-data.ts` imports the same generated files directly)

By default (`RESUME_MODE` unset, or `RESUME_MODE=masked`) real client identities never leave `generate-resume-source.ts`. `RESUME_MODE=unredacted pnpm build:json-resume` restores real values into the same gitignored `.generated/` files instead, for local, personal use (e.g. printing your own real CV) — never use this mode for a build that gets committed, deployed, or shared, and regenerate the masked version afterward before running `dev`/`build` again.

Guidelines:

- Anything that needs localized resume data should import `.generated/resume.<locale>.json` directly, the same way `resume.ts`/`build-typst-data.ts` do — never add another parallel decrypt path.
- Keep German and English resume structures aligned.
- Validate both localized outputs against the same TypeScript/schema model.
- Use stable `id` fields for translatable entries such as projects and talks.
- IDs should not change just because a title or URL changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roschaefer/roschaefer.de](https://github.com/roschaefer/roschaefer.de) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
