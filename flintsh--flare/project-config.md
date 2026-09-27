---
trigger: always_on
description: Flare’s documentation is part of the product. A feature is not complete until its
---

# Working on Flare

Flare’s documentation is part of the product. A feature is not complete until its
documentation, examples, and relevant visual walkthroughs describe the behavior
that ships. Apply this requirement to **every feature and every commit that changes
observable behavior**, including fixes, removals, defaults, limits, and permissions.
Do not defer documentation to another task or release.

## Documentation location and boundaries

- The public handbook lives in `docs/site/`, an independently built VitePress site.
- Keep it in this repository so implementation and documentation ship together.
- Preserve automatic version/commit provenance on every page and in
  `build-info.json`. Never hardcode a release label or conceal uncommitted changes.
- Preserve the handbook’s creator credit linking to `https://fl1nt.dev`, its
  official 88×31 button, and xNefas’s icon attribution. Follow the
  [credit and asset guidance](docs/site/contributing.md#project-credits-and-the-author-button).
- The existing `docs/*.md` files are historical engineering notes and linked
  references. Keep any affected live references accurate; put new public guidance
  in the handbook, and cross-link rather than create a second canonical guide.
- Keep docs tooling out of Flare’s runtime dependencies and Docker image. Run
  `npm ci --prefix docs/site`; the docs have their own lockfile.
- Do not commit generated site output, dependency directories, copied screenshots,
  copied recordings, or caches. `prepare-assets.mjs` generates optimized assets
  from the existing source assets. Add only new visual evidence that is necessary.

## Required workflow for every change

1. **Map the impact before coding.** Read the current relevant guide and actual
   implementation. Identify affected personas (user, administrator, operator,
   integration author), defaults, access rules, failure cases, configuration,
   migrations, API requests/responses, and webhook events.
2. **Implement and document together.** Update all affected handbook pages in the
   same commit as the behavior change. Include what people can do, where to find
   controls, steps, expected results, important limits, and recovery instructions.
   Explain changes to existing installations and old integrations, not just fresh
   setup. Remove obsolete instructions. Update the feature explorer and sidebar
   when a capability or page is added, moved, or removed.
3. **Update executable contracts.** API changes require the endpoint inventory,
   relevant API guide, `public/openapi.json`, working request examples, and any
   affected request-builder controls. Webhook changes also require the event JSON
   Schema, signing examples, retry/ordering/deduplication guidance, and operator
   settings. Never imply named tokens authorize session-only or admin endpoints.
4. **Show real behavior.** If a visible workflow changes, capture or refresh its
   screenshots and/or recording using the real rendered application with isolated
   demonstration data. Inspect every image for stale controls, credentials, and
   private data. Update the tour/lab/feature explorer if affected. Label simulations
   as simulations. Do not fabricate app screenshots or present a simulation as a
   real server operation. Supply useful alt text and written video transcripts.
5. **Validate before committing or shipping.** Run the checks below, inspect changed
   pages at desktop and mobile sizes, follow important links, and actually execute
   changed commands/API examples against disposable fixtures when feasible. Build
   under a non-root base path too when changing navigation or assets. Review the
   docs diff against the implementation diff. No placeholder pages or TODO-only
   instructions count as coverage.
6. **Report the evidence.** In the PR or final delivery, name the affected guides,
   examples, visual evidence, and checks. Include real limitations. For changes
   with no user-visible effect, explain that conclusion in the PR and verify the
   relevant docs still match; do not make meaningless prose edits just to satisfy
   a check. The CI coverage gate still requires review of affected documentation
   areas for source changes; a short useful clarification or source verification
   note can record that review.

## Coverage matrix

| Change                                                                | Required documentation review/update                                                                        |
| --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Uploads, previews, sharing, files, tags, folders, pastes, short links | Relevant `guide/` pages, feature explorer, screenshots/tour, affected API references                        |
| Profile or account behavior, tools, preferences                       | Relevant `guide/` pages; token and email references where relevant                                          |
| Setup, instance settings, branding, access, moderation                | Relevant `admin/` pages; deployment and user guides if behavior crosses personas                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlintSH/Flare](https://github.com/FlintSH/Flare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
