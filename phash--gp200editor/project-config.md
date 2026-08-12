---
trigger: always_on
description: Inoffizieller Browser-Editor für Valeton GP-200 Gitarren-Multi-Effektpedal Preset-Dateien (`.prst`).
---

# Preset Forge — GP-200 Editor – CLAUDE.md

Inoffizieller Browser-Editor für Valeton GP-200 Gitarren-Multi-Effektpedal Preset-Dateien (`.prst`).
Live USB-MIDI Editing, Preset-Galerie, Community-Sharing.

## Projekt-Überblick

- **Name:** Preset Forge
- **Domain:** https://preset-forge.com
- **Zweck:** `.prst` Preset-Dateien im Browser laden, bearbeiten, speichern, teilen, live per USB-MIDI ans Gerät senden
- **GitHub:** https://github.com/phash/gp200editor
- **Stack:** Next.js 15 App Router · React 19 · TypeScript strict · Tailwind CSS · Prisma 5 · PostgreSQL 16 · Lucia v3 · Garage S3 · next-intl 4 (DE/EN/ES/FR/IT/PT/pt-BR — 7 Locales)
- **Tests:** Vitest · Playwright + @axe-core/playwright (E2E + A11y)
- **Ziel:** WCAG 2.1 AA
- **UI:** Dark pedalboard theme (JetBrains Mono + DM Sans, Amber-Akzente, LED-Style Buttons)

---

## Entwicklung

```bash
npm install --legacy-peer-deps   # IMMER --legacy-peer-deps (npm 11.x vs Docker npm 10.x)
npm run dev                      # http://localhost:3000
npm run test                     # Vitest Unit-Tests
npm run test:e2e                 # Playwright E2E (App muss laufen + Garage + DB)
npm run lint                     # ESLint
npm run build                    # Production Build
npx next start -p 3123           # Prod-Build lokal servieren — Header, Canonicals, Redirects prüfen ohne Docker
npm run ci                       # Lokale CI: lint + typecheck + test + build (ersetzt GH Actions)
bash scripts/local-ci.sh lint typecheck   # Einzelne Stages
npx vitest run path/test.ts -t "pattern"  # Single-File + Test-Name-Grep (schnelles TDD-Iterate)
```

**Prod-Deploy:** `ssh musikersuche@musikersuche.org` → `cd /opt/gp200editor && bash scripts/deploy-update.sh` — Script macht `git pull` auf **master** (prod ist dort ausgecheckt), PR also vorher mergen
**Prod-SQL:** `source .env.prod && docker compose -f docker-compose.prod.yml -f docker-compose.caddy.yml exec -T postgres psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"`

---

## Architektur

```
src/
├── core/                    # Pure TypeScript, framework-unabhängig
│   ├── types.ts             # Zod-Schemas: GP200PresetSchema, EffectSlotSchema
│   ├── BinaryParser.ts      # DataView-basierter Reader (uint8/16/32, float32, ASCII, bytes)
│   ├── BufferGenerator.ts   # DataView-basierter Writer (uint8/16/32, float32, ASCII)
│   ├── PRSTDecoder.ts       # .prst → GP200Preset (1224 Bytes, echtes Format)
│   ├── PRSTEncoder.ts       # GP200Preset → .prst (1224 Bytes, echtes Format)
│   ├── effectNames.ts       # 305 Effekt-ID→Name Mappings + MODULE_COLORS (aus algorithm.xml)
│   ├── effectParams.ts      # Parameter-Definitionen pro Effekt (Knob/Slider/Switch/Combox)
│   ├── effectDescriptions.ts # 255 Effekt→Beschreibung Mappings (original Pedal/Amp Names)
│   └── HLXConverter.ts      # Line6 HX Stomp .hlx (JSON) → GP200Preset Konvertierung
│
├── hooks/
│   ├── usePreset.ts         # React-State: loadPreset, setPatchName, toggleEffect,
│   │                        #   changeEffect (applies default params), reorderEffects, setParam, reset
│   └── useTimelinePlayer.ts # rAF-basierter Timer für Playlist Cue Points
│
├── components/              # Navbar, FileUpload, EffectSlot, EffectParams, AmpHeadPanel,
│                            #   ControllerPanel, AdminDashboard, AdminActions, CuePointTable,
│                            #   ConfirmDialog, WarnDialog, LocaleSwitcher, Footer,
│                            #   YouTubeEmbed, audio/ (AudioPlayer + Provider, 30s Preset-Snippets)
│
├── lib/                     # auth.ts (Lucia v3), prisma.ts, session.ts, admin.ts,
│                            #   errorLog.ts, email.ts, storage.ts, validators.ts
│
├── app/[locale]/            # layout, page (Home), editor, auth/, admin, profile/, presets/, share/,
│                            #   playlists/, gallery/, changelog/, help/, legal/
├── app/api/                 # auth/, admin/, profile/, avatar/, presets/, share/, gallery
├── i18n/                    # routing.ts, request.ts
└── middleware.ts            # next-intl + Auth-Guards
```

---

## i18n-Konventionen

- `routing.ts` exportiert typisierte Navigation: `import { Link, useRouter, usePathname } from '@/i18n/routing'`
- Nie `next/link` oder `next/navigation` direkt importieren (Ausnahme: `redirect()` in Server Components)
- Alle UI-Strings über `useTranslations()` / `getTranslations()` (kein Hardcoding)
- Translations in `messages/{de,en,es,fr,it,pt,pt-BR}.json` — 7 Locales, Key-Parität per Unit-Test erzwungen
- `messages-parity` prüft nur Locales gegeneinander — ein überall fehlender Key sähe wie perfekte Parität aus; `messages-usage` schließt die Lücke und prüft jeden literalen `t('…')`-Aufruf in `src/` gegen `en.json`
- `localePrefix: 'as-needed'` — EN läuft unpräfigiert (`/help`), `/en/help` 307t dorthin; alle anderen Locales behalten ihr Präfix
- Public URLs immer über `localeUrl(locale, path)` aus `@/lib/hreflang` — kennt die EN-Ausnahme; `tests/unit/canonical-urls.test.ts` erzwingt es
- Hreflang: `src/lib/hreflang.ts` mit `buildAlternates(path, locale)` — nie inline schreiben

---

## USB-Gerätekommunikation

- **Status:** SysEx-Protokoll reverse-engineered ✓, Web MIDI Live-Editing hardware-verifiziert ✓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phash/gp200editor](https://github.com/phash/gp200editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
