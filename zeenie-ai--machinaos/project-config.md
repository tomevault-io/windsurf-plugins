---
trigger: always_on
description: handles ADB dispatch via `SERVICE_ID_MAP`. See the
---

# MachinaOs - Claude Documentation

## Project Overview
This is a React Flow-based workflow automation platform implementing n8n-inspired architectural patterns. The project has undergone a comprehensive refactoring to implement modern INodeProperties interface system with full TypeScript compliance and code cleanup.

## Documentation Reference

**Always refer to these documentation files for detailed guides:**

| Document | Description |
|----------|-------------|
| **[Frontend Architecture](./docs-internal/frontend_architecture.md)** | Current frontend stack (React 19 + Vite + Tailwind v4 + shadcn/ui + Radix + RHF/zod + TanStack Query + Zustand). Tokens, primitives, forms, credentials exemplar, ownership boundary, `uiHints` catalogue. |
| **[UI Migration Plan](./docs-internal/ui_migration_plan.md)** | antd → shadcn/ui migration plan + completion log. Waves 1–10 done. Full frontend is schema-driven (backend SSOT); remaining DIY widget registry (ex-Phase 6) is the one deferred item. |
| **[Node Allowlist](./docs-internal/node_allowlist.md)** | Single-config UI visibility — `server/config/node_allowlist.json` controls which nodes / credential categories / skill folders show in the UI. Five lists with two enforcement tiers (mode-gated allowlist + absolute blocklist). `useNodeAllowlist` hook exposes `isVisible` / `isBlocked` / `isAllowed` / `isCredentialCategoryDisabled` / `isSkillFolderDisabled`. Adding a new disabled domain = single JSON edit, no code change. |
| **[Theme System](./docs-internal/theme_system.md)** | 10-way visual theme system — 5 utopian (light, dark, renaissance, greek, edo, steampunk, atomic) + 5 dystopian (cyber, wasteland, rot, plague, surveillance) — driven by `<html data-theme>` + per-theme CSS files in `client/src/themes/`. Token taxonomy (surface / fg / border / accent / typography / motion), shadcn HSL-triplet bridge, action role tokens, decorative-layer wrappers (`.app-frame` / `.canvas-host` / `.modal-frame`), per-component decorative ornaments (panel textures + canvas decorations + node pseudo-element overlays + theme-specific keyframes), **canvas-node visual contract via `--node-color` CSS custom property** (no inline `background` / `border` on node components — base.css + per-theme CSS owns visuals; `NodeStyle` helper type at [types/NodeTypes.ts](./client/src/types/NodeTypes.ts) makes the inline custom-prop typecheck-clean), **`--node-pulse-color` separate from `--node-color`** so executing-node glow uses each theme's highest-contrast accent regardless of plugin accent (Cyber neon cyan, Surveillance REC red, Renaissance ultramarine, etc.), **`data-page-hidden` animation pause** (toggled by Dashboard's `visibilitychange` listener; base.css declares `html[data-page-hidden] *, *::before, *::after { animation-play-state: paused !important }` to prevent compositor stall on tab return), **per-theme icon glyph system** (290 SVGs across 29 keys × 10 themes via [themedGlyphs.ts](./client/src/assets/icons/themedGlyphs.ts) + theme-aware [NodeIcon.tsx](./client/src/assets/icons/NodeIcon.tsx)), **per-theme canvas-grid + custom cursors** via `--canvas-grid` / `--cursor-default` slots, **decorative HTML primitives** (`<SvgFilterDefs>` mounting `#ink-blot` / `#noise` / `#crt` filter IDs at app root, `<DropCap>` wrapper for Renaissance ornament rule), **parameter panel migrated to Tailwind tokens** (no `useAppTheme()` reads; section headers carry the display-typography triplet; raw `<Button>` swapped to `<ActionButton intent>`), **per-theme scrollbar webkit rules** in all 10 themes, 9-event WebAudio sound system (10 packs via `--sound-pack` token + `useSound()` hook + global hover delegate + sonner toast monkey-patch + `withSound()` HOC + `Sounds.unlock()` gesture-unlock for AudioContext autoplay-policy compliance), `@media (prefers-reduced-motion: reduce)` accessibility, 30 ms throttle on `type` / `hover`, migration recipe, anti-patterns. Read this before adding a new theme, migrating a component to the new contract, or adding a canvas-node component. |
| **[Schema Source of Truth RFC](./docs-internal/schema_source_of_truth_rfc.md)** | Backend is SSOT for node schemas, visual metadata, handlers, palette metadata, icons. Plugin pattern: one `BaseNode` subclass in `server/nodes/<group>/<name>.py`. Wire format: `asset:<key>` / `<lib>:<brand>` / URL / emoji. Endpoint: `/api/schemas/nodes/{type}/spec.json`. 124 pytest invariants lock the contract. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeenie-ai/MachinaOS](https://github.com/zeenie-ai/MachinaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
