---
trigger: always_on
description: This is the documentation repository for evcc (Electric Vehicle Charge Controller).
---

# AI Assistant Instructions for evcc Documentation

## Project Overview

This is the documentation repository for evcc (Electric Vehicle Charge Controller).
The site is built with [Astro](https://astro.build/) and the [Starlight](https://starlight.astro.build/) docs theme.
Device pages are generated from YAML templates that are synchronised with the main evcc repository.

## Repository Structure

```
/astro.config.mjs                   # Astro + Starlight config (sidebar, locales, plugins)
/src/content.config.ts              # Content collections (docs + device templates)
/src/content/docs/en                # English documentation (default locale)
/src/content/docs/de                # German documentation
/src/content/docs/{lang}/blog       # Blog posts per locale
/src/pages/[lang]/...               # Auto-generated device list & detail pages
/src/components                     # Astro components (.astro)
/src/utils                          # Shared TS helpers (devices, usages, icons)
/src/styles/custom.css              # Theme overrides
/src/scripts                        # Build-time scripts (legacy redirect emitter)
/templates                          # YAML device templates (input for device pages)
/public                             # Static assets served as-is (favicon, openapi.yaml, …)
```

### Documentation content (`/src/content/docs/{en,de}`)

Most content is **manually written**, except for:

- **Device list pages** — `chargers`, `meters`, `vehicles`, `smartswitches`, `heating`, `tariffs` — rendered by routes under `src/pages/[lang]/` using `src/utils/devices.ts` against the YAML templates.
- **Device detail pages** — same routes, one per template.
- **CLI reference** — `reference/cli/*.md` generated from the main evcc repository (English only, German falls back automatically).
- **REST API** — generated from `public/openapi.yaml` by `starlight-openapi`.

### Templates (`/templates`)

YAML device templates organised by:

- Channel: `nightly/` and `release/`
- Language: `de/`, `en/`
- Category: `charger/`, `meter/`, `vehicle/`, `tariff/`

The nightly/release toggle in the UI switches which collection is used at runtime.

### Internationalization

- **English is the default locale** (`defaultLocale: "en"` in `astro.config.mjs`).
- German content lives under `src/content/docs/de/`.
- Pages missing in one locale fall back to the default locale (English) automatically; the URL still reflects the requested locale.
- Sidebar group labels are translated via the `translations` field in the sidebar config in `astro.config.mjs`.
- Header UI strings come from Starlight's built-in i18n; only locale-specific overrides go in our components (e.g. `src/components/SiteTitle.astro`, `LanguageSelect.astro`).

### Auto-generated content

#### Device & tariff pages

1. Templates in `/templates/{channel}/{lang}/{category}/`
2. Loaded via content collections defined in `src/content.config.ts`
3. Rendered by routes in `src/pages/[lang]/{chargers,meters,vehicles,smartswitches,heating,tariffs}/`
4. Shared layout & helpers in `src/components/DeviceDetailLayout.astro` and `src/utils/devices.ts`

#### CLI documentation

1. Generated from the main evcc repository — see "Update CLI docs" in `README.md`
2. Output to `src/content/docs/en/reference/cli/*.md` (English only; German falls back)

#### REST API

1. Spec at `public/openapi.yaml`
2. `starlight-openapi` plugin (configured in `astro.config.mjs`) renders the pages under `integrations/rest-api`

### Manual content

Everything else is hand-written:

- Introduction, installation, features, integrations, reference (config & Modbus), FAQ, sponsorship
- Blog posts

## Writing Style Guide

### Product Name

- **Always write "evcc" in lowercase** — even at the beginning of sentences
- Never use "EVCC", "Evcc", or other variations
- Also lowercase "evcc" in external titles (e.g. the media list): rewrite "EVCC" → "evcc" even if the original title uses uppercase
- Prefer "your evcc instance" over a bare "evcc" when referring to a running instance
- Don't mention "evcc" unless necessary for context — within evcc docs the reader already knows. Drop self-referential fluff like "another strength of evcc", "evcc ships with", "the evcc UI"

### Language & Tone

- **Be informal and casual** — address readers directly with "you" (English) or "du" (German)
- Direct address only for actions the reader actually performs (UI steps, setup instructions). For technical requirements use an impersonal subject: "Das Plugin muss den Zählerstand liefern", not "Liefere den Zählerstand"
- Write complete, natural sentences — no telegraphic or clipped phrasing, no semicolon-jammed clauses
- Write for individual professionals, not businesses
- Avoid corporate or marketing language (e.g. don't use words like "bequem", "convenient", "seamlessly")
- Be concise and direct; describe behaviour, not internals (no libraries / quotas / internal naming schemes in user-facing docs)
- Don't pitch the absence of analytics/telemetry as a feature — frame it as the default

### Terminology


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evcc-io/docs](https://github.com/evcc-io/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
