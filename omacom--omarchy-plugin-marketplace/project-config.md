---
trigger: always_on
description: This file defines the working rules for the entire repository. Use it as the default context for future changes. Follow a direct user instruction when it conflicts with this file. Communicate with the maintainer in German unless they request another language. Keep source code, interface copy, and public project documentation in English.
---

# Repository guidelines

This file defines the working rules for the entire repository. Use it as the default context for future changes. Follow a direct user instruction when it conflicts with this file. Communicate with the maintainer in German unless they request another language. Keep source code, interface copy, and public project documentation in English.

## Product intent

Omarchy Plugins is an independent community marketplace at `omarchyplugins.com`. It helps people discover, inspect, and install plugins for Omarchy. It is not affiliated with Omarchy or 37signals.

Preserve these product qualities:

- Minimal, precise, technical, and aesthetically restrained
- Command-first, with source and trust information visible
- Fast, static, accessible, and usable without an application server
- Curated, while stating clearly that listing is not a security review

Do not introduce accounts, a database, a backend, a frontend framework, or a new dependency unless the maintainer explicitly approves that architectural change. The approved exception is the credential-free engagement feature under `worker/`: a narrowly scoped Cloudflare Worker and D1 database may store anonymous aggregate plugin detail views, successful command-copy actions, and hearts guarded by local browser storage. Hearts are anonymous reactions, not unique or verified votes. Do not expand it into identity, profiling, comments, scored ratings, installation telemetry, or general analytics without separate approval.

## Project structure and sources of truth

- `site/index.html` contains the marketplace and catalog interface
- `site/explore.html` contains the community graph and catalog growth interface
- `site/plugin.html` contains the plugin detail shell
- `site/publish.html` contains the publishing guide
- `site/assets/css/style.css` is the shared visual system
- `site/assets/css/explore.css` contains the Explore page layout and visualization styles
- `site/assets/js/shared.js` contains shared browser behavior
- `site/assets/js/engagement.js` contains the credential-free engagement API client
- `site/assets/js/search.js` contains shared catalog matching semantics
- `site/assets/js/app.js`, `plugin.js`, and `publish.js` contain page-specific behavior
- `site/assets/js/explore.js`, `explore-search.js`, and `growth-range.js` contain Explore rendering, matching integration, and date-range behavior
- `registry.json` is the curated registry and the source of marketplace metadata
- Upstream plugin `manifest.json` files are the source of plugin-owned metadata
- `scripts/build-catalog.mjs` combines registry and upstream data
- `scripts/build-explorer-data.mjs` derives graph, community, and daily growth data from the generated catalog and its Git history
- `site/catalog.json` and `site/assets/img/plugins/` are generated build outputs
- `site/explorer-data.json` is the generated Explore data output
- `test/explorer.test.js` covers Explore data integrity, search behavior, UI structure, and growth ranges
- `sharp` is the build-only image dependency; source previews are normalized into card and detail WebP variants
- `package.json` defines the Node.js engine, project commands, and direct development dependencies
- `package-lock.json` pins the complete transitive npm dependency graph and integrity hashes used by `npm ci`
- `SECURITY.md` is the unified security policy: it defines private vulnerability reporting, public reporting boundaries, high-level scope, and the deterministic scanner policy, limits, outcomes, enforcement, exact-SHA rules, and contributor requirements
- `VERIFICATION.md` defines the public meaning, request flow, publication safeguards, and display contract for `Verified` and `Unverified`
- `SUBMISSION.md` defines the public command-line and AI-assisted submission contract
- `worker/src/index.js`, `worker/migrations/`, and `worker/wrangler.example.jsonc` define the approved engagement service, D1 schema, and credential-free deployment template
- `PLAN.md` is the living implementation and security roadmap; current code, tests, workflows, and focused policy documents remain authoritative for implemented behavior

Do not manually edit generated catalog data or preview assets. Change their source or build logic, then regenerate them. Do not include unrelated catalog drift in a UI-only change.

Use `npm ci` for reproducible installs and CI. Do not hand-edit `package-lock.json`. When an explicitly approved dependency or other lockfile-represented field changes, update `package.json` and regenerate and review `package-lock.json` together. Script or metadata-only `package.json` changes may legitimately leave the lockfile unchanged. Do not commit a dependency change without its corresponding lockfile update, or a lockfile change that cannot be explained by the manifest.

## Change workflow

1. Run `git status --short --branch` before editing
2. Read the relevant source, tests, and nearby patterns
3. Make the smallest coherent change that solves the request

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omacom/omarchy-plugin-marketplace](https://github.com/omacom/omarchy-plugin-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
