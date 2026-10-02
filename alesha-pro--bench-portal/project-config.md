---
trigger: always_on
description: Bench Portal is a static catalog of playable AI-built games, 3D showcases and interactive visual experiments. The interface is in English; communicate with Alexey in Russian unless asked otherwise.
---

# Bench Portal: guide for agents

Bench Portal is a static catalog of playable AI-built games, 3D showcases and interactive visual experiments. The interface is in English; communicate with Alexey in Russian unless asked otherwise.

## Project and commands

- Production: https://bench-portal.pages.dev
- Mirror: https://alesha-pro.github.io/bench-portal/
- Git remote: https://github.com/alesha-pro/bench-portal.git
- Production branch: `main`.
- `npm test` runs catalog/manifest tests with Node's built-in test runner.
- `npm run build` validates every manifest and copies the site into `dist/`.
- `npm run serve` serves `dist/` at http://127.0.0.1:4176. Build first. Rebuild after edits; this server does not provide hot reload.
- `npm run deploy:cloudflare` builds and deploys `dist/` with Wrangler to the existing `bench-portal` Pages project on branch `main`.

The catalog requires no npm dependencies or frontend framework. Node 22 is used in CI; Python 3 powers the local preview. Wrangler is invoked through `npx wrangler@4` and uses the user's existing Cloudflare authentication.

## Files and ownership

- `index.html`, `styles.css`: homepage shell and responsive styling.
- `src/catalog.js`: DOM rendering, filter controls, loading/error states and browser history.
- `src/catalog-data.js`: shared categories, manifest validation, filtering, sorting and URL parsing. Keep this module free of browser globals so Node can test and import it.
- `games/<slug>/`: one self-contained, playable static build per directory.
- `games/<slug>/game.json`: its catalog metadata, the source of truth for the card.
- `scripts/build.mjs`: static build and filesystem validation.
- `tests/catalog.test.mjs`: tests against the actual collection and edge cases.
- `.github/workflows/pages.yml`: GitHub Pages deployment on pushes to `main`.
- `dist/` and `dist/games.json`: generated output. Never edit or commit them.

Inspect `git status` before work. Preserve unrelated changes and existing game URLs. Changes to the homepage should not modify game code or assets. Only change a game's implementation when the task includes that game.

## Add a build

1. Pick a unique, lowercase, URL-safe slug. Use letters, numbers and hyphens; existing slugs may also contain dots. Do not rename published directories casually: their URLs are shared publicly.
2. Copy the production static build into `games/<slug>/`. It must contain `index.html` and all required local assets. Never copy `node_modules/`, caches, secrets or an entire unrelated repository.
3. Make asset URLs relative to the game directory. For Vite, use `vite build --base=./`. Root-relative `/assets/...` breaks the GitHub Pages mirror and nested game routes.
4. Create `game.json` using the schema below. Keep labels and descriptions factual. Do not invent benchmark results, model authorship, dates, scores or completion claims.
5. If a cover is available, put it inside the game folder and set `cover` to its relative path, such as `cover.webp`. Otherwise omit it; the catalog supplies category artwork. Do not invent a screenshot of gameplay that was not captured.
6. Run `npm test` and `npm run build`.
7. Preview the catalog card, its category/model filters, and the actual game at `/games/<slug>/`. Check asset loading and the console. A passing static build does not prove a game plays correctly.

Example manifest:

```json
{
  "slug": "example-arena",
  "title": "Example Arena",
  "category": "shooters",
  "model": "GPT-6 Astra",
  "version": "September build",
  "summary": "An arena shooter with escalating enemy waves.",
  "description": "Describe the actual gameplay, notable features and relevant creation context.",
  "accent": "#ff703e",
  "tags": ["FPS", "Three.js", "WebGL"],
  "cover": "cover.webp"
}
```

Required: `slug`, non-empty `title`, valid `category`, and `model` (non-empty string or `null`). Optional: `version`, `summary`, `description`, `accent`, `tags`, `cover`.

`model` means the model that **built** the project. It is not the model being visualized or benchmarked. For example, Ox Alpha visualizes Laguna routing, but its builder is unspecified, so its `model` is `null`. Use `null` rather than guessing. `version` is a build label and is not used as a substitute for authorship.

Use the same model spelling as existing manifests, so a model does not split into duplicate filters. Current names include `GPT-6 Astra`, `GPT-5.6 Luna`, `Claude Fable 5.1`, `DeepSeek V4.1`, `GLM-5.3`, `GLM-5.3 Flash`, `Qwen3.8-27B`, `Qwen3.8-Flash-Next`, and `Qwen3.8-Max`. A genuinely new model name automatically becomes a filter option.

`summary` is a short card description. The complete `description` remains accessible through “Build notes.” Use tags consistently (for example `Three.js`, `WebGL`, `GLSL`, `Blender`, `Voxel`). Tags are searched and can be filtered case-insensitively. `accent` must be a 3- or 6-digit hex color. Covers must resolve to a real file inside the game folder; absolute URLs and traversal paths are rejected.

## Categories

Choose exactly one primary category for each build:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alesha-pro/bench-portal](https://github.com/alesha-pro/bench-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
