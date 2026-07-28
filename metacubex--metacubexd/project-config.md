---
trigger: always_on
description: metacubexd is the official dashboard and managed runtime for the Mihomo proxy
---

# Copilot Instructions for metacubexd

metacubexd is the official dashboard and managed runtime for the Mihomo proxy
kernel. Keep changes inside the owning workspace and preserve the boundary
between Mihomo's Clash API and metacubexd's Control API.

## Read First

- [CONTEXT.md](../CONTEXT.md) defines the project's domain language.
- [packages/ui/PRODUCT.md](../packages/ui/PRODUCT.md) defines the product and its
  users.
- [packages/ui/DESIGN.md](../packages/ui/DESIGN.md) defines the UI design system.
- [packages/agent/MANUAL.md](../packages/agent/MANUAL.md) contains real-kernel
  smoke tests that are intentionally outside CI.
- [README.md](../README.md) documents supported deployment forms.

## Monorepo Map

This is a pnpm 10 workspace with four workspaces:

| Workspace        | Responsibility                                                                                            |
| ---------------- | --------------------------------------------------------------------------------------------------------- |
| `packages/ui`    | Nuxt 4/Vue 3 dashboard shared by every runtime form                                                       |
| `packages/agent` | Framework-neutral Control API, profile store, kernel supervisor, scheduler, and shared types              |
| `apps/server`    | Nitro all-in-one server that serves the UI and mounts the agent                                           |
| `apps/desktop`   | Electron shell, local control server, OS integration, privileged TUN helper, and bundled kernel packaging |

Do not move host-specific behavior into `packages/ui`. Put reusable lifecycle,
profile, and Control API behavior in `packages/agent`; keep Docker/Nitro wiring in
`apps/server` and Electron/OS wiring in `apps/desktop`.

## Runtime Forms

There are three runtime forms:

1. **Hosted panel**: the static UI connects directly to a user-managed Mihomo.
   There is no Control Agent, so agent-only features remain hidden. The
   standalone panel Docker image is another distribution of this form.
2. **Desktop app**: Electron serves the bundled UI from a loopback control
   server, runs a local Control Agent, and supervises a bundled Mihomo. The
   preload bridge injects the per-launch Control and Clash endpoints.
3. **All-in-one server**: Nitro serves the UI and `/api/control` on the control
   port and supervises the bundled Mihomo in the same container. The Clash API
   and mixed proxy remain separate ports.

Default server ports are `8080` for UI + Control API, `9090` for the Clash API,
and `7890` for the mixed proxy. Desktop ports are selected from free loopback
ports at startup; never hard-code them in UI code.

## API Boundary

- **Clash API** is Mihomo's `external-controller` HTTP/WebSocket surface. It owns
  proxies, proxy groups, traffic, connections, rules, configs, version, and live
  Clash log data. UI access is centered on `packages/ui/composables/useApi.ts`,
  `useWebSocket.ts`, and the endpoint store.
- **Control API** is metacubexd's `/api/control/**` surface. It owns kernel
  lifecycle and subprocess logs, profiles, runtime config, subscriptions,
  kernel/Geo asset management, WebDAV backup, System Proxy, and TUN. UI access
  goes through `packages/ui/composables/useControlApi.ts`; features are gated by
  the agent's advertised capabilities.

Do not send Clash API traffic through the Control API. In server mode the UI
talks directly to the published Clash API port because Nitro does not proxy the
required WebSocket streams. Do not confuse Clash WebSocket logs with the
agent's kernel-process SSE logs.

## UI Stack and Conventions

- Nuxt 4, Vue 3, strict TypeScript, CSR-only rendering, and hash routing.
- Pinia owns shared client state; persistent state commonly uses VueUse
  `useLocalStorage`.
- TanStack Vue Query owns server-state queries and invalidation. Keep query keys
  stable and invalidate affected data after mutations.
- `ky` v2 is the HTTP client. This version uses `prefix`, not `prefixUrl`.
- Tailwind CSS v4 and daisyUI v5 provide styling. Follow semantic daisyUI roles
  and the design system rather than hard-coded theme colors.
- Vue, Nuxt, VueUse, project composables, stores, utilities, constants, types,
  and components are auto-imported according to `packages/ui/nuxt.config.ts`.
- Use `<script setup lang="ts">`, explicit props/emits types, computed values for
  derived state, and watchers only for side effects.

Do not assume Zod, `tailwind-merge`, `tailwind-variants`, or
`@tanstack/vue-table` exists in this repository. Tables and conditional classes
use project components and normal Vue/Tailwind patterns.

### Internationalization

All user-facing UI text must use `useI18n()`. Keep the seven JSON locale files
in `packages/ui/i18n/locales/` aligned:

`en.json`, `fa.json`, `fr.json`, `ja.json`, `ko.json`, `ru.json`, `zh.json`.

Add the same key to every locale and preserve valid JSON. Do not create legacy
TypeScript locale modules.

## Commands

Run commands from the repository root unless noted.

### Root scripts

```bash
pnpm install       # install the workspace
pnpm dev           # alias for dev:ui; pure-panel Nuxt development
pnpm dev:ui        # Nuxt UI development server
pnpm dev:server    # generate the UI, then start Nitro development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaCubeX/metacubexd](https://github.com/MetaCubeX/metacubexd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
