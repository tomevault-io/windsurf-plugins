---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev             # start dev server
npm run build           # tsc + vite build
npm run lint            # ESLint (type-aware)
npm run lint:fix        # auto-fix lint errors
npm run format          # Prettier check
npm run format:fix      # Prettier write
npm run test            # Vitest (watch mode)
npm run test:coverage   # coverage report (≥85% required)
npm run docker:build    # build production image
npm run docker:run      # run container at http://localhost:8080
```

Run a single test file: `npx vitest run src/utils/share.test.ts`

Before opening any PR, all four must pass locally: `npm run test && npm run lint && npm run build && npm run test:e2e`

Never push directly to `main`. All changes must go through a pull request. A `pre-push` git hook in `.githooks/` enforces this — activated automatically via the `prepare` npm script on `npm install`.

## Architecture

### State flow

`useQRGenerator` owns all QR config state. Input fields update "pending" state (e.g. `inputFgColor`); clicking **Generate** snapshots them into `config`, which drives the `qrcode.react` preview. Downloads use the headless `qrcode` library against the pending input state — not the DOM.

**Capacity guard.** `src/utils/qrCapacity.ts` holds the version-40 byte-mode capacity per EC level and `getCapacityStatus` (measured in UTF-8 bytes). `useQRGenerator` treats content past that capacity like its length validation: `isBlocked` clears the live preview and disables downloads, so `qrcode.create` never sees an unencodable value. This matters because `getMatrixSize` (`qrShapeRenderer.ts`) runs at render and `qrcode.create` throws on over-capacity input — reachable at Q/H levels under the 2000-char input cap — so it catches and falls back to the version-1 size rather than crashing the generator. The `CapacityCounter` under the Text/URL field (text mode only) is the visible signal: `used / max`, amber near the limit, red over it.

### Context providers (wired in `src/main.tsx`)

- `ThemeProvider` — reads/writes `localStorage`, toggles `.dark` on `<html>`, exposes `useThemeContext()`
- `LocaleProvider` — reads/writes `localStorage`, syncs `document.documentElement.lang`, exposes `useLocaleContext()` with `translate(key)` and a locale-aware `seo` object. Locales are additive: register a new JSON in `src/data/i18n/index.ts` and `SupportedLocale`, the `TranslationKey` union, and the registry all widen from it (no type edits). `LocaleMetadata.switchTo` is a `Partial` record, so a new locale needs no edits to existing locale files; `getCopy()` falls back to the default locale for any missing key. The navbar `LanguageToggle` is a native `<select>` dropdown that lists every locale in `localeCodes` by its `locale.name`, so it picks up new locales automatically.

### Directory conventions

| Path | Purpose |
|---|---|
| `src/components/common/` | Reusable primitives (Button, Input, Textarea, Card, etc.) |
| `src/components/feature/qr/` | QR-specific views |
| `src/hooks/` | Stateful hooks and context providers |
| `src/utils/` | Pure helpers — every file here requires a corresponding test |
| `src/data/` | Static config and i18n JSON (`en.json`, `es.json`) |
| `src/types/` | Shared TypeScript types |

### Styling

Tailwind CSS v4 via `@tailwindcss/vite`. Entry point is `src/index.css`. Use semantic design tokens (CSS custom properties) for all colors — never hard-code hex values in component classes. The `dark` class on `<html>` drives dark-mode variants.

### Views

Three top-level views toggle via a `PillGroup` in `src/App.tsx`: **Generate** (`QRGenerator`), **Batch** (`BatchGenerator`), **Scan** (`QRScanner`). Generate stays mounted (`hidden`); Batch and Scan mount on demand.

### Share / export

`useQRShare` handles the share button: tries `navigator.share` with files → `ClipboardItem` image write → download fallback. `useExportState` + `src/utils/export/` drive the hi-res export modal (PNG / SVG / PDF via jspdf).

Headless rendering (no DOM preview) is shared: `renderQrPngBlob` (`src/utils/export/pngRenderer.ts`) for PNG, `exportSvg` / `exportPdf` for the rest. The single-QR download path and batch generation both go through these.

### Batch generation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyaethu-aung/qr-generator](https://github.com/pyaethu-aung/qr-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
