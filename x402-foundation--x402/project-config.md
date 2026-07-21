---
trigger: always_on
description: - Mintlify documentation source for docs.x402.org
---

# Documentation Agent Instructions

## Package Identity
- Mintlify documentation source for docs.x402.org
- MDX/Markdown files with `docs.json` as navigation configuration

## Directory Structure
- `advanced-concepts/` — Advanced topics (lifecycle hooks, etc.)
- `core-concepts/` — Protocol explanations (HTTP 402, client-server, facilitator, wallet)
- `schemes/` — Payment scheme documentation with end-user server/client setup
- `extensions/` — x402 extension documentation (Bazaar discovery, and future extensions)
- `getting-started/` — Quickstart guides for buyers and sellers (MDX files with tabs)
- `guides/` — How-to guides (MCP server, v1→v2 migration)
- `dev-tools/` — Curated third-party SDKs, extensions, and facilitators
- `introduction.md` — Welcome/landing page
- `faq.md` — Frequently asked questions
- `docs.json` — Mintlify navigation and configuration
- `sdk-features.md` - Feature list of SDKs

## Code-to-Doc Mapping
- Changes to `typescript/packages/core/src/` affect Core Concepts docs
- Changes to `typescript/packages/mechanisms/`, `go/mechanisms/`, or `python/x402/mechanisms/` affect Payment Schemes docs and `sdk-features.md`
- Changes to `typescript/packages/extensions/src/` affect Extensions docs (Bazaar, etc.)
- Changes to `typescript/packages/*/src/` affect SDK references and quickstart guides
- Changes to `python/x402/` affect Python SDK references
- Changes to `python/x402/extensions/` affect Extensions docs
- Changes to `go/` affect Go SDK references
- Changes to `go/extensions/` affect Extensions docs
- Changes to facilitator endpoints affect quickstart guides
- Changes to `specs/schemes/` may require updates to Payment Schemes docs
- Changes to other `specs/` files may require updates to Core Concepts docs
- Curated third-party SDK, extension, or facilitator listings → update `dev-tools/*.md`

## Style Guidelines
- Use TypeScript for primary code examples (it's the reference SDK)
- For scheme pages, lead with server setup before client setup
- Keep scheme overviews network-agnostic; put network-specific behavior in the relevant scheme page section and link to specs for protocol details
- Include error handling in all API examples
- Write for developers with 2-5 years experience
- Use MDX components (`<Tabs>`, `<Tab>`, `<Callout>`, `<Card>`) for interactive content
- Show both success and error response examples for API endpoints
- Use real-world parameter values in examples (not foo/bar placeholders)

## Conventions
- DO: Add new pages to `docs.json` navigation
- DO: Include code examples from real SDK files (not made-up snippets)
- DO: Link to relevant specs in `specs/` for protocol details
- DO: Use `<Tabs>` for multi-language code examples
- DO: Add frontmatter (title, description) to all pages
- DO: Link scheme pages to relevant server/client examples
- DON'T: Duplicate protocol details from `specs/` — link instead
- DON'T: Add pages without updating `docs.json`
- **Git: Create PRs for review; NEVER commit directly to main**

## Touch Points / Key Files
- `introduction.md` — Landing page
- `faq.md` — Frequently asked questions
- `docs.json` — Navigation and configuration (MUST update when adding pages)
- `core-concepts/*.md` — Conceptual documentation
- `schemes/*.mdx` — Payment scheme docs. Keep `overview` network-agnostic; individual scheme pages may include network-specific SDK setup and spec links
- `extensions/*.mdx` — Extension documentation (Bazaar, future extensions)
- `advanced-concepts/*.mdx` — Advanced topics (lifecycle hooks for custom workflows, etc.)
- `getting-started/*.mdx` — Quickstart guides (MDX for tab components)
- `guides/*.md` — How-to guides
- `dev-tools/*.md` — Curated third-party SDKs, extensions, and facilitators
- `sdk-features.md` — Update when SDK capabilities change

## File Extensions
- Use `.md` for standard markdown pages
- Use `.mdx` for pages with React components (Tabs, Cards, etc.)

## Common Gotchas
- `docs.json` controls Mintlify navigation; pages not listed won't appear
- Images/diagrams go in project root `static/` directory
- Code examples should reference actual SDK file paths
- Links between pages should omit file extensions (e.g., `../core-concepts/http-402` not `../core-concepts/http-402.md`)

## Pre-PR Checks
- All links work (no broken references)
- New pages added to `docs.json` navigation
- Code examples are from actual SDK files and compile
- Frontmatter present on all pages (title, description)
- MDX syntax is valid (run `mint dev` to verify)

## Agent Behavior Rules (Automated Workflows)

When triggered by GitHub Actions or other automated workflows:

### DO
- ONLY update documentation directly related to the specific code changes
- Focus on the files and commits mentioned in the trigger
- Update SDK references if API signatures change
- Update quickstart guides if SDK usage patterns change
- Update core-concepts if protocol behavior changes

### DO NOT
- Perform general documentation audits or sync operations
- Add documentation for ecosystem partners not mentioned in the code change
- Add documentation for features unrelated to the trigger
- Create PRs for trivial changes (comment removal, formatting, etc.)
- Sync ecosystem partner data from `typescript/site/app/ecosystem/` unless explicitly changed


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [x402-foundation/x402](https://github.com/x402-foundation/x402) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
