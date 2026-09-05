---
trigger: always_on
description: Four browser-based editors, three for embedded devices with monochrome
---

# Pixpix Studio

Four browser-based editors, three for embedded devices with monochrome
displays plus one general-purpose RGB sprite editor:

- **Scene editor** (`/scene`) — draws into a packed 1-bpp pixel buffer (like a
  real display framebuffer) and generates
  [u8g2](https://github.com/olikraus/u8g2) C/C++ or XBM code from the scene.
- **Font editor** (`/font`) — a BDF glyph editor.
- **Icon editor** (`/icon`) — edits many named icon bitmaps that share one
  fixed size (an icon set) and generates u8g2-ready XBM C byte arrays.
- **Sprite editor** (`/sprite`) — edits many named sprites that share one
  fixed size (a sprite set), with a full RGB color per pixel instead of the
  1-bit pixels the other three editors use. No code generation (u8g2 is
  1bpp-only and has no natural RGB target).

Editing works anonymously with no account, but nothing persists until you
sign in: the scene, the font, the icon set and the sprite set all stay in
memory only (import/export). Signing in with GitHub adds a **dashboard**
(`/dashboard`) that saves scenes, fonts, icon sets and sprite sets per-user to
a Cloudflare D1 database, so they can be reopened from any browser.

Astro, server-rendered, + React islands, deployed to Cloudflare Workers
(`@astrojs/cloudflare`) with a D1 binding — not a static-assets-only site.

> `CLAUDE.md` is a symlink to this file. Edit `AGENTS.md`.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

Other commands:

- `npm run build` / `npm run preview` — production build and local preview
- `npm run generate-types` — `wrangler types` (Cloudflare Worker bindings)
- `node tools/generate-fonts.js` — regenerate `src/font-data.ts` from `res/bdf/`

Database (D1 via Drizzle, no npm script wraps these yet — run directly):

- `npx drizzle-kit generate` — add a migration under `drizzle/migrations` after
  editing `src/lib/db/schema.ts`
- `npx wrangler d1 migrations apply DB --local` / `--remote` — apply pending
  migrations to the local dev DB or the remote D1 database

Local dev needs a `.dev.vars` (gitignored, see `.dev.vars.example`) with
`GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` (GitHub OAuth app) and
`BETTER_AUTH_SECRET`.

There is no test framework. Verify changes by running the dev server and
exercising the editor.

## Layout of the source tree

```
src/
  pages/
    index.astro     landing page; redirects to /dashboard if signed in
    login.astro     GitHub sign-in (better-auth), redirects to /dashboard if signed in
    dashboard.astro auth-required; redirects to /login if not signed in
    scene.astro     scene editor; ?id= loads a saved scene (auth-required only then)
    font.astro      font editor; ?id= loads a saved font (auth-required only then)
    icon.astro      icon editor; ?id= loads a saved icon set (auth-required only then)
    sprite.astro    sprite editor; ?id= loads a saved sprite set (auth-required only then)
    api/
      auth/[...all].ts    better-auth catch-all handler
      scenes/index.ts, scenes/[id].ts             CRUD for the signed-in user's scenes
      fonts/index.ts, fonts/[id].ts               CRUD for the signed-in user's fonts
      icon-sets/index.ts, icon-sets/[id].ts       CRUD for the signed-in user's icon sets
      sprite-sets/index.ts, sprite-sets/[id].ts   CRUD for the signed-in user's sprite sets
  apps/
    scene-editor/  the u8g2 scene editor app (AppContext, engine, UI, commands)
    font-editor/    the BDF glyph editor app (self-contained)
    icon-editor/    the icon set editor app (self-contained)
    sprite-editor/  the sprite set editor app (self-contained, RGB pixels)
    dashboard/      lists/manages a signed-in user's saved scenes, fonts, icon sets and sprite sets
  components/
    editor/         reusable editor core — canvas, shapes, tools, undo/redo
    ui/             shadcn components
    icons/          hand-written SVG icons
    dialogs/        confirm + code dialogs (each owns a small zustand store)
    astro/head.astro, logo.tsx, appbar.tsx
  lib/
    utils.ts        cn, detectPlatform, generateNewName, odd
    auth.ts         getAuth() — lazy better-auth singleton (GitHub OAuth + D1)
    auth-client.ts  authClient — better-auth browser client (signIn/signOut)
    db/             schema.ts (Drizzle), index.ts (getDb()), fonts.ts, scenes.ts, icon-sets.ts, sprite-sets.ts
  middleware.ts     resolves the session on every request into Astro.locals
  font-data.ts      generated — embedded BDF fonts (deflate + base64)
  styles/           global.css (Tailwind v4 + theme), fonts.css (@font-face)
```

All pages render their app's `<App client:only="react" />` inside
`<body class="dark">` (there is no light theme) with the shared
`components/astro/head.astro`, which takes an optional `title` and includes
the Google Analytics tag. `scene.astro`/`font.astro`/`icon.astro`/
`sprite.astro`/`dashboard.astro` share `components/appbar.tsx` for their
header (logo, current-page label, a "Dashboard" back-link, and app-specific
actions on the right).

The apps share `components/editor/`, `ui/`, `icons/`, `dialogs/`, `logo.tsx`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niklauslee/pixpix-studio](https://github.com/niklauslee/pixpix-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
