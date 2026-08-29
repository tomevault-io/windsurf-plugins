---
trigger: always_on
description: AI-personalizable market dashboard framework (working name — do NOT call it "hyperframes"; that's HeyGen's project, which we use as the distribution-model reference). An agent reads the frame catalogue (Zod → JSON Schema), emits a `dashboard.json` spec, and the runtime validates + renders it as a live dashboard.
---

# zframes

AI-personalizable market dashboard framework (working name — do NOT call it "hyperframes"; that's HeyGen's project, which we use as the distribution-model reference). An agent reads the frame catalogue (Zod → JSON Schema), emits a `dashboard.json` spec, and the runtime validates + renders it as a live dashboard.

## Commands

```bash
pnpm install
pnpm dev          # runtime at http://localhost:37263 (project default port, strict)
pnpm typecheck    # tsc --noEmit across all workspace packages (pnpm -r)
pnpm lint         # eslint . — also where the layer DAG is enforced (no-restricted-imports)
pnpm format       # prettier --write . (pnpm format:check is the CI gate)
pnpm build        # vite build of the runtime
pnpm build:cli    # build the zframes CLI + its prebuilt runtime bundle
pnpm zframes serve <dashboard.json>   # the runtime: serve a dashboard live (--port to change)
pnpm --filter @zframes/storybook dev  # Storybook — every frame in all variants/states at :6006
pnpm test:providers      # LIVE smoke: hit every keyless provider's real API, assert the response shape
pnpm test:frames:render  # headless-render every frame in a built Storybook, flag error cards / crashes
pnpm test:e2e            # Playwright e2e: runtime (fixture board + save round-trip) then explorer (needs Docker)
```

`pnpm test` (hermetic, stubs fetch) and `pnpm format:check` gate every PR. Test-suite
map, e2e, scheduled monitors → [tests/AGENTS.md](tests/AGENTS.md). Releasing the CLI
→ [packages/cli/AGENTS.md](packages/cli/AGENTS.md).

## Package topology

Core was decomposed into bounded-context packages (2026-07-01) and the transitional
facade is gone (2026-07-03): **every consumer imports the leaf package directly.**

| Package | Role | May import |
|---|---|---|
| `@zframes/spec` | domain kernel: types, `DashboardSpecSchema`, routes, frame/registry, presets, catalogue | nothing |
| `@zframes/data-primitives` | `fetch` + `cache` transport primitives, `csv` row parsing | spec |
| `@zframes/store` | the XDG global store (Node) | nothing |
| `@zframes/zai` | agent-orb harness (Node) | spec |
| `@zframes/account` | keyed-account HMAC relay + credentials (Node) | spec, store |
| `@zframes/serve` | spec read/write + official-data proxy (Node) | spec |
| `@zframes/vite` | the `dashboardWriteback()` dev plugin (Node, composition only) | serve, zai, account, store |
| `@zframes/core` | presentation: `DashboardRenderer`, `frame-content`, capability hooks | spec |
| `@zframes/editor` | the `DashboardEditor` authoring UI (React + GridStack) | core, spec |
| `@zframes/charts` | D3 base chart layer | nothing |
| `@zframes/unicorn` | shared Unicorn Studio scene loader + backdrop gates | nothing |
| `@zframes/frames` | the frames | charts, core, spec |
| `provider-*` (31) | React-free data adapters | spec, data-primitives |
| `@zframes/provider-demo` | the synthetic default source: seeded data for every capability, zero network | spec |
| `@zframes/providers-keyless` | composition leaf: the shipping keyless fleet | the providers, spec |
| `@zframes/plugins` | composition leaf: the built-in plugin registry (`/registry`, manifests, Node) + browser loader (`/load`, one lazy chunk per plugin) | providers-keyless, provider-demo, provider-binance, provider-wallet, spec |

ESLint `no-restricted-imports` enforces this DAG per directory, `tests/dep-dag.test.ts`
pins it at the package.json level, and `packages/core/src/barrel-surface.test.ts`
snapshot-pins the public core barrel. Node-loaded files (serve/vite/account/agent)
import siblings by **package subpath**, never relative.

**Where new code goes** (always import the leaf package):

- New spec field / frame-meta shape / theme preset / catalogue change → `packages/spec` (then the `--zf-*` mapping in core's `frame-content.tsx`/`renderer.tsx` and the editor rail if it's cosmetic).
- Provider-plugin contract (what an installed adapter declares: capabilities, hosts, credits, terms) → `packages/spec/src/provider-plugin.ts`, reached at the `@zframes/spec/provider-plugin` subpath. **Its helpers must NOT be re-exported through the spec root barrel** — core mirrors that barrel, so a value there lands on the presentation package's public API. → [packages/spec/AGENTS.md](packages/spec/AGENTS.md)
- Transport, caching or delimited-response parsing (fetch, proxy rewrite, TTL/dedup/persist, CSV rows) → `packages/data-primitives`.
- New capability hook / renderer chrome → `packages/core` (`hooks.tsx` / `frame-content.tsx`).
- Authoring-UI behaviour (palette, rail, grid interactions, default-config seeding) → `packages/editor`.
- Spec read/write or proxy route → `packages/serve`; new proxied provider host → the `hosts` list (with `proxied: true`) of the plugin manifest that fetches it — every mount derives its allowlist from the manifests it mounts (`proxyHostsOf`), so the manifest entry IS the grant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zentryHQ/zframes](https://github.com/zentryHQ/zframes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
