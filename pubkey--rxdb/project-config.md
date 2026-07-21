---
trigger: always_on
description: - **Database**: RxDB (local-first, NoSQL)
---

# AGENTS.md

## Project Overview
- **Database**: RxDB (local-first, NoSQL)
- **Language**: TypeScript
- **State Management**: Reactive (RxJS Observables)
- **Paths**: Source code in `src/`, tests in `test/`, documentation in `docs-src/`.

## Tooling
- **Build All**: `npm run build`
- **Documentation Build**: `npm run docs:build`
- **Run All Tests**: `npm run test`
- **Fast Tests (Parallel)**: `npm run test:fast`
- **Fast Memory Tests**: `npm run test:fast:memory`
- **Node Tests**: `npm run test:node`
- **Browser Tests**: `npm run test:browser`
- **Performance Tests**: `npm run test:performance`
- **Lint**: `npm run lint`
- **Lint Fix**: `npm run lint:fix`
- **Check Types**: `npm run check-types`
- **Unwatch Tests**: `npm run dev`

## Code Style & Patterns
- **Language**: TypeScript
- **Formatting**: Uses ESLint. Run `npm run lint` to check and `npm run lint:fix` to auto-fix.
- **Imports**: Uses ES modules (import/export).
- **TypeScript**: Do not use enums. Prefer types instead of interfaces.
- **Errors**: Do not use `throw new Error()`. Use `throw new RxError()` instead to reduce build size and do not include full error messages in production builds. Use the error codes from `src/rx-error.ts` and add new error codes if needed like `PL1`, `PL2`. Example: `throw newRxError('PL1', { plugin });`

## Documentation Style
- SHOULD use clear, simple language.
- SHOULD use data and examples to support claims when possible.
- SHOULD be informative.
- SHOULD focus on practical, actionable insights.
- AVOID using em dashes (–) anywhere.
- AVOID constructions like "not just this, but also this".
- AVOID metaphors and cliches.
- AVOID generalizations.
- AVOID upfront warnings or notes, just the output requested.
- AVOID rhetorical questions.
- AVOID specific words like: very, really, literally, actually, certainly, probably, basically, delve, embark, enlightening, esteemed, shed light, craft, creative, imagine, realm, game-changer, unlock, discover, skyrocket, abyss, not alone, in a world where, revolutionize, disruptive, utilize, utilizing, dive deep, tapestry, illuminate, unveil, pivotal, intricate, elucidate, hence, furthermore, realm, however, harness, exciting, groundbreaking, cutting-edge, remarkable, it remains to be seen, glimpse into, navigating, landscape, stark, testament, in summary, in conclusion, moreover, boost, skyrocket, opened up, powerful, inquiries, ever-evolving.
- Review your response and ensure no em dashes.
- MUST format FAQ sections using HTML `<details>` and `<summary>` tags. Ensure there is an empty line before and after the inner markdown content so it parses correctly.
- SHOULD try to use components from the `docs-src/src/components` folder when writing docs.

## Documentation Writing Style Guide

This guide is derived from an analysis of all existing pages in `docs-src/docs/`. Older pages (2023-era) contain hype vocabulary that is now banned; when patterns conflict, follow this guide and the rules above, not legacy pages. Good style models: `articles/realm-to-rxdb-migration.md`, `webmcp.md`, `testing.md`, `rx-storage-localstorage.md`, the newer `articles/alternatives/*.md` pages.

### Frontmatter
- Exactly four fields, always in this order: `title`, `slug`, `description`, `image`. No other fields.
- `slug`: kebab-case filename plus `.html`, for example `slug: partial-sync.html`.
- `image`: `/headers/<slug-basename>.jpg`. Alternative articles use `/headers/alternatives/<slug-basename>.jpg`.
- `title`: Title Case, 40 to 80 chars, keyword first. Use a plain hyphen `-` as separator, never an em dash or `|`. Two modes: plain feature name for reference pages ("Key Compression", "RxQuery") or keyword phrase for articles ("RxDB as a Dexie.js Alternative with Mango Queries and Replication"). The H1 may differ slightly from the title.
- `description`: 1 to 2 sentences, about 120 to 160 chars, contains the primary keyword. Openers like "Compare X with RxDB." or "Learn how ...". Do not use the banned words even though older descriptions contain them.

### Page structure
- MDX component imports go between the frontmatter and the H1.
- One H1 per page. Integration and feature landing pages may use `<HeadlineWithIcon h1 icon={...}>` with an optional `subtitle`.
- Opening paragraph: define the topic in 1 to 4 sentences, bold the primary keyword on first mention, link `[RxDB](https://rxdb.info/)` on first mention in articles, and include 2 to 6 internal links. Articles add a roadmap sentence: "This page explains what X is, where it falls short, and how RxDB ...".
- Place `<RxdbLogo alt="<keyword phrase>" />` after the intro paragraph in articles. It is globally registered, no import needed.
- Article flow: What is X → why X matters or its limits → What RxDB adds (numbered `### 1. ...` subsections) → code samples → FAQ → `## Follow Up` link list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pubkey/rxdb](https://github.com/pubkey/rxdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
