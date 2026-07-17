---
trigger: always_on
description: This repository is for building a SITCON Flickr photo finder: a lightweight photo index and future search workflow for helping SITCON organizers find usable public event photos.
---

# AGENTS.md

## Project Context

This repository is for building a SITCON Flickr photo finder: a lightweight photo index and future search workflow for helping SITCON organizers find usable public event photos.

The main goal is not to replace Flickr. The repository should keep a practical index layer on top of Flickr so organizers can search by real work needs such as social promotion, website visuals, sponsor proposals, sponsor fulfillment evidence, press materials, recap posts, and design assets.

## Repository Map

Use `docs/README.md` as the current documentation index, implementation status, and source-of-truth map. Do not maintain a second full artifact inventory in this file.

Key areas:

- `README.md`: human-facing project overview and quick start.
- `docs/`: architecture, workflow, data-entry, Apps Script, public frontend, GA4, and AI labeling documentation.
- `app/`: GitHub Pages and local static public finder UI.
- `apps-script/`: Google Apps Script source for Sheets-side maintenance helpers. `GeneratedConfig.js` is generated from repo schema and taxonomy.
- `config/project.json`: project-level organization, Flickr, Sheets, Apps Script, and GA4 identifiers. These identifiers are not credentials.
- `data/`: machine-readable schema, taxonomy, search aliases, validation messages, sponsorship snapshot, and AI sampling plan.
- `fixtures/`: local samples, export-format references, and validator fixtures. These files are not authoritative production data.
- `scripts/commands/` and `scripts/workflows/`: CLI tools and guided workflows. Prefer `pnpm workflow` or `docs/README.md` before invoking low-level scripts directly.
- `prompts/ai-labeling.md`: reusable AI labeling prompt template.

## Data Principles

- Google Sheets is the authoritative photo index. If Google Sheets and repo sample data disagree, Google Sheets wins.
- This repo is the governance and tooling layer: schema, taxonomy, validation, import/export scripts, Apps Script source or generators, AI prompts, and maintenance documentation.
- Keep reusable organization-specific values in `config/project.json` when practical. SITCON is the default instance, but the project should remain forkable by other organizations.
- `config/project.json` may include the public Google Sheets `spreadsheetId`. This is not treated as a secret for this project; write access is managed by Google Drive/Sheets permissions.
- Google Sheets tab names are fixed for the 1.0 workflow: `photos`, `albums`, `import_batches`, `taxonomy`, and `sponsorship_items`.
- Use the official Google Sheets API SDK as the primary direction for repo CLI operations that read or write Sheets tabs, ranges, appends, batch updates, and read-back verification.
- Do not build Google Drive file transfer flows for Sheets table semantics.
- Do not assume the current user's local authorization method, such as OAuth token caches, browser sessions, gcloud accounts, clasp login, or third-party tool sessions, will be available to other users.
- Document required capabilities, OAuth scopes, credential expectations, dry-run behavior, and verification steps separately from local credential setup.
- Treat `data/photo-schema.json` as the machine-readable source for photo, album, and import batch field order, basic field metadata, reviewed completeness rules, and approved-use requirements.
- Do not duplicate reviewed/approved field lists in docs. Reference `data/photo-schema.json` instead.
- Do not treat `fixtures/photos.csv` as production data. It exists for 1.0 demos, local UI development, validation fixtures, and future export-format tests.
- Do not treat `fixtures/albums.csv` as production data. It exists for 1.0 demos, debugging, validation fixtures, and future export-format tests.
- Treat `tmp/sheets-export/*.csv` as local work cache exported from the formal Google Sheets photo index. Do not commit it.
- The public GitHub Pages frontend is read-only. It should read Google Sheets public output data and must not contain secrets or database-write credentials.
- `photos` is the public photo index. Public CSV/JSON exports are transport formats with the same fields, not an additional filtered table.
- Album intake should start from the SITCON Flickr album catalog discovered by tools. Users should choose which discovered album to process instead of manually supplying album URLs as the primary workflow.
- GitHub Pages should be deployed through a GitHub Actions artifact, not by publishing the whole repository root.
- Apps Script should be deployed through `clasp`. Keep Apps Script source in the repo, but do not commit personal clasp credentials, Google API credentials, or tokens. `config/project.json` may record fixed Sheet-bound Apps Script IDs because they are identifiers for rebuilding local binding, not credentials. Apps Script push/status/open/deployments should resolve their target from repo config; production is the default target, and practice must be explicit.
- Treat `data/sponsorship-items.json` as a fixed snapshot. SITCON 2026 CFS has ended, so do not build auto-sync behavior for that data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sitcon-tw/flickr-photo-finder](https://github.com/sitcon-tw/flickr-photo-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
