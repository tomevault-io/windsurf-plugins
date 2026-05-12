---
trigger: always_on
description: - **Directory size limit**: A single directory must not exceed **10** direct children (files + subdirectories). Split by responsibility when approaching this limit.
---

# ContextGo - Project Guide

## Code Conventions

### File & Directory Structure

- **Directory size limit**: A single directory must not exceed **10** direct children (files + subdirectories). Split by responsibility when approaching this limit.

See [docs/conventions/file-structure.md](docs/conventions/file-structure.md) for complete rules on directory naming, page module layout, and shared vs private code placement. Agents working in this repository must also read and follow the `architecture` skill (`.claude/skills/architecture/SKILL.md`) when creating files, modules, or making structure decisions.

### Naming

- **Components**: PascalCase (`Button.tsx`, `Modal.tsx`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Hooks**: camelCase with `use` prefix (`useTheme.ts`)
- **Constants files**: camelCase (`constants.ts`) — values inside use UPPER_SNAKE_CASE
- **Type files**: camelCase (`types.ts`)
- **Style files**: kebab-case or `ComponentName.module.css`
- **Unused params**: prefix with `_`

### UI Library & Icons

- **Components**: `@arco-design/web-react` — no raw interactive HTML (`<button>`, `<input>`, `<select>`, etc.)
- **Icons**: `@icon-park/react`
- **Icon alignment**: renderer entry must keep `@icon-park/react/styles/index.css` imported globally; for icon+text rows, menu items, and icon buttons, use the shared classes in `src/renderer/styles/icon.css` (`app-icon`, `app-icon-slot`, `app-icon-row`, `app-icon-button`) instead of per-component `marginTop` / inline `lineHeight` fixes

### CSS

- Prefer **UnoCSS utility classes**; complex styles use **CSS Modules** (`ComponentName.module.css`)
- Colors must use **semantic tokens** from `uno.config.ts` or CSS variables — no hardcoded values
- Arco overrides go in the component's CSS Module via `:global()` — no global override files
- Global styles only in `src/renderer/styles/`

See [docs/conventions/file-structure.md](docs/conventions/file-structure.md) for full CSS and UI library rules.

### TypeScript

- Strict mode enabled — no `any`, no implicit returns
- Use path aliases: `@/*`, `@process/*`, `@renderer/*`, `@worker/*`
- Prefer `type` over `interface` (per Oxlint config)
- English for code comments; JSDoc for public functions

### Architecture

Three process types — never mix their APIs:

- `src/process/` — main process, no DOM APIs
- `src/renderer/` — renderer, no Node.js APIs
- `src/process/worker/` — fork workers, no Electron APIs

Cross-process communication must go through the IPC bridge (`src/preload.ts`).
See [docs/tech/architecture.md](docs/tech/architecture.md) for details.

### Connector Terminology Boundary

When changing connectors, channels, IM publishing, or external product access, keep these terms distinct:

- `Context Connector` means a cross-product context-access capability such as browser activity, Feishu OpenAPI, Google Workspace, GitHub, or another external datasource/product surface
- `IM Bot Channel` / `channel account` means the transport and publication surface used to publish an Agent into Telegram, Slack, Discord, Lark, DingTalk, or WeChat
- do not use `connector` to mean IM bot channel in new docs, UI copy, or API names; keep that wording only where a legacy compatibility alias already exists
- treat Feishu as two separate boundaries:
  - `IM Channels > Lark` configures the bot transport used for Agent publication into Feishu/Lark conversations
  - `Context Connector > Feishu` configures the `lark-cli` / `cgo feishu` runtime used to access docs, files, calendar, chat history, and other context surfaces

Read these before changing the model:

- [docs/tech/repo-topology.md](docs/tech/repo-topology.md)
- [src/process/services/space/connectors/README.md](src/process/services/space/connectors/README.md)
- [src/process/channels/ARCHITECTURE.md](src/process/channels/ARCHITECTURE.md)

### Context Engine Index

When changing the Context Engine, session/project/space memory modeling, vault sync, context jobs, event-driven context flows, or connector digestion, read these first:

- [docs/tech/context-engine-event-architecture.md](docs/tech/context-engine-event-architecture.md)
- [packages/context-engine/docs/domain-model.md](packages/context-engine/docs/domain-model.md)
- [packages/context-engine/docs/reference-landscape.md](packages/context-engine/docs/reference-landscape.md)

Current implementation anchor points:

- `src/process/services/context/ContextRuntimeService.ts`
- `src/process/services/context/ContextJobOrchestrator.ts`
- `src/process/services/context/contextDomain.ts`
- `src/process/services/space/SpaceVaultContextSyncService.ts`

### Space Product Boundary

When changing `Space`, canvas/doc editing, collaboration surfaces, members/roles, or context governance:

- treat `Space` as a first-class ContextGo product object, not as a thin wrapper around any external product
- do not expose third-party product names such as `AFFiNE` in user-facing UI copy, navigation labels, product concepts, or default empty states
- third-party editor/canvas code may be absorbed as implementation detail, but the product surface must stay branded and modeled as ContextGo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contextgo/contextgo](https://github.com/contextgo/contextgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
