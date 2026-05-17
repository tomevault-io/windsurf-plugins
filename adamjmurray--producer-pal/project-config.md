---
trigger: always_on
description: Producer Pal is an AI music composition tool that integrates with Ableton Live
---

## Project Overview

Producer Pal is an AI music composition tool that integrates with Ableton Live
through a Max for Live device using the Model Context Protocol (MCP). The
codebase is written entirely in TypeScript.

## Essential Commands

```bash
# Build with all tools (use this for development/testing!)
npm run build:debug

# Code quality checks
npm run fix   # Auto-fix formatting and linting issues
npm run check # Run all checks: lint + typecheck + format check + tests
npm run lint
npm run typecheck
npm run format
npm test

# Chat UI development
npm run ui:build # Production build

# Documentation site (VitePress at https://producer-pal.org)
npm run docs:dev     # Development server with hot reload
npm run docs:build   # Build static site
npm run docs:preview # Preview production build
# When editing docs, use clean URLs: /chat-ui not /chat-ui.html (no trailing slash)
# Page files named after folder: docs/guide.md not docs/guide/index.md (except top-level docs/index.md)
```

## Architecture

Portal script → Max for Live Device (MCP Server) → Live API

Key entry points:

- MCP Server: `src/mcp-server/mcp-server.ts`
- Max V8 code: `src/live-api-adapter/live-api-adapter.ts`
- Portal: `src/portal/producer-pal-portal.ts`
- Chat UI: `webui/src/main.tsx`
- Claude Desktop extension: `claude-desktop-extension/manifest.template.json`
- Tools: `src/tools/**/*.ts`
- Chat CLI: `evals/chat/index.ts`
- Evaluation scenarios: `evals/scenarios/index.ts`

See `dev/Architecture.md` for detailed system design and `dev/Chat-UI.md` for
web UI architecture.

## Critical Coding Rules

- **License headers**: All source files must have an SPDX license header at the
  top (after any shebang). Format:

  ```typescript
  // Producer Pal
  // Copyright (C) <year> <author>
  // AI assistance: <AI tool> (<company>)
  // SPDX-License-Identifier: GPL-3.0-or-later
  ```

  List all authors who contributed to the file. New files should include the
  current year and the contributor's name. When AI tools modify a file, add the
  AI assistance line (e.g., `// AI assistance: Claude (Anthropic)`). If an AI
  assistance line already exists, append additional tools as a comma-separated
  list (e.g., `// AI assistance: Gemini (Google), Claude (Anthropic)`).

- **File naming**: React components use PascalCase (e.g., `ChatHeader.tsx`). All
  other files use kebab-case (e.g., `use-chat.ts`, `live-api-adapter.ts`)

- **Function organization**: In files that export functions, the first exported
  function should be the main function named after the file (e.g.,
  `updateClip()` in `update-clip.ts`, `readTrack()` in `read-track.ts`). All
  helper functions (both internal and exported) must be placed below the main
  exported function(s). This improves code readability and makes it immediately
  clear what the primary purpose of each file is.

- **Import extensions**: Code in `src/` must include `.ts` file extensions in
  imports matching the actual file type (e.g., `import foo from './bar.ts'`).
  Peggy-generated parsers are wrapped in TypeScript files (e.g.,
  `barbeat-parser.ts`) - import from the wrapper, not the `.js` file. Code in
  `webui/` is bundled and must NEVER use file extensions in relative imports
  (e.g., `import foo from './bar'`).

- **Path aliases**: Use `#src/` for src imports (e.g.,
  `import foo from '#src/shared/utils.ts'`), `#webui/` for webui imports (e.g.,
  `import { App } from '#webui/components/App'`), and `#evals/` for evals
  imports (e.g.,
  `import { runScenario } from '#evals/scenarios/run-scenario.ts'`). All use
  Node.js package subpath imports configured in package.json `"imports"` field.
  The `#` prefix is required by Node.js for unbundled execution (build scripts,
  CLI tools). Never use relative paths like `../../` when a path alias is
  available.

- **No barrel files**: Do not create index.ts or other files that only re-export
  from other modules. Import directly from the source file instead.

- **Testing builds**: Always use `npm run build:debug` for development (includes
  debugging tools like `ppal-raw-live-api`)

- **Exact dependency versions**: All versions in package.json must be exact (no
  `^`, `~`, or ranges). `.npmrc` enforces this for `npm install`. A test in
  `src/test/package-json-versions.test.ts` validates it.

- **Zod limitations**: Use only primitive types and enums in tool input schemas.
  For list-like inputs, use comma-separated strings

- **Tool schema coercion**: Use `z.coerce.string()` instead of `z.string()` for
  ID parameters in tool input schemas (e.g., `ids`, `trackId`, `clipId`,
  `sceneIndex` when it accepts comma-separated values). Use `z.coerce.number()`
  instead of `z.number()` for numeric parameters (e.g., `trackIndex`,
  `sceneIndex`, `count`, `tempo`, `gainDb`). This allows LLMs to pass values as
  either strings or numbers (like `id: 123` or `trackIndex: "3"`) which Zod
  automatically coerces. The MCP SDK validates schemas before our handler runs,
  so coercion must happen at the schema level.

- **Small model mode**: When modifying tool definitions (`.def.ts` files) —
  adding, removing, or renaming parameters, or changing descriptions — check
  whether corresponding changes are needed in the `smallModelModeConfig`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamjmurray/producer-pal](https://github.com/adamjmurray/producer-pal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
