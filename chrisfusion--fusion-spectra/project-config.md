---
trigger: always_on
description: Vue 3 + Quasar 2 + Vite micro-frontend shell (Module Federation host).
---

# fusion-spectra

Vue 3 + Quasar 2 + Vite micro-frontend shell (Module Federation host).
Dev server: `npm run dev` → http://dev.fusion.local:5174
Requires `127.0.0.1 dev.fusion.local` in `/etc/hosts` (localhost breaks SameSite=Lax cookie sharing with bff.fusion.local)
Vite config: `server.host:'0.0.0.0'`, `server.port:5174`, `server.allowedHosts:['dev.fusion.local']`
Type check: `npm run typecheck`

## CHANGELOG maintenance
- `CHANGELOG.md` is the project changelog — update it for every feature and bugfix going forward
- Format: add entries under `## [Unreleased]` with today's date as a comment; use `### Added`, `### Changed`, `### Fixed` subsections; one line per item
- When a deployment bumps the image tag (values-dev.yaml), also promote `[Unreleased]` to a versioned section (e.g. `## [0.9.1] — YYYY-MM-DD`) matching the new tag
- Use `date +%Y-%m-%d` in Bash to get today's date when writing changelog entries

## README maintenance
- `README.md` feature-status claims (Live/Placeholder) drift from reality easily — verify against `src/pages/<context>/` file listing and `src/data/navigation.ts` before trusting or updating them
- `INSTALL.md` does not exist (never committed, despite historical references) — don't link to it; point setup/deployment docs at `CLAUDE.md` and `ARCHITECTURE.md` instead

## Stack
- Vue 3, Quasar 2, Pinia, Vue Router 4, Vite 5
- Icons: `@quasar/extras` mdi-v7 (use `mdi-*` names)
- Fonts: DM Sans (UI), JetBrains Mono (data/mono) — loaded via Google Fonts in index.html
- CSS custom properties in `src/css/app.scss` (all `--fs-*`)
- Quasar theme vars in `src/css/quasar-variables.scss`

## Layout architecture
- `src/layouts/MainLayout.vue` — shell: topbar + activity rail + sidebar + canvas
- Activity rail (`src/components/ActivityRail.vue`) — three-zone model: regular (top) → separator (flex:1) → util (bottomUtil) → admin (adminOnly); admin section gated by `isAdmin`, util section always visible
- Sidebar (`src/components/AppSidebar.vue`) — IDE-style tree; 3-level when `NavGroup.section` is set (renders a small uppercase section header when label changes across sibling groups); `NavLeaf.tooltip` renders a `q-tooltip` on hover (400ms delay, anchors right)
- Canvas panels use `src/components/CanvasPanel.vue`
- Context/nav data: `src/data/navigation.ts` — single source of truth

## CanvasPanel component
Props: `title`, `icon`, `wide` (span 2 cols), `loading` (spinner overlay), `error` (error state + retry).
Emits: `refresh`. Slots: default body, `actions` (header right area).
No footer slot — add pagination below the table inside the default slot.

## Contexts (activity rail order)
1. Data → `/data`
2. Weave → `/pipelines` (label "Weave", id `pipelines`) — groups use `section` to form two topics: **Runs** (Monitoring + Control sub-groups) and **Blueprints** (Run Blueprints + Step Blueprints); all "templates" renamed to "blueprints" in labels
3. Monitoring → `/monitoring`
4. Forge → `/forge` — async Python venv builder (fusion-forge backend)
5. Fusion Index → `/fusion-index` — live registry UI backed by fusion-index API
6. Admin → `/admin` (admin-only, amber accent, bottom of rail)

## Auth
- BFF owns all OIDC — frontend knows nothing about Keycloak or tokens
- Auth store (`src/stores/auth.ts`): `init()` calls `GET /bff/userinfo` with `credentials:'include'`; 401 → `window.location.href = bffUrl + '/bff/login'`
- `UserInfo` shape: `{ sub, email, name, roles: string[], permissions: string[], resource_permissions: ResourcePermission[] }` — populated from BFF session
- `ResourcePermission` shape: `{ permission: string, resource_type: string, resource_id: string }` — resource-scoped grants
- Router guard in `src/router/index.ts` calls `auth.init()` on every navigation; routes with `meta.adminOnly: true` redirect non-admins to `/data`
- BFF URL from `src/config/runtime.ts` → `window.FUSION_CONFIG.bffUrl` → `VITE_BFF_URL` → `http://bff.fusion.local`
- Runtime config file: `public/config.js` (overridden by ConfigMap mount in K8s)

## RBAC (permissions)
- `src/composables/usePermission.ts` — call `usePermission()` in any component that needs access control
  - `can(permission: string, resourceId?: number | string)` — true if user has global permission OR a resource-scoped grant for that ID
  - `hasRole(role: string)` — true if `auth.user.roles` contains the role
  - `isAdmin` — computed: `hasRole('admin')`
- Gate UI elements with `v-if="can('index:artifacts:delete')"` etc., NOT with role checks — roles are too coarse for UI gates
- Resource-scoped gating: `v-if="can('index:artifacts:delete', artifact.id)"` — true only if user has global perm OR a specific grant for that resource
- Admin icon in ActivityRail is hidden via `v-if="isAdmin"` — no admin entry renders for non-admin users
- Admin routes (`/admin/*`) have `meta.adminOnly: true`; the router guard redirects to `/data` if user lacks `admin` role
- Permission strings mirror the BFF `rbac.yaml` (e.g. `index:artifacts:read`, `forge:builds:create`, `admin:roles:manage`)

## API clients
- `src/api/bffClient.ts` — base fetch with `credentials:'include'`; 401 auto-redirects to BFF login
  - FormData detection: skips `Content-Type: application/json` when `body instanceof FormData` (multipart uploads)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisfusion/fusion-spectra](https://github.com/chrisfusion/fusion-spectra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
