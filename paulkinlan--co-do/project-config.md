---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project Overview

Co-do is a Cowork-like browser experience using the File System Access API for collaborative coding with native filesystem integration.

> **IMPORTANT: Always run `npm test` after code changes.** Tests must pass before committing.
> **Exception:** Not required for documentation-only changes (`.md` files, comments).

> **IMPORTANT: Never modify `version` in `package.json`.** Version bumping is automatic via GitHub Action on PR merge.

## Browser Support

**Target: Latest Chrome (Chrome 140+)** — File System Access API is the core dependency.

- Use modern web platform APIs freely; no polyfills or legacy support needed
- Chrome-specific APIs acceptable; focus on Baseline Newly/Widely Available features
- See `.claude/skills/modern-web-dev/SKILL.md` for detailed guidance

## Development Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install dependencies |
| `npm run dev` | Vite dev server on http://localhost:3000 |
| `npm run build` | Full production build (WASM + TS + Vite → `dist/`) |
| `npm run build:web-only` | Build without WASM compilation |
| `npm run type-check` | TypeScript type checking only |
| `npm run preview` | Preview production build |
| `npm test` | All Playwright tests (visual + accessibility) |
| `npm run test:visual` | Visual regression tests only |
| `npm run test:accessibility` | Accessibility tests only (WCAG 2.1 AA) |
| `npm run test:visual:update` | Update baseline screenshots |
| `npm run test:unit` | Vitest unit tests |
| `npm run test:unit:watch` | Unit tests in watch mode |
| `npm run test:ui` | Playwright interactive UI mode |
| `npm run test:debug` | Debug mode with Playwright Inspector |
| `npm run test:report` | Open HTML test report |
| `npm run wasm:build` | Build all WASM tools (requires wasi-sdk) |
| `npm run wasm:build:native` | Build native-only WASM tools |

## Project Structure

```
Co-do/
├── src/
│   ├── main.ts               # Entry point, PWA init, version checking
│   ├── ui.ts                  # UI manager, event handlers, conversations, permissions UI
│   ├── ai.ts                  # AI SDK integration (streaming, multi-provider, dynamic import)
│   ├── tools.ts               # 20 file operation tools + pipe command
│   ├── pipeable.ts            # Self-registering pipeable command registry
│   ├── fileSystem.ts          # File System Access API wrapper with caching
│   ├── preferences.ts         # Tool permissions (ToolName type, permission levels)
│   ├── storage.ts             # IndexedDB manager (configs, conversations, workspaces, WASM tools)
│   ├── router.ts              # Hash-based workspace URL routing
│   ├── diff.ts                # Unified diff generation (LCS algorithm)
│   ├── markdown.ts            # Markdown rendering in sandboxed iframes (XSS protection)
│   ├── toasts.ts              # Toast notification system
│   ├── notifications.ts       # Native browser notifications
│   ├── viewTransitions.ts     # View Transitions API integration
│   ├── provider-registry.ts   # Provider cookie management + CSP coordination
│   ├── network-monitor.ts     # CSP violation monitoring, network request logger, visual firewall
│   ├── tool-response-format.ts # Pure functions for tool response formatting
│   ├── toolResultCache.ts     # Caching large tool outputs (>2KB → summary to AI, full to UI)
│   ├── styles.css             # CSS with custom properties for dark mode
│   └── wasm-tools/            # WebAssembly custom tools system
│       ├── manager.ts         # Central tool orchestrator
│       ├── runtime.ts         # WASI runtime (main-thread fallback)
│       ├── wasm-worker.ts     # Worker-based WASM runtime (default, sandboxed)
│       ├── worker-manager.ts  # Worker lifecycle and pooling
│       ├── vfs.ts             # Virtual file system (WASI syscall interception)
│       ├── loader.ts          # ZIP package loader and manifest validator
│       ├── registry.ts        # Built-in tool config (41 tools by category)
│       ├── types.ts           # TypeScript interfaces and Zod schemas
│       ├── worker-types.ts    # Worker message protocol types
│       ├── index.ts           # Public API exports
│       └── adapters/          # Non-WASI tool adapters (FFmpeg)
├── server/
│   ├── main.ts                # Vite server plugins entry
│   ├── providers.ts           # Provider registry and cookie parsing
│   └── csp.ts                 # Dynamic CSP header generation per provider
├── tests/
│   ├── visual/                # Visual regression tests (screenshots)
│   ├── accessibility/         # WCAG 2.1 Level AA compliance tests
│   ├── unit/                  # Unit tests (Vitest)
│   └── helpers/               # Test utilities (test-utils.ts, dom-inspector.ts)
├── docs/                      # Architecture docs (WASM, CSP, security)
├── public/                    # PWA manifest, service worker, icons
├── wasm-tools/                # C/WASI source, manifests, build scripts
├── index.html                 # HTML entry point (UI structure, modals, permissions)
├── vite.config.ts             # Vite config with dynamic CSP + version plugin
├── playwright.config.ts       # Playwright test configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulKinlan/Co-do](https://github.com/PaulKinlan/Co-do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
