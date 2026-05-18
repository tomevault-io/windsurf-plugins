---
trigger: always_on
description: Panduan utama untuk AI coding agent yang bekerja di repository `EduCore`.
---

# AGENTS.md

Panduan utama untuk AI coding agent yang bekerja di repository `EduCore`.
File ini harus dianggap sebagai **instruksi operasional utama** untuk chat baru agar agent memahami arsitektur, batasan runtime, standar kualitas, dan urutan eksekusi kerja.

---

## 1. Project Identity

**EduCore** adalah aplikasi manajemen sekolah hybrid:
- **Desktop**: Tauri v2
- **Web**: Next.js 16 App Router + React 19
- **Bahasa**: TypeScript strict
- **Package manager**: Bun
- **ORM**: Drizzle ORM
- **Desktop local DB**: SQLite
- **Cloud shared DB**: Turso / libSQL

Target produk:
- Bisa dipakai di **PC/laptop** via desktop app
- Bisa dipakai di **browser** tanpa install
- Bisa diakses dari **HP** via web responsif / PWA
- Mendukung **offline-first di desktop**
- Mendukung **multi-user dengan data sekolah yang sama**

---

## 2. Current Architecture Truth

Instruksi di file ini harus mengikuti kondisi aktual repo, bukan asumsi lama.

### Shared model
- **Cloud DB (Turso)** = shared collaboration layer antar user/device
- **Local SQLite desktop** = operational source of truth saat desktop offline
- **Web runtime** = online-first, akses data melalui backend web
- **Desktop runtime** = local-first, akses data melalui local runtime bridge / service lokal
- **Sync engine** = push/pull delta antara desktop local DB dan cloud

### Source of truth policy
- **Global shared truth**: Turso
- **Local operational truth**: SQLite per desktop device

Jangan memodelkan sistem sebagai “semua device memakai satu DB lokal yang sama”.
Sistem yang benar adalah **replicated local-first system**.

---

## 3. Runtime Boundary Policy

### Desktop runtime
Desktop harus:
- usable tanpa internet
- membaca/menulis ke SQLite lokal
- tidak bergantung pada `/api/*` web untuk flow inti
- tidak bergantung pada session provider web untuk flow inti
- hanya sync ke cloud saat online

Desktop tidak boleh:
- memerlukan Next server untuk CRUD inti
- menarik dependency web/server-only ke client bundle desktop
- mengandalkan secret cloud di frontend

### Web runtime
Web harus:
- memakai route handler / backend layer untuk akses cloud
- memakai auth web
- usable di browser desktop dan HP

Web tidak boleh:
- mengakses Tauri API
- mengakses SQLite desktop
- membawa secret Turso ke browser bundle

### Client component policy
Client component **dilarang**:
- import service DB secara langsung
- import module native-only / node-only yang berisiko terbawa ke browser
- mencampur business rule penting dengan presentational state

---

## 4. Build and Distribution Policy

### Web distribution
Web adalah jalur distribusi universal:
- untuk browser desktop
- untuk HP
- untuk user yang belum install aplikasi

### Desktop distribution
Desktop adalah jalur distribusi offline-first:
- untuk admin / staff / guru / operator
- harus memakai local DB + sync

### Important build rule
Desktop production build **tidak boleh** dibuka jika masih ada flow inti yang bergantung ke runtime web.
Lebih baik build desktop **fail-secure** daripada menghasilkan installer yang terlihat valid tapi rusak.

Status aktual saat ini:
- jalur Windows desktop yang sudah disignoff = `MSI`
- `NSIS` belum otomatis ikut dianggap siap
- signoff desktop harus disebut per-channel installer, bukan digeneralisasi

---

## 5. Technology Baseline

Gunakan stack ini, jangan mengganti tanpa instruksi eksplisit user:

- **Desktop**: Tauri v2
- **Web**: Next.js 16 App Router
- **UI**: React 19
- **Language**: TypeScript strict
- **Styling**: Tailwind CSS v4
- **Components**: shadcn/ui + Radix UI
- **Icons**: Lucide React
- **Toasts**: Sonner
- **State**: Zustand + Nuqs
- **ORM**: Drizzle ORM
- **Cloud DB**: Turso / libSQL
- **Desktop DB**: SQLite
- **Lint/format**: Biome
- **Testing**: Vitest + React Testing Library + Playwright

---

## 6. Database and Schema Policy

### Schema authority
Single source of truth schema ada di:
- [src/core/db/schema.ts](src/core/db/schema.ts)

Kalau ada perubahan schema, agent wajib cek:
- connection layer
- query terkait
- relasi
- migration
- validasi Zod
- sync contract
- data adapter web/desktop

### Minimum entity rules
Untuk entity yang disync, standar field minimal:
- `id`
- `version`
- `updatedAt`
- `deletedAt`
- `syncStatus` jika dipakai local queue
- `hlc` jika dipakai conflict resolution

### Delete policy
- gunakan **soft delete** untuk entity bisnis penting
- jangan hard delete tanpa alasan kuat
- query harus sadar `deletedAt`

---

## 7. Sync Policy

Sync bukan fitur tambahan. Sync adalah bagian inti arsitektur.

### Contract
Desktop:
- write ke local DB dulu
- tandai perubahan untuk sync
- push ke cloud saat online
- pull perubahan cloud ke local

### Conflict policy
Default:
- **LWW dengan HLC-aware policy** jika tersedia

Fallback:
- bandingkan `updatedAt`
- gunakan deterministic tie-breaker jika perlu

### Security
- cloud tidak boleh percaya payload mentah dari client
- semua payload sync harus tervalidasi
- secret sync tidak boleh masuk browser bundle

---

## 8. Auth Policy

### Web auth
- auth web dikelola lewat flow web
- session web hanya untuk runtime web

### Desktop auth
- desktop harus punya flow auth/session lokal sendiri
- desktop tidak boleh mewajibkan `SessionProvider` web untuk berfungsi

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ciola1999/educore](https://github.com/ciola1999/educore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
