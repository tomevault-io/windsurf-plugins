---
trigger: always_on
description: - **Plugin**: Vizier, a local AI agent for Obsidian powered by Ollama.
---

# Vizier — agent guide

## Project overview

- **Plugin**: Vizier, a local AI agent for Obsidian powered by Ollama.
- Target: Obsidian Community Plugin (TypeScript + React → bundled JavaScript).
- Entry point: `src/main.ts` compiled to `main.js` and loaded by Obsidian.
- Required release artifacts: `main.js`, `manifest.json`, `styles.css`.

## Key features & architecture

| Feature | Entry point |
|---|---|
| Chat UI | `src/ui/ChatView.tsx` → `src/ui/ChatApp.tsx` |
| Slash commands (`/write`, `/find`, `/clip`, `/clip long`, `/clip learn`, `/summarize`, `/read`, `/handwriting`) | `src/commands/slashCommands.ts` |
| Settings | `src/settings.ts` |
| Vizier server setup modal + model download modal | `src/ui/ServerSetupModal.ts` |
| AI prompts | `src/prompts.ts` |
| Ollama API wrapper | `src/utils/ollama.ts` |
| Map-reduce summarizer | `src/utils/chunking.ts` |

**External services used:**
- `r.jina.ai` — article fetching (converts any URL to clean markdown; free, no auth)
- `vizier_server.py` — local Python HTTP server; provides YouTube transcripts (`/transcript`) and handwriting OCR (`/ocr`) on port 11435. Each route loads its dependencies lazily so a missing package only breaks that route, not the whole server.

## Chat history

Persisted in `localStorage` under key `vizier-chat-history` (last 60 messages). Cleared when the user clicks "Clear".

## Environment & tooling

- Node.js: use current LTS (Node 18+ recommended).
- **Package manager: npm** (required for this sample - `package.json` defines npm scripts and dependencies).
- **Bundler: esbuild** (required for this sample - `esbuild.config.mjs` and build scripts depend on it). Alternative bundlers like Rollup or webpack are acceptable for other projects if they bundle all external dependencies into `main.js`.
- Types: `obsidian` type definitions.

**Note**: This sample project has specific technical dependencies on npm and esbuild. If you're creating a plugin from scratch, you can choose different tools, but you'll need to replace the build configuration accordingly.

### Install

```bash
npm install --legacy-peer-deps
```

### Dev (watch)

```bash
npm run dev
```

### Production build

```bash
npm run build
```

## Linting

```bash
npm run lint        # run eslint across all source files
```

The project uses `eslint-plugin-obsidianmd` which enforces Obsidian-specific rules (sentence-case UI text, no inline `style.display` assignments, no plugin name in command names). Fix all errors before submitting a PR. Use `// eslint-disable-next-line <rule>` only when a rule fires on a legitimate exception (e.g. a proper noun the linter misreads as a casing violation).

## File & folder conventions

- **Organize code into multiple files**: Split functionality across separate modules rather than putting everything in `main.ts`.
- Source lives in `src/`. Keep `main.ts` small and focused on plugin lifecycle (loading, unloading, registering commands).
- **Example file structure**:
  ```
  src/
    main.ts           # Plugin entry point, lifecycle management
    settings.ts       # Settings interface and defaults
    commands/         # Command implementations
      command1.ts
      command2.ts
    ui/              # UI components, modals, views
      modal.ts
      view.ts
    utils/           # Utility functions, helpers
      helpers.ts
      constants.ts
    types.ts         # TypeScript interfaces and types
  ```
- **Do not commit build artifacts**: Never commit `node_modules/`, `main.js`, or other generated files to version control.
- Keep the plugin small. Avoid large dependencies. Prefer browser-compatible packages.
- Generated output should be placed at the plugin root or `dist/` depending on your build setup. Release artifacts must end up at the top level of the plugin folder in the vault (`main.js`, `manifest.json`, `styles.css`).

## Manifest rules (`manifest.json`)

- Must include (non-exhaustive):  
  - `id` (plugin ID; for local dev it should match the folder name)  
  - `name`  
  - `version` (Semantic Versioning `x.y.z`)  
  - `minAppVersion`  
  - `description`  
  - `isDesktopOnly` (boolean)  
  - Optional: `author`, `authorUrl`, `fundingUrl` (string or map)
- Never change `id` after release. Treat it as stable API.
- Keep `minAppVersion` accurate when using newer APIs.
- Canonical requirements are coded here: https://github.com/obsidianmd/obsidian-releases/blob/master/.github/workflows/validate-plugin-entry.yml

## Testing

- Manual install for testing: copy `main.js`, `manifest.json`, `styles.css` (if any) to:
  ```
  <Vault>/.obsidian/plugins/<plugin-id>/
  ```
- Reload Obsidian and enable the plugin in **Settings → Community plugins**.

## Commands & settings

- Any user-facing commands should be added via `this.addCommand(...)`.
- If the plugin has configuration, provide a settings tab and sensible defaults.
- Persist settings using `this.loadData()` / `this.saveData()`.
- Use stable command IDs; avoid renaming once released.

## Versioning & releases

- Bump `version` in `manifest.json` (SemVer) and update `versions.json` to map plugin version → minimum app version.
- Create a GitHub release whose tag exactly matches `manifest.json`'s `version`. Do not use a leading `v`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AADaoud/vizier](https://github.com/AADaoud/vizier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
