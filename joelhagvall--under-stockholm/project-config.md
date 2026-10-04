---
trigger: always_on
description: Browser first-person game in a stylized Stockholm metro. Vite + TypeScript + three.js + Rapier. See README.md for the pitch, docs/FEATURES.md for the full list and structure, docs/GAMEPLAY.md for controls.
---

# Under Stockholm: agent guide

Browser first-person game in a stylized Stockholm metro. Vite + TypeScript + three.js + Rapier. See README.md for the pitch, docs/FEATURES.md for the full list and structure, docs/GAMEPLAY.md for controls.

## Rules

- Use Bun for everything (`bun install`, `bun run dev`, `bunx`). Never npm, yarn or pnpm.
- Code, comments and docs in English. The world is always Swedish: signs, boards, announcements, anything spoken and quoted speech. Everything said to the player comes in Swedish and English: menus, help, settings, the `E ·` prompts, captions of what happens (with any quoted speech left in Swedish inside the English sentence), the pause menu's status and the discovery book: `src/game/i18n/text.ts` holds the current language (`src/lang.ts` picks it: the player's choice, else Swedish), `en.json` overrides only those keys, and modules that draw the world import `sv.json` directly. The landing page exists in both languages, `index.html` and `en/index.html`, linked with `hreflang`: change them together (`tests/i18n.test.ts` checks they match).
- Never use em-dashes in code, comments, docs or copy. Use a comma, colon or period, and `|` as a title separator.
- Do not commit unless explicitly asked. No `Co-Authored-By` or other trailers.
- **The recorded announcements never go public.** The clips of the Utrop library in `public/audio/` stay local, ignored by git (only `catalog.json`, their lengths and checksums, is committed): SL does not own the voice and declined to approve it (September 2026), and the rights holder is unknown. The game goes live without them, and videos and copy must not use them or claim the real voice: a plain `bun run deploy` builds with `RECORDINGS=0` (`__RECORDINGS__` false: every call is spoken by the browser's voice, the chime and the door warning are synthesized in `audio.ts`, and no clip is built or uploaded, which the deploy checks). The sounds of people in `public/audio/sfx/` are free and go out. Never deploy with `--with-recordings` unless the user says the recordings have been cleared. `--landing` still deploys the landing page alone (`LANDING_ONLY=1`).
- Deploy only with `bun run deploy` (`scripts/deploy.ts`, to https://understockholm.com/), and only when asked: it refuses a dirty or unpushed tree, runs the whole quality gate and uploads only if every gate passed. It uploads the Worker too (`worker/`, `wrangler.jsonc`): the static build plus the hub, a Durable Object that serves the relay's paths on the same origin in production (docs/DRIFT.md). Never run `wrangler deploy` or any other deploy command directly, and never push with `--no-verify` or `SKIP_CHECK`: a hook blocks them (`scripts/hooks/before-bash.ts`). If the user wants to go around the gate, they run the command themselves.

## Architecture notes

- **Coordinates.** Meters, y up, rail top at y = 0. The line runs along +x (index 0, Kungsträdgården, at x = 0, heading west toward +x). Trains keep left, as on SL's metro: track 1 is at z = -`TRACK_Z` and runs toward +x, track 2 at z = +`TRACK_Z` runs back (`trackSide` in `layout.ts`). All dimensions live in `src/game/layout.ts`: change them there, never hardcode numbers in generators.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhagvall/under-stockholm](https://github.com/joelhagvall/under-stockholm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
