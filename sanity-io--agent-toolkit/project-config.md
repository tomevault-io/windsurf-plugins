---
trigger: always_on
description: This is a Sanity-powered project. Use the Knowledge Router below to find Sanity guidance for your task. Available as a [Claude Code and Cursor plugin](https://github.com/sanity-io/agent-toolkit#option-3-install-plugin).
---

# Sanity Project

This is a Sanity-powered project. Use the Knowledge Router below to find Sanity guidance for your task. Available as a [Claude Code and Cursor plugin](https://github.com/sanity-io/agent-toolkit#option-3-install-plugin).

## Commands

```bash
# MCP Setup
npx sanity@latest mcp configure  # Configure MCP for your AI editor

# Schema & Types
npx sanity schema deploy     # Deploy schema to Content Lake (REQUIRED before MCP!)
npx sanity schema extract    # Extract schema for TypeGen
npx sanity typegen generate  # Generate TypeScript types

# Development
npx sanity dev               # Start Studio dev server
npx sanity build             # Build Studio for production
npx sanity deploy            # Deploy Studio to Sanity hosting

# Help
npx sanity docs search "query"  # Search Sanity documentation
npx sanity --help               # List all CLI commands
```

## Knowledge Router

If the Sanity MCP server (`https://mcp.sanity.io`) is available, use `list_sanity_rules` and `get_sanity_rules` to load always up-to-date rules on demand. Otherwise, use the table below to find local reference files.

> For the full reference index, see `skills/sanity-best-practices/SKILL.md`.

| Topic | Trigger Keywords | Reference |
| :--- | :--- | :--- |
| **Onboarding** | `start`, `setup`, `init`, `new project` | `skills/sanity-best-practices/references/get-started.md` |
| **Project Structure** | `structure`, `monorepo`, `embedded studio`, `file naming` | `skills/sanity-best-practices/references/project-structure.md` |
| **Schema** | `schema`, `model`, `document`, `field`, `defineType` | `skills/sanity-best-practices/references/schema.md` |
| **Deprecation** | `deprecate`, `remove field`, `legacy`, `migration` | `skills/sanity-best-practices/references/schema.md` |
| **Import/Migration** | `import`, `wordpress`, `html`, `markdown`, `migrate` | `skills/sanity-best-practices/references/migration.md` |
| **Next.js** | `next.js`, `app router`, `server component`, `fetch` | `skills/sanity-best-practices/references/nextjs.md` |
| **Nuxt** | `nuxt`, `vue`, `nuxt.js` | `skills/sanity-best-practices/references/nuxt.md` |
| **Angular** | `angular`, `signals`, `resource api` | `skills/sanity-best-practices/references/angular.md` |
| **Astro** | `astro`, `islands` | `skills/sanity-best-practices/references/astro.md` |
| **Remix/React Router** | `remix`, `react router`, `loader` | `skills/sanity-best-practices/references/remix.md` |
| **Svelte** | `svelte`, `sveltekit`, `kit` | `skills/sanity-best-practices/references/svelte.md` |
| **Visual Editing** | `stega`, `visual editing`, `clean`, `overlay`, `presentation`, `usePresentationQuery` | `skills/sanity-best-practices/references/visual-editing.md` |
| **Page Builder** | `page builder`, `pageBuilder`, `block component`, `alignment`, `switch render` | `skills/sanity-best-practices/references/page-builder.md` |
| **Rich Text** | `portable text`, `rich text`, `block content`, `serializer`, `PTE`, `marks`, `annotations` | `skills/sanity-best-practices/references/portable-text.md` |
| **Images** | `image`, `urlFor`, `crop`, `hotspot`, `lqip` | `skills/sanity-best-practices/references/image.md` |
| **Studio Structure** | `structure`, `desk`, `sidebar`, `singleton`, `grouping` | `skills/sanity-best-practices/references/studio-structure.md` |
| **Localization** | `i18n`, `translation`, `localization`, `language`, `multilingual`, `localized singleton` | `skills/sanity-best-practices/references/localization.md` |
| **SEO** | `seo`, `metadata`, `sitemap`, `og image`, `open graph`, `json-ld`, `redirect` | `skills/sanity-best-practices/references/seo.md` |
| **Shopify/Hydrogen** | `shopify`, `hydrogen`, `e-commerce`, `storefront`, `sanity connect` | `skills/sanity-best-practices/references/hydrogen.md` |
| **GROQ** | `groq`, `query`, `defineQuery`, `projection`, `filter`, `order` | `skills/sanity-best-practices/references/groq.md` |
| **TypeGen** | `typegen`, `typescript`, `types`, `infer`, `satisfies`, `type generation` | `skills/sanity-best-practices/references/typegen.md` |
| **App SDK** | `app sdk`, `custom app`, `useDocuments`, `useDocument`, `DocumentHandle`, `SanityApp`, `sdk-react` | `skills/sanity-best-practices/references/app-sdk.md` |
| **Blueprints** | `blueprints`, `IaC`, `infrastructure`, `stack`, `defineBlueprint` | `skills/sanity-best-practices/references/blueprints.md` |
| **Sanity Functions** | `functions`, `serverless`, `event handler`, `documentEventHandler`, `defineDocumentFunction`, `sanity.blueprint.ts` | `skills/sanity-best-practices/references/functions.md` |

### Using the Knowledge Router

**Before modifying any code:**
1. Identify which topics from the table above apply to your task
2. Read the corresponding reference file(s) using the file path
3. Follow the patterns and constraints defined in those references

Example: If asked to "create a blog post schema", read `skills/sanity-best-practices/references/schema.md` first.

## Agent Behavior

- Specialize in **Structured Content**, **GROQ**, and **Sanity Studio** configuration.
- Write best-practice, type-safe code using **Sanity TypeGen**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanity-io/agent-toolkit](https://github.com/sanity-io/agent-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
