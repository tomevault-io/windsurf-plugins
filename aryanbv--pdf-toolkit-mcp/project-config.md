---
trigger: always_on
description: **What**: A write-capable, zero-config, TypeScript-native MCP server for PDF manipulation. Provides 22 tools for reading, modifying, creating, and rendering PDFs — including page-to-image rendering for vision-capable clients, high-fidelity Markdown→PDF, AES-256 encryption, and form-preserving merge/split — usable from any MCP client (Claude Code, Claude Desktop, etc.) with zero installation beyond `npx`. Ships 5 guided-workflow prompts and a templates resource.
---

# pdf-toolkit-mcp

## 1. Project Overview

**What**: A write-capable, zero-config, TypeScript-native MCP server for PDF manipulation. Provides 22 tools for reading, modifying, creating, and rendering PDFs — including page-to-image rendering for vision-capable clients, high-fidelity Markdown→PDF, AES-256 encryption, and form-preserving merge/split — usable from any MCP client (Claude Code, Claude Desktop, etc.) with zero installation beyond `npx`. Ships 5 guided-workflow prompts and a templates resource.

**Why**: Learning MCP server development, building open-source credibility, and laying groundwork for Prevyl.

- **Package**: `@aryanbv/pdf-toolkit-mcp`
- **Install**: `npx -y @aryanbv/pdf-toolkit-mcp`
- **License**: MIT
- **Repo**: github.com/AryanBV/pdf-toolkit-mcp

## 2. Tech Stack

Multi-engine architecture: **@pdfme/pdf-lib** manipulates existing PDFs + **@react-pdf/renderer** driven by **remark/mdast** handles rich creation (Markdown, templates) + **unpdf** (pdf.js) reads text/metadata and positional text + **@hyzyla/pdfium** (WASM) renders pages to images + **@neslinesli93/qpdf-wasm** (WASM) does AES-256 encryption.

| Dependency                                | Version                          | Why                                                                                                                                         |
| ----------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| TypeScript                                | strict, ESM (`"type": "module"`) | Type safety, all `.ts` imports use `.js` extension (Node16 resolution)                                                                      |
| Node.js                                   | >= 20                            | Conservative non-breaking floor (18 and 20.x are EOL); CI tests 20/22/24, 22 or 24 LTS recommended                                          |
| `@modelcontextprotocol/sdk`               | ^1.29.0                          | MCP protocol implementation. Use `server.registerTool()` — NOT `.tool()` (deprecated)                                                       |
| `@pdfme/pdf-lib`                          | ^5.5.10                          | Fork of abandoned `pdf-lib`. Existing PDF manipulation (merge, split, rotate, watermark, embed, page numbers, reorder, delete, flatten, QR) |
| `unpdf`                                   | ^1.4.0                           | Text extraction, metadata, and positional text (pdf.js). Replaces `pdfjs-dist` which requires Node 22+ canvas bindings                      |
| `@react-pdf/renderer`                     | ^4.5.1                           | High-fidelity PDF creation for Markdown and templates (Helvetica default; custom TTF via `Font.register`). Lazy-loaded inside handlers      |
| `react`                                   | ^19.2.6                          | Peer of `@react-pdf/renderer`; document trees are built with `React.createElement`                                                          |
| `unified` + `remark-parse` + `remark-gfm` | ^11 / ^11 / ^4                   | Markdown → mdast AST (CommonMark + GFM tables/strikethrough) consumed by the react-pdf renderer                                             |
| `mdast-util-to-markdown`                  | ^2.1.2                           | Serializes reconstructed layout nodes back to Markdown for `pdf_to_markdown`                                                                |
| `@hyzyla/pdfium`                          | ^2.1.13                          | PDFium compiled to WASM — renders PDF pages to raster bitmaps for `pdf_render_pages`. Zero native deps. Lazy-loaded                         |
| `@neslinesli93/qpdf-wasm`                 | ^0.3.0                           | qpdf compiled to WASM — AES-256 encryption for `pdf_encrypt`. Zero native deps. Lazy-loaded                                                 |
| `pngjs` + `jpeg-js`                       | ^7.0.0 / ^0.4.4                  | Encode PDFium raster bitmaps to PNG / JPEG buffers                                                                                          |
| `diff`                                    | ^9.0.0                           | Line-level text diff for `pdf_compare`                                                                                                      |
| `@bwip-js/node`                           | ^4.8.0                           | Barcode/QR code generation as PNG buffers. ESM — `import { toBuffer } from '@bwip-js/node'`                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AryanBV/pdf-toolkit-mcp](https://github.com/AryanBV/pdf-toolkit-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
