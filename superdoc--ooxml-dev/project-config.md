---
trigger: always_on
description: The OOXML spec, explained by people who actually implemented it.
---

# ooxml.dev

The OOXML spec, explained by people who actually implemented it.

An interactive reference for ECMA-376 (Office Open XML) built by the [SuperDoc](https://superdoc.dev) team. Live previews are rendered with SuperDoc itself — every example on the site is a working document.

## Why This Exists

The official ECMA-376 spec is 5,000+ pages. Most of it you'll never need, and the parts you do need often omit critical rendering details that only surface when you compare against Word's actual behavior. This site fills that gap with implementation notes from building SuperDoc — a document engine that renders OOXML natively in the browser.

This is also how people discover SuperDoc. By sharing what we've learned, we position ourselves as the OOXML experts. Every page should reflect that authority: practical, specific, from-experience.

## Content Philosophy

**Write for implementers, not spec lawyers.** The audience is developers building document tools who need to know what the spec doesn't tell them.

Every doc page should answer:
1. **What does the XML look like?** — Structure tree and live examples
2. **What does Word actually do?** — Rendering behavior, especially where it diverges from the spec
3. **What will trip you up?** — Implementation notes from real experience

Keep notes concise (1-2 sentences). Lead with the insight, not the backstory. Use `app: "Word"` when the behavior is Word-specific.

## Project Structure

```
apps/
  web/                 React app (Vite, React Router, Tailwind)
    src/data/docs.ts   ← All doc pages live here (single source of truth)
    src/components/    UI components (Sidebar, SuperDocPreview, etc.)
    src/pages/         Route pages (Home, Docs, SpecExplorer, Mcp)
  mcp-server/          Cloudflare Worker - MCP server (semantic + structural tools)
packages/
  shared/              Database client, embedding client, types
scripts/
  ingest-pdf/          ECMA PDF -> spec_content (semantic search corpus)
  ingest-xsd/          ECMA XSDs -> schema graph (structural query corpus)
  sources-sync.ts      data/sources.json -> reference_sources
  db-migrate.ts        Apply db/migrations/*.sql in order
db/
  schema.sql           PostgreSQL + pgvector + XSD schema graph
  migrations/          Numbered, idempotent SQL migrations
data/
  sources.json         Source manifest (artifact URLs, sha256, license notes)
  xsd-cache/           Local-only XSD download cache (gitignored)
dev/
  data/                Extracted/chunked/embedded PDF content
```

## Commands

```bash
bun install            # Install dependencies
bun dev                # Web app at http://localhost:5173
bun dev:mcp            # MCP server at http://localhost:8787
bun run build          # Production build (web)
bun run typecheck      # Type-check all packages
```

## Adding a Doc Page

All documentation lives in `apps/web/src/data/docs.ts` as a keyed object. Each page has a `title`, optional `badge` (OOXML element), and `content` array of typed blocks.

### Content block types

| Type | Purpose |
|------|---------|
| `heading` | Section heading (level 2, 3, or 4) |
| `paragraph` | Prose text (supports markdown links) |
| `code` | Code/structure block with optional language |
| `preview` | Live OOXML rendered by SuperDoc (editable XML + preview) |
| `note` | Implementation note (critical / warning / info / tip) with optional `app` |
| `table` | Data table with headers and rows |

### Steps

1. Add an entry to the `docs` object in `apps/web/src/data/docs.ts`
2. Add a sidebar link in `apps/web/src/components/Sidebar.tsx` under the right section
3. The page auto-routes to `/docs/{key}`

### Page structure convention

Follow this order (see existing pages for examples):
1. Intro paragraph — what the element does, one sentence on why it matters
2. **Structure** — element tree showing hierarchy and attributes
3. **Examples** — live `preview` blocks, start simple, build complexity
4. **Implementation Notes** — the real value; what the spec doesn't tell you
5. **Schema** — reference table of elements/attributes

### Writing implementation notes

- **critical** — things that will break your implementation if you get them wrong
- **warning** — non-obvious behavior that affects rendering
- **info** — good to know, won't break things
- **tip** — helpful shortcuts or techniques

Use `app: "Word"` (or `"Word, LibreOffice"`) when the behavior is application-specific. Omit `app` for universal observations.

## SuperDoc Preview Component

The `preview` block type renders XML with SuperDoc loaded from unpkg. It creates a minimal .docx in-memory (via JSZip), passes it to SuperDoc, and shows a split view: editable XML on the left, live rendering on the right.

The XML you provide is wrapped in a minimal `w:document > w:body` structure automatically. Just provide the body content (paragraphs, tables, etc.).

## MCP Server

Cloudflare Worker exposing three tool families over MCP. Prose and schema-lookup tools are backed by the database; package-metadata tool reads a curated static dataset bundled with the worker.

Prose search over the spec PDFs (powered by `spec_content`):

- `ooxml_search` - semantic vector search across 18,000+ spec chunks
- `ooxml_section` - fetch a specific section by ID (e.g., "17.3.1.24")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superdoc/ooxml-dev](https://github.com/superdoc/ooxml-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
