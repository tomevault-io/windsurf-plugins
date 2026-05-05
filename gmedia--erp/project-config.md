---
trigger: always_on
description: Aturan agent untuk project ERP
---


# Agent Rules

## 1. Stack Arsitektur

> **PENTING:** Project ini menggunakan arsitektur **Laravel API Backend + React Full SPA**. **TIDAK ADA Inertia.js.**

| Layer | Teknologi |
|-------|-----------|
| Backend API | Laravel (JSON API only) |
| Frontend SPA | React + TypeScript (standalone SPA, bukan Inertia) |
| Routing Frontend | `react-router-dom` → `resources/js/app-routes.tsx` |
| Routing Backend | `routes/api/*.php` (40+ modular files) |
| Data Fetching | React Query (`useCrudQuery`, `useCrudMutations`, custom hooks) |
| Meta Tags | `Helmet` via import `react-helmet-async` (alias lokal ke `resources/js/lib/react-helmet-async.tsx`) |
| Autentikasi | Sanctum Bearer Token (stateless, bukan session/cookies) |
| State Management | React Context (`auth-context.tsx`) + React Query cache |
| Lazy Loading | Route-level code splitting di `app-routes.tsx` |

### Routing Convention

- **Backend**: Buat file route baru di `routes/api/{module-slug}.php`. File di-include otomatis oleh `routes/api.php`.
- **Frontend**: Register lazy-loaded route baru di `resources/js/app-routes.tsx` dengan pattern `<Route path="/{module}" element={<P><Component /></P>} />`.
- **Web.php**: `routes/web.php` hanya berisi catch-all SPA route → **JANGAN tambah route di sini**.

### Anti-Pattern (DILARANG!)

- ❌ **JANGAN** import dari `@inertiajs/react` (`Head`, `Link`, `router`, `usePage`)
- ❌ **JANGAN** gunakan `Inertia\Inertia` atau `Inertia::render()` di backend
- ❌ **JANGAN** gunakan `actingAs($user)` di feature test → gunakan `Sanctum::actingAs($user)`
- ❌ **JANGAN** gunakan `assertInertia()` di test → gunakan `assertJson()`, `assertJsonStructure()`
- ❌ **JANGAN** tambah route di `routes/web.php`
- ❌ **JANGAN** re-add dependency upstream `react-helmet-async` ke `package.json`; tetap gunakan import `react-helmet-async` yang sudah dialias ke shim lokal React 19

## 2. Skills & MCP

1) **Skills**: selalu cek Skill yang tersedia; pakai jika relevan; jika tidak, tulis alasan singkat.
2) **MCP**: selalu cek MCP server/tools yang tersedia; pilih server yang paling spesifik untuk task; jika tidak dipakai, tulis alasan singkat.
	- `laravel-boost`: prioritaskan untuk schema DB, routes, logs, docs Laravel ecosystem, dan tinker di project ini.
	- `context7`: wajib untuk docs library/framework/SDK/API/CLI yang butuh syntax, konfigurasi, migration, upgrade notes, version-specific behavior, atau debugging berbasis package docs. Mulai dengan `mcp_context7_resolve-library-id`, lalu `mcp_context7_query-docs` dengan query spesifik.
	- `depwire`: wajib untuk dependency graph, blast radius, impact analysis, health score, dead code, security scan, dan refactor aman lintas file atau symbol. Sebelum rename/move/delete/split/merge, jalankan `mcp_depwire_get_file_context(...)`, `mcp_depwire_impact_analysis(...)`, atau `mcp_depwire_simulate_change(...)` sesuai scope.

## 3. Sail

Semua command runtime project wajib via `./vendor/bin/sail <command>` (artisan/composer/npm/test). Command git (status/log/diff/commit) boleh dijalankan langsung di host.

## 4. Testing

| Tipe | Auth Pattern | Assertions |
|------|-------------|------------|
| Feature Test (PHP) | `Sanctum::actingAs($user)` | `assertJson()`, `assertJsonStructure()`, `assertOk()` |
| Unit Test (PHP) | Tidak perlu auth | Pest assertions |
| E2E (Playwright) | Bearer token injection ke localStorage | `waitForResponse('/api/...')` |

## 5. Empty Wrapper Class

- Untuk class PHP yang sengaja kosong dan hanya mewarisi behavior dari base CRUD class seperti `SimpleCrudIndexRequest`, `SimpleCrudExportRequest`, `SimpleCrudResource`, atau `SimpleCrudCollection`, **JANGAN** biarkan body class benar-benar kosong.
- Selalu pakai body multiline dan tambahkan komentar intent berikut:

```php
class ExportBranchRequest extends SimpleCrudExportRequest
{
	// Intentionally empty. Behavior is inherited from the base class.
}
```

- Alasan: `./vendor/bin/sail bin duster fix` dapat mengompak class kosong menjadi one-line class, yang membuat intent kurang jelas dan dapat memicu issue style/Sonar.

## 6. Import & FQCN Hygiene

- Untuk kode PHP executable seperti controller, action, request, model, factory, migration, seeder, dan test, **WAJIB** import dependency di bagian atas file lalu gunakan short class name di body code.
- **JANGAN** gunakan fully-qualified class name dengan leading backslash di body code seperti `\App\Models\User::factory()`, `\Laravel\Sanctum\Sanctum::actingAs(...)`, `\Illuminate\Validation\Rule::unique(...)`, `\Carbon\Carbon::setTestNow(...)`, atau `\Illuminate\Support\Facades\Storage::disk(...)`.
- FQCN tetap boleh dipakai di PHPDoc, generic annotations, dan `::class` metadata jika memang dibutuhkan.
- Setelah generate atau refactor file PHP, verifikasi dengan `./vendor/bin/sail bin duster fix` agar issue TLint semacam ini tidak lolos.

## 7. MCP Routing, Security & Token Efficiency

- **Routing (WAJIB)**:
	- Gunakan `laravel-boost` untuk schema DB, routes, browser/app logs, dan docs Laravel ecosystem yang sesuai versi project ini.
	- Gunakan `context7` untuk docs package umum/non-Laravel atau saat butuh referensi API, syntax, atau konfigurasi yang version-specific dan up-to-date.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmedia/erp](https://github.com/gmedia/erp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
