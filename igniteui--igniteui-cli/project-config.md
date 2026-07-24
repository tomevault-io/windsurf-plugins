---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Ignite UI Documentation MCP Server** — a Model Context Protocol server that provides AI assistants with tools to search and retrieve Ignite UI component documentation. The server loads compressed markdown docs at startup and exposes them via MCP tools such as `list_components`, `get_doc`, and `search_docs`. It includes documentation processing pipelines for Angular, React, WebComponents, and Blazor.

## Project Structure

```
├── src/index.ts                        # MCP server entry point — dual-mode (remote or local SQLite)
├── src/providers/
│   ├── DocsProvider.ts                 # DocsProvider interface (listComponents, getDoc, searchDocs)
│   ├── RemoteDocsProvider.ts           # Remote mode — proxies to URL given via --remote
│   └── LocalDocsProvider.ts            # Local mode — sql.js WASM SQLite with FTS4
├── scripts/
│   ├── build-db.ts                    # Build SQLite DB from compressed docs (better-sqlite3)
│   ├── export-angular-docs.ts          # Export Angular docs from docfx (toc-driven, template expansion, include resolution, API URL resolution)
│   ├── inject-angular-docs.ts          # Inject sample code into docs (replaces <code-view> with component source)
│   ├── compress-angular-docs.ts        # LLM-based compression of docs (~50% size reduction, supports --batch mode)
│   ├── export-react-docs.ts            # Export React docs from xplat gulp build (toc.json-driven, flatten hierarchy)
│   ├── inject-react-docs.ts            # Inject React sample code (github-src based resolution, TSX files)
│   ├── compress-react-docs.ts          # LLM-based compression with React-specific prompt (Igr prefix, supports --batch mode)
│   ├── export-wc-docs.ts              # Export WebComponents docs from xplat gulp build (toc.json-driven, flatten hierarchy)
│   ├── inject-wc-docs.ts              # Inject WC sample code (github-src based, HTML/TS/CSS files)
│   ├── compress-wc-docs.ts            # LLM-based compression with WC-specific prompt (Igc prefix + Component suffix, supports --batch mode)
│   ├── export-blazor-docs.ts          # Export Blazor docs from xplat gulp build (toc.json-driven, flatten hierarchy)
│   ├── inject-blazor-docs.ts          # Inject Blazor sample code (github-src based, .razor/.razor.cs/.css files)
│   ├── compress-blazor-docs.ts        # LLM-based compression with Blazor-specific prompt (Igb prefix, no suffix, supports --batch mode)
│   ├── validate-docs.ts               # LLM-as-Judge validation of compressed docs (platform-independent)
│   ├── export-angular-api.ts          # Build Angular API docs from blazor/api-docs submodule → docs/angular-api/
│   ├── export-react-api.ts            # Build React API docs from blazor/api-docs submodule → docs/react-api/
│   ├── export-wc-api.ts               # Build Web Components API docs from blazor/api-docs submodule → docs/webcomponents-api/
│   └── export-blazor-api.ts           # Build Blazor API docs from blazor/api-docs submodule → docs/blazor-api/
├── docs/
│   ├── knowledgebase.md                # Lessons learned and issues for cross-platform reference (32 entries)
│   ├── db.md                           # SQLite + FTS4 database integration (IMPLEMENTED)
│   ├── batch-compression.md            # OpenAI Batch API for compression (IMPLEMENTED)
│   ├── incremental-processing.md       # Plan: Incremental processing with diff-based pipeline (NOT YET IMPLEMENTED)
│   ├── progress.md                     # Implementation progress tracker
│   ├── impl_plan.md                    # Original implementation plan for code-view replacement
│   ├── prefix_fix.md                   # Plan for fixing component prefix issue
│   ├── toc_based_processing.md         # Plan for toc.yml-driven file selection
│   ├── xplat-docs-architecture.md      # Cross-platform docs architecture analysis (variable replacement, toc.json, apiMap)
│   ├── react-pipeline.md              # React pipeline implementation plan
│   ├── wc-pipeline-plan.md            # WebComponents pipeline implementation plan
│   └── blazor-pipeline-plan.md        # Blazor pipeline implementation plan
├── angular-api/                        # Angular API docs (llms-full.txt files, built by build:docs:angular-api)
├── react-api/                          # React API docs (llms-full.txt files, built by build:docs:react-api)
├── webcomponents-api/                  # Web Components API docs (llms-full.txt files, built by build:docs:wc-api)
├── blazor-api/                         # Blazor API docs (llms-full.txt files, built by build:docs:blazor-api)
├── angular/                            # Git submodules
│   ├── igniteui-docfx/                 # DocFX-based Angular documentation (en/jp/kr)
│   │   └── en/components/toc.yml       # Table of contents — source of truth for which files to process
│   ├── igniteui-angular-samples/       # Angular sample apps for Ignite UI components
│   └── igniteui-angular-examples/      # Angular example projects
├── react/
│   └── igniteui-react-examples/        # React example projects (git submodule)
├── webcomponents/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IgniteUI/igniteui-cli](https://github.com/IgniteUI/igniteui-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
