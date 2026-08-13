---
trigger: always_on
description: Things that aren't obvious from the code or the Joplin docs. If you're about to release, start with **Release pipeline**.
---

# Joplin Explorer — Project Notes

Things that aren't obvious from the code or the Joplin docs. If you're about to release, start with **Release pipeline**.

---

## Workflow with the user (IMPORTANT)

- **Do not publish eagerly.** Default loop while iterating: edit → `npx webpack --env production` → `node scripts/pack-jpl.js` → tell the user it's in `publish/`, and let them test. Only bump the version, tag, GitHub-release and `npm publish` **when the user says 发布 / "release"**.
- The user tests via **Development plugins** pointed at the repo root (`D:\repos\Joplin-explorer`), so a rebuild + Joplin restart is enough — no "install from file" needed. Note a dev-loaded plugin conflicts with the store-installed copy (same id); one must be disabled.
- Commit locally during development; push together with the release.

## Release pipeline (CRITICAL)

`publish/plugin.jpl` is a **gzipped tar archive** containing its own copy of `manifest.json`. Joplin uses the **outer** manifest (or the npm registry metadata) to decide whether an update is available, but reads the version actually installed from the **inner** manifest inside the `.jpl`.

If these disagree, Joplin gets stuck in an update loop: it sees a newer outer version, downloads and installs the `.jpl`, the inner manifest still reports the old version, on next restart Joplin prompts again — forever.

**Before every release:**

1. Bump version in BOTH `src/manifest.json` AND `package.json`.
2. Run `npm run dist` — this MUST regenerate `publish/plugin.jpl` so the inner manifest matches.
3. Verify the inner manifest:
   ```
   tar -xzf publish/plugin.jpl -C /tmp/check && cat /tmp/check/manifest.json
   ```
   Inner version MUST equal `src/manifest.json`'s version.
4. Only then `npm publish` and upload `publish/plugin.jpl` to the GitHub release.

`publish/plugin.jpl` is NOT tracked in git (`.gitignore` covers the whole `publish/` directory). `scripts/pack-jpl.js` is the only thing that rebuilds it. **Do not skip it, do not hand-edit the .jpl.**

Between v1.2.0 and v1.2.3 nothing refreshed `publish/plugin.jpl`. The outer manifest advanced; the inner one was frozen at v1.2.2. Users got trapped in the update loop. v1.2.4 fixed the build. Do not let this recur.

### Semver

`1.2.1.1` is **not** a valid semver — npm will reject it. Use `1.2.2`. Patch bumps only have three components.

### npm publishing

- npm Granular Access Tokens default to **7-day expiration**. Set longer if you need.
- For security-key users, the token MUST have **"Bypass 2FA when publishing"** checked. Without it, `npm publish` errors with `EOTP` and a security-key user cannot satisfy the prompt.
- `npm unpublish` is only allowed within 24h. After that, the broken version stays forever — bump and move on, optionally `npm deprecate`.
- Default to one-shot tokens: generate → publish → delete.
- The token lives in `D:\repos\.npm-publish-token.txt` (one level above the repos, so it is never git-tracked). That file is a **memo**, not a bare token — extract the value with a regex (`npm_[A-Za-z0-9]+`) before writing `.npmrc`. Dumping the whole file in makes `npm publish` fail with a misleading **404** (npm masks auth failures as 404).

---

## Plugin runtime context

- The plugin entry (`src/index.ts`) runs in **Node.js context** inside Joplin's plugin host — `fs`, `path`, `crypto`, `process`, etc. are all available.
- `joplin` is injected as a **global** at runtime. Use `declare const joplin: any;` at the top of `src/index.ts`. **Do NOT add `import joplin from 'api'`** — it breaks the build with "Cannot find module 'api'". The `api/` stub was removed in commit c84a518.
- The webview (`src/webview/panel.js`) runs in a sandboxed browser-like context. It cannot import npm packages — it's loaded as a plain script via `copy-webpack-plugin`. Communicate with the host via `webviewApi.postMessage` / `joplin.views.panels.postMessage`.
- `process.env.HOME` is not set on Windows (it's `USERPROFILE`). Code that expands `~/` paths must handle this — currently `normalizeLocalIconPath` only expands on POSIX. Acceptable since `~/` is a Unix convention.

## Joplin API quirks

- `SettingItemType` numeric values: `Int=1`, `String=2`, `Bool=3`, `Array=4`, `Object=5`, `Button=6`.
- Settings registration must use `joplin.settings.registerSettings({...})` (**plural**). The singular `registerSetting()` is deprecated and silently fails — manifest as the panel hanging at "Loading..." with no error.
- `joplin.workspace.onNoteChange(handler)` fires very frequently while a user is typing in the editor. **Always debounce** (we use 600 ms) — direct refresh per fire is unusable.
- `joplin.settings.onChange` event payload has `event.keys: string[]`.
- `joplin.data.get(['search'], { query, ... })` uses **FTS5 with tokenization** — it does **not** substring-match. Querying `"8121R"` will not find `"KY8121R"`. We combine FTS results with a local case-insensitive title substring scan against `allNotesCache` to compensate. Don't lose that fallback.
- `joplin.plugins.dataDir()` returns a per-plugin persistent directory. Use it for caches like icon files; never write under the install location.

## Theming (CSS variables)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lim0513/joplin-explorer](https://github.com/lim0513/joplin-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
