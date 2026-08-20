---
trigger: always_on
description: Interactive SvelteKit 5 portfolio app integrating GitHub, Spotify, and Steam APIs. Designed as a publishable Svelte library (`svelte-package`) and a standalone showcase app.
---

# Project-Me — Copilot Instructions

## Overview

Interactive SvelteKit 5 portfolio app integrating GitHub, Spotify, and Steam APIs. Designed as a publishable Svelte library (`svelte-package`) and a standalone showcase app.

## Architecture

### Schema-First Types

Types are defined in YAML (`docs/types/*.yaml`) and code-generated into `src/generated/types/` via **type-crafter**.

- **Rule**: ALL shared types and configuration schemas must be defined in YAML.
- **Workflow**: Edit YAML -> `pnpm generate` -> Use generated types.
- **Never** hand-write types in `src/generated/` or manually type configuration files without a backing schema.

### Source-Based Polymorphism

The `Source` union (`"github" | "spotify" | "games"`) from `src/generated/types/Projects.ts` drives most of the app: API routing (`/api/infos?source=...`), navigation menus, theme selection, and UI rendering. When adding a new data source, update the YAML schema, regenerate types, and wire through configuration, services, and stores.

### Client ↔ Server Split

- **`src/lib/services/`** — Server-side services that call external APIs (GitHub, Spotify, Steam, mailer). Each uses `APICaller` with generated decoders.
- **`src/lib/client/`** — Browser-side fetchers that call internal SvelteKit API routes. Check store state (`null` = unfetched) before fetching.
- **`src/lib/server/`** — Server-only utilities: `APIResponseHandler` (builder pattern for standardized responses) and domain mappers.

### Component Architecture

- **Modular Design**: Break down complex UIs into small, single-purpose components (e.g., `Badge`, `Button`, `Card`).
- **Data-Driven**: Components should receive data via props, typically sourced from typed configuration files.
- **Icons & SVGs**: Use dedicated Svelte components for all icons and graphical assets (e.g., `src/lib/components/icons/PackageIcon.svelte`). **NEVER** use inline `<svg>` blocks directly inside feature components or pages. Always extract them to the `$components/icons` directory.
- **Composition**: Build "Section" components (e.g., `Technical.svelte`) by composing smaller UI atoms.

### Writing CSS, Themes, and Class Names

- **Theme Variables Only**: **NEVER** hardcode hex codes (e.g., `#FFFFFF`), `rgba()` strings, or pixel sizes directly in component `<style>` blocks or feature SCSS files. Every color must exist as CSS custom properties (`var(--color-...)`) inside `src/css/theme.scss`.
- **Class naming**: Class names should be in kebab-case and should be descriptive of the element's purpose or content. For example, use `class="profile-card"` instead of `class="card1"`.
- **Global Abstractions**: We can use generic class names whose styles would be defined in `src/css/style.scss`. For example, a generic `class="button"` defined in `style.scss` can be used across multiple components.
- **Typography & Scale**: Utilize `clamp()` for responsive, fluid typography (especially for massive Hero headers) to ensure seamless scaling between mobile and desktop without excessive media queries.

### API Route Pattern

All routes in `src/routes/api/` follow this template:

```typescript
export async function GET({ url, request }: RequestEvent) {
  const tag = "GET /api/...";
  logger.logServerRequest(tag, { ... });
  try {
    // business logic using services + decoders
    response = APIResponseHandler.successResponse("message", data);
  } catch (err) {
    logger.logException(tag, String(err));
    response = APIResponseHandler.badRequestResponse(String(err));
  }
  logger.logServerResponse(tag, { ... });
  return APIResponseHandler.toResponse(response);
}
```

Use `APIResponseHandler` static factories (`successResponse`, `badRequestResponse`, `notFoundResponse`, etc.) — never construct raw `Response` objects in API routes.

### Stores

Svelte `writable` stores with typed state objects and dedicated updater functions. Convention: `null` means "not yet fetched." See `src/lib/stores/site.ts` (theme/source/loader) and `src/lib/stores/skills.ts` (cached API data).

## Path Aliases

Defined in both `svelte.config.js` and `vite.config.ts`. Always use these in imports:

| Alias            | Path                    |
| ---------------- | ----------------------- |
| `$generated`     | `src/generated`         |
| `$services`      | `src/lib/services`      |
| `$stores`        | `src/lib/stores`        |
| `$configuration` | `src/lib/configuration` |
| `$components`    | `src/lib/components`    |
| `$pages`         | `src/lib/pages`         |
| `$constants`     | `src/lib/constants`     |
| `$server`        | `src/lib/server`        |
| `$client`        | `src/lib/client`        |
| `$models`        | `src/lib/models`        |
| `$css`           | `src/css`               |

## Key Libraries

- **vergins** — UI component library (Navbar, Flyer, TimeLine, etc.) and its shipped CSS
- **lilac-typescript** — Structured logger (`ProcessLogger`); use `logger.logServerRequest`, `logServerResponse`, `logException`, `logExternalApiRequest/Response`
- **type-decoder** — Runtime decoding primitives (`decodeString`, `decodeNumber`, `isJSON`, etc.) used inside generated decoders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmreetKumarkhuntia/project-me](https://github.com/AmreetKumarkhuntia/project-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
