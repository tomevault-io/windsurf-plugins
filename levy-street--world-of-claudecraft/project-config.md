---
trigger: always_on
description: <!-- World of ClaudeCraft, project-root CLAUDE.md. Keep this lean (about 200 lines)
---

<!-- World of ClaudeCraft, project-root CLAUDE.md. Keep this lean (about 200 lines)
     and strictly repo-wide. Area-specific guidance lives in each subdirectory's own
     CLAUDE.md (src/sim/, src/render/, server/, ...), which load on demand when you
     open files there, so do NOT duplicate them here. Anchor guidance on stable paths,
     symbols, and pinned tests, never on counts that rot. HTML comments like this are
     stripped before load (zero tokens). No em dashes, en dashes, or emojis. -->

# World of ClaudeCraft

A classic-style micro-MMO **and** a headless reinforcement-learning
environment, both driven by one deterministic TypeScript simulation core.
Stack: TypeScript (ESM, `strict`) · Three.js renderer · `ws` WebSockets ·
Postgres (`pg`) · Vite + esbuild · Vitest. No UI framework in the game client; tiny
dependency set. The one sanctioned exception is the standalone admin dashboard
(`src/admin/`), which is built with Svelte 5 (it never touches the game client bundle).

## Repo map
| Path | What it is |
|---|---|
| `src/sim/` | **Deterministic game core, the source of truth.** No DOM/Three deps; runs in browser, server, and headless. |
| `src/sim/content/` | Data-as-code: classes, abilities, talents, zones, dungeons, items, professions, mounts, deeds, Reliquary pages. |
| `src/render/` | Three.js renderer (procedural geometry/textures/VFX + curated GLBs). Reads the world; never mutates it. |
| `src/game/` | Local input, camera, keybinds, gamepad, mobile controls, sampled WebAudio SFX, and procedural music. |
| `src/ui/` | Classic HUD (frames, windows, tooltips, map, FCT), procedural icons, i18n. |
| `src/styles/` | Extracted HUD CSS under one `@layer` order, imported once via `src/main.ts`. See `src/styles/CLAUDE.md`. |
| `src/net/` | Online client: REST auth + WebSocket world mirror (`ClientWorld`), reconnect, native-app glue, wallet glue. |
| `src/admin/` | Admin dashboard SPA (separate `admin.html` entry). |
| `src/guide/` | Public guide/wiki SPA (separate `guide.html` entry, served at `/wiki`); spoiler-safe content generated from `src/sim/`. |
| `src/editor/` | World editor SPA (separate root-level `editor.html` entry); its 3D viewport composes the real `Sim` + `Renderer`. |
| `src/world_api.ts` + `src/world_api/` | `IWorld`, the seam render/ui depend on: one facet interface per domain file under `src/world_api/`, re-aggregated by the barrel (see Architecture). |
| `src/main.ts` | Client entry; fixes the world seed. |
| `server/` | Authoritative game server: HTTP+WS, world loop, Postgres, auth, social, moderation. |
| `server/http/` | The REST request pipeline spine: table router, middleware onion, per-domain `RouteDef` tables, typed schemas, stable error codes. |
| `server/epic/` · `server/steam/` · `server/parse/` · `server/email/` | Store/platform glue, combat-parse ingest, transactional email; each has its own `CLAUDE.md`. |
| `headless/` + `python/` | RL env server (`env_server.ts`) + Python Gym bindings. |
| `bot/` | Discord bot (role sync, relay, activity feed; own `CLAUDE.md`). |
| `electron/` (+ `build/`) | Desktop (Steam) shell + packaging assets; see `docs/desktop-release.md`. |
| `android/` + `ios/` | Capacitor native shells (`npm run native:*`). |
| `tests/` | Vitest suite (subdirectory map in `tests/CLAUDE.md`). |
| `scripts/` | Asset/build/i18n/SFX tooling + browser E2E / screenshot scripts. |
| `patches/` + `data/` | pnpm-patched dependencies (Three.js is patched; check before bumping it) + checked-in map data. |
| `public/` · `docs/` | Static assets, **deployed verbatim to the live site** · design + PRD + ops docs. |
| `mediawiki/` + `deploy/` | Player-wiki container + production first-boot assets (see `DEPLOY.md`). |

Most directories above have their own `CLAUDE.md` with local conventions; read it when you work there.

## Commands
Install once per clone/worktree with **pnpm** (pinned via `packageManager` in
`package.json`; Corepack not required): `npm install -g pnpm@<the pinned version>`, then
`pnpm install --frozen-lockfile`. Same on macOS, Linux, and Windows; the shared store
makes multi-worktree installs cheap. Never commit `package-lock.json`. Full policy:
CONTRIBUTING.md. After install, `pnpm run <script>` and `npm run <script>` both work;
the nested `npm run` forms below are the package.json script names.

- `npm run dev`: Vite client on :5173 (proxies `/api`, `/admin/api`, `/ws` to :8787).
- `npm run server`: esbuild-bundle + run the authoritative server on :8787.
- `npm test`: Vitest. **Prefer a single file while iterating:** `npx vitest run tests/sim.test.ts`.
- `node scripts/gate_select.mjs`: **the pre-merge gate.** Same step list as `npm run gate`
  (nothing dropped) with one substitution: the full vitest run becomes ONE merged
  `vitest related` invocation (the always-run floor rides it as self-selecting seeds,
  same form as the CI shards). Roughly 3x faster; falls back to the full suite for any change it
  cannot reason about. See `docs/qa-gate.md`.
- `npm run gate`: the full CI-equivalent gate, still the deeper check (i18n gen + freshness, malware scan,
  changed-files biome, SFX conformance, full tests with bounded workers, the real-browser
  regression suite, `tsc`, all builds;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levy-street/world-of-claudecraft](https://github.com/levy-street/world-of-claudecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
