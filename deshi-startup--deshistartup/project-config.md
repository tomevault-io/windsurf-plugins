---
trigger: always_on
description: Read this before changing the project. For priorities and planned content, start at
---

# Deshi Startup repository guide

Read this before changing the project. For priorities and planned content, start at
[`plan/README.md`](./plan/README.md). For human contribution steps, use
[`CONTRIBUTING.md`](./CONTRIBUTING.md).

## Mission and scope

Deshi Startup is a free, open-source operating manual for founders building new, scalable
businesses in Bangladesh, published in Bangla and English. Each completed guide is available in
both languages.
Registration, tax, payments and hiring guides may also help small businesses, but the project
does not broaden its scope to become a generic SME, family-business, import/export or online-seller
portal.

English is the canonical authoring edition; the Bangla edition is translated from the finished
English guide with the `translate-bangla-guide` skill. Both editions use the same standards for
evidence, accuracy and practical usefulness. A page counts as written only when it is a real guide
without `<StubNotice />`; run `npm run backlog:status` for current counts.

## Architecture

- Next.js + Nextra render mostly static MDX content.
- Next.js exports the site to `out/`; Cloudflare Static Assets serve it without invoking the Worker.
- A small native Worker handles the contact endpoint, contribution APIs and legacy review-link redirects.
- Pagefind supplies client-side static search.
- Milkdown Crepe powers the inline editor.
- `jose` verifies Google ID tokens on every contribution request.

Key paths:

- `app/(contents)/(bn)/` – Bengali pages at clean root URLs.
- `app/(contents)/en/` – matching English pages under `/en`.
- `app/components/LocalizedLayout.tsx` – shell, navigation, page chrome and editor entry.
- `app/components/ContributionEditor.tsx` – browser editor and draft recovery.
- `worker/api/` and `worker/lib/` – contact, contribution, authentication, GitHub and media-review logic.
- `worker/index.ts` – explicit API router and static-asset fallback.
- `data/directory/` – structured directory entries.
- `data/glossary.json` – the one glossary source, read by the glossary page and every `<Term>`.
- `plan/content-backlog.csv` – canonical planned-topic and route registry.
- `app/nav.config.ts` – curated top-level navigation.
- `app/nav-groups.json` – section-hub groups.
- `app/generated/` and generated files in `public/` – build outputs; never hand-edit them.

## Routes and content trees

Content URLs have at most two semantic segments, mirrored exactly in both locales. The
`Path` column in `plan/content-backlog.csv` owns permanent planned URLs. Internal content links are
always root-relative:

- Bengali: `/registration/private-limited`
- English: `/en/registration/private-limited`

Do not derive a URL from an editable title. Do not use relative links such as `../page`.
`npm run lint:routes` enforces depth, charset, mirror parity and `<StubNotice path>`.

Section hubs use `<SectionIndex section="..." locale="..." />`; do not maintain page lists in MDX.
Add a new top-level destination to `app/nav.config.ts`, and a new section child to the appropriate
group in `app/nav-groups.json`.

## Writing a page

Before writing or editing any Bangla anywhere in the repository, including public documentation,
metadata and UI copy, read [`STYLE.md`](./STYLE.md).

Before writing content, read:

- [`EDITORIAL.md`](./EDITORIAL.md) for the quality standard: standalone pages, sources, visuals,
  and the five finish gates;
- [`plan/guide-playbook.md`](./plan/guide-playbook.md) for the English-first pipeline and the
  visual toolkit (`DataBars`, `Waterfall`, `Timeline`, `Figure`, `YouTube`, calculators);
- [`STYLE.md`](./STYLE.md) for natural Bangladeshi Bangla, used when translating; and
- the finished `/en/operations/cod-risk`, `/en/metrics/unit-economics` and
  `/en/metrics/cashflow-vs-profit` pages for working examples of the full standard.

Write the English guide first and finalise it against the five gates before translating to
Bangla.

Default guide shape:

1. frontmatter with `title` and `description`;
2. one `#` heading;
3. `> **সারকথা:**` / `> **In short:**`;
4. the decision, steps, cost/time, mistakes and checklist the topic actually needs; and
5. `## প্রাসঙ্গিক সোর্স` / `## Relevant Sources`.

Use official sources for legal, tax, fee, registration and regulatory claims. Date changeable
numbers. Never fabricate a statistic, quote, example or anecdote. Do not bump `verified:` unless
the relevant claims were re-checked against official sources.

Use GFM footnotes for inline citations: put `[^source-name]` beside the claim and define the same
identifier under the sources heading as `[^source-name]: [Source title](https://example.com)`.
Identifiers use lowercase ASCII words separated by hyphens. Numbering and backlinks are generated.

Page types with separate rules:

- Case studies use [`plan/case-study-format.md`](./plan/case-study-format.md).
- Journeys order existing guides and must not link missing routes.
- Directory pages render `data/directory/*.json`; do not hand-maintain prose tables.
- The glossary at `/start-here/glossary` renders `data/glossary.json` through `<Glossary />`;
  add or edit a term there, never as prose in the page. One entry feeds the A–Z page, the inline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deshi-Startup/deshistartup](https://github.com/Deshi-Startup/deshistartup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
