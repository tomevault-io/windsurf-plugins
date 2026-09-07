---
trigger: always_on
description: Fork upstream `chuanqisun/PDFCraft` (AGPL-3.0) zaadaptowany jako **AIwBiznesie PDF Studio** — privacy-first PDF toolkit działający w 100% w przeglądarce (WASM, pdfjs-dist, pdf-lib, Pyodide dla DOCX/PPTX).
---

# PDFCraft fork (tools-PDFCraftTool) — CLAUDE.md

## O projekcie

Fork upstream `chuanqisun/PDFCraft` (AGPL-3.0) zaadaptowany jako **AIwBiznesie PDF Studio** — privacy-first PDF toolkit działający w 100% w przeglądarce (WASM, pdfjs-dist, pdf-lib, Pyodide dla DOCX/PPTX).

**Strategia produktu** (decyzja 06.05 + 07.05):
- **Lead magnet** dla AIwBiznesie (no-friction, brand-controlled URL)
- **Internal tool** dla Dariusza + zespołu
- **Klienci premium** — opcjonalnie udostępniany jako gratis dla VIP
- **NIE płatny SaaS** — AGPL viral OK dla tego modelu

## Tech stack

- Next.js 15 + Turbopack, App Router, **`output: 'export'`** (static SSG, brak server runtime)
- React 19 + TypeScript
- Tailwind CSS + custom UI components (NIE shadcn)
- next-intl (14 języków, primary: pl, fallback: en)
- pdfjs-dist 4.8 + pdf-lib 1.17 + Pyodide (dla DOCX/PPTX export)
- Zustand 5 (`studioStore`)
- @radix-ui/react-dropdown-menu (menu bar, context menus)
- @dnd-kit/{core,sortable,utilities} (page reorder)
- @supabase/supabase-js 2.105 (auth + DB)

## Architektura

### Tools-first vs Studio mode
**Dwa równoległe tryby:**
- `/[locale]/tools/[tool]/` — klasyczny tool-first dispatcher (97 narzędzi, każde własna strona, SEO-friendly deep linki)
- `/[locale]/studio` — Acrobat-inspired 3-column layout (PagesPanel + PdfViewer + ToolsPanel) — **rebuild UX nad tymi samymi 97 ToolComponents**

Tools logic w `src/lib/pdf/processors/<tool>.ts` jako pure functions, UI w `src/components/tools/<slug>/<XxxTool>.tsx`. **CZYSTA separacja** umożliwiła rebuild UX (Studio) w 8-12h zamiast spodziewanych 33-67h.

### Auth (Supabase)
- Project **PDF Studio** (`wvjoeyulugbpovhjboag`) w eu-central-1 Frankfurt
- Browser-only client (`src/lib/supabase/client.ts`) — NIE używamy @supabase/ssr (output:export)
- AuthContext + useAuth hook
- Schema: `user_preferences`, `recent_documents`, `_keepalive` (3 tabele) + RLS na każdej + auto-create user_preferences trigger on `auth.users` insert
- Site URL config: production + preview wildcards + localhost (PATCH przez Management API)

### Studio Mode komponenty
- `StudioLayout` — root flex container
- `StudioHeader` — open/clear/export/theme/avatar (TODO P0)
- `StudioMenuBar` — Plik/Widok/Narzędzia/Pomoc + skróty
- `StudioFooter` — file metadata
- `PagesPanel` — thumbnail per strona + DnD reorder + delete
- `PdfViewer` + `ViewerToolbar` — pdfjs render + page nav + zoom
- `ToolsPanel` — search + 7 batchowych tools w prawym panelu
- `LoginModal` — signin/signup/forgot-password z eye toggle

### State management
- `useStudioStore` (Zustand) — files, currentTool, currentPage, zoom, sidebar widths, recent
- File state: `data: Uint8Array | null` (lazy populated po pierwszym load) + `version: number` (inkrementowany przy mutation, trigger re-render w PdfViewer/PagesPanel)

## Konwencje

### TypeScript
- Strict mode, brak `any`
- Typy z lucide-react: `LucideIcon` (NIE `React.ComponentType`)
- Zustand selectors: **primitives** (id, version) w useEffect deps, NIE objects
- Blob types: `data as BlobPart` cast (TS lib DOM strict)

### Style
- HSL CSS variables (`--color-primary`, `--color-card`, `--color-border`)
- `bg-[hsl(var(--color-card))]` pattern (NIE shadcn semantic classes)
- Theme: `<html class="dark">` toggle + localStorage

### i18n
- Top-level namespace `studio` w `messages/{pl,en,...}.json`
- `useTranslations('studio')` w komponentach
- Fallback do `en` gdy klucz nie istnieje w innych locale

### Commits
- Conventional commits: `feat(studio):`, `fix:`, `chore(vercel):`
- HEREDOC z `Co-Authored-By: Claude Opus 4.7 (1M context)` ostatnia linia
- `git add <konkretne_pliki>` — NIE `-A` (lekcja PM 2026-04-27)

## Lessons Learned

- [2026-05-07] KONTEKST: Cloudflare 1010 dla Python urllib user-agent przy `POST /v1/projects/.../database/query` Supabase Management API. Fix: użyj `curl` z `-H "User-Agent: ..."` zamiast Python urllib. Reguła: dla Supabase Management API zawsze przez curl, nie urllib. Plus payload przez `-d @plik.json` żeby uniknąć escaping issues w shell.
- [2026-05-07] KONTEKST: Infinite re-render loop w PdfViewer gdy useEffect deps zawierał `currentFile` (object derived przez Zustand selektor). Każda mutacja store → nowa identity obiektu → useEffect re-fire → load → setPageCount → store update → loop. Fix: deps to **primitives** (`currentFileId`, `fileVersion`), pobieranie obiektu przez `useStudioStore.getState()` w callback. Reguła: nigdy nie używaj selektora obiektu z Zustand jako useEffect dep.
- [2026-05-07] KONTEKST: PDFCraft fork ma czystą separację `[tool]/page.tsx` (dispatcher) + 97 osobnych ToolComponents w `src/components/tools/<slug>/`. Logika PDF w `src/lib/pdf/processors/` jako pure functions. Audyt architektury PRZED estymatą umożliwił rebuild UX (Acrobat-style Studio) w 8-12h zamiast spodziewanych 33-67h. Reguła: ZAWSZE audyt architektury (separacja UI/logic) przed estymatą rebuildu.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DariuszCiesielski/tools-PDFCraftTool](https://github.com/DariuszCiesielski/tools-PDFCraftTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
