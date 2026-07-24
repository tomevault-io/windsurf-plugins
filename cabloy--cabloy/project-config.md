---
trigger: always_on
description: This repository is **Cabloy Basic**, the public framework/reference edition, identified by the root marker file:
---

# Cabloy Monorepo Guidance

## Repository identity

This repository is **Cabloy Basic**, the public framework/reference edition, identified by the root marker file:

- `__CABLOY_BASIC__`

A sibling repository named `cabloy-start` is the private commercial edition, delivered as a licensed separate source repository and identified by:

- `__CABLOY_START__`

Always detect the active edition before making UI-sensitive assumptions, choosing frontend examples, recommending module-specific workflows, or assuming the project creation path.

## Monorepo layout

- `package.json` is the primary shared workflow entrypoint.
- `vona/` contains the backend framework, backend modules, and the Vona CLI.
- `zova/` contains the frontend framework, frontend modules, and the Zova CLI.
- `cabloy-docs/` contains the unified public documentation.
- `.docs-internal/` contains internal engineering notes and ADRs.
- `.claude/` contains Claude commands, skills, and settings.

## Preferred workflow

Before inventing a custom implementation path:

1. check the root `package.json`
2. detect the edition marker
3. inspect the relevant CLI entrypoint:
   - `npm run vona`
   - `npm run zova`
4. use existing command families before writing framework scaffolding manually
5. verify the result with targeted checks or shared root scripts

## Documentation boundary

- Put user-facing and agent-facing guidance in `cabloy-docs/`.
- Put maintainer rationale, architecture notes, and ADRs in `.docs-internal/`.
- Do not mix internal rationale into public how-to pages unless a trimmed user-facing explanation is genuinely needed.

## AI development rules

- Prefer CLI-backed workflows over manual scaffolding whenever Vona or Zova already provides a generator, refactor, metadata, or verification command.
- The Cabloy contract-loop model applies to both Cabloy Basic and Cabloy Start; detect the edition to choose commands and output paths, not to redefine the workflow model.
- Treat contract-loop work as one of four branches: forward chain, reverse chain, consumer drift, or local dependency drift.
- For the forward chain, change backend contract truth first and regenerate frontend consumers rather than hand-patching them.
- After forward regeneration, keep frontend follow-up thin: prefer semantic model facades and reuse the existing resource-owner when the custom API still belongs to the same resource.
- When one persisted business resource serves both Admin Resource and Web self-service consumers, keep one domain and persistence boundary; where audience authority or experience differs, split API/DTO contracts, server-side scope, frontend state ownership, and page architecture—reuse the generic Resource owner for Admin and use dedicated state/pages for a genuinely distinct Web self-service contract.
- For the reverse chain, always run the relevant Zova build first, then run `npm run deps:vona`: use `npm run build:zova:admin` for Admin changes, and also run `npm run build:zova:web` when the Web flavor is affected. Do not treat `build:rest:*` alone as sufficient, because the SSR bundle and rest output must move together.
- If the generated `.zova-rest` artifacts already contain the expected changes but Vona consumers still see stale types after `npm run deps:vona`, treat it as local dependency drift: delete `vona/node_modules` and reinstall dependencies before further debugging or hand-patching dependency links.
- For Cabloy Start, apply the same reverse-chain logic but resolve the Start-specific flavor names and generated-output paths from the active Start repo before recommending commands.
- Treat legacy docs as input material, not as unquestioned truth. When docs conflict with source code, prefer current source code.
- For frontend work, assume Cabloy Basic and Cabloy Start share a frontend engineering layer but may diverge in UI layer, frontend flavors, suite/module availability, SSR site baselines, project assets, and generated outputs.
- For Zova frontend analysis, do not default to generic Vue reinterpretation first. Read the code through Zova’s controller / bean / IoC architecture before mapping it to Vue concepts.
- For Zova source-reading or Vue-vs-Zova explanation tasks, start from the frontend reading guides and source-reading map in `cabloy-docs/frontend/` before doing framework-neutral reinterpretation.
- For frontend async state that affects rendering or interaction across consumers, prefer model-owned `$useStateData(...)` over controller-managed fetch/cache state.
- Default to establishing such query state during render. Use `disableSuspenseOnInit: true` only for relatively stable query-backed state when you want to skip the init-time `query.suspense()` kick; it does not prevent query creation, fetches, or hydration-time rendering. If strict readiness is needed later, wait explicitly at the interaction boundary.
- In SSR, keep server HTML and the client's hydration-time initial render equivalent. When server rendering intentionally omits private, cookie-unavailable, or browser-only state, keep the same neutral shell or placeholder through hydration and defer its query/load/render branch to an explicit post-hydration, admission, mounted, or interaction boundary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cabloy/cabloy](https://github.com/cabloy/cabloy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
