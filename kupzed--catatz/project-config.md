---
trigger: always_on
description: CatatZ adalah aplikasi pencatatan keuangan berbasis Next.js 16 App Router, Supabase, Server Actions, dan PWA Serwist. File ini adalah instruksi repository-level untuk agent saat bekerja dari root repository CatatZ.
---

# CatatZ - Codex Operating Instructions

## Origin

CatatZ adalah aplikasi pencatatan keuangan berbasis Next.js 16 App Router, Supabase, Server Actions, dan PWA Serwist. File ini adalah instruksi repository-level untuk agent saat bekerja dari root repository CatatZ.

Instruksi detail tetap berada di `docs/ai-development-rules.md`. Anggap file ini sebagai operating manual singkat yang mengikat Codex agar setiap task selesai dengan perubahan yang rapi, dokumentasi yang sinkron, validasi yang jujur, dan suggested commit message yang teknis.

## Source of Truth

Ada beberapa sumber kebenaran yang harus dipakai sebelum membuat perubahan:

- Source code saat ini adalah kebenaran utama untuk behavior aplikasi.
- `docs/` adalah kebenaran dokumentasi dan harus mengikuti behavior aktual, bukan rencana.
- `docs/ai-development-rules.md` adalah kebenaran utama untuk aturan kerja AI di repo ini.
- **`DESIGN.md` adalah kebenaran utama untuk semua keputusan visual, design token, dan component styling.**
- `docs/frontend-guidelines.md` adalah kebenaran implementasi desain di codebase ini (adaptasi DESIGN.md ke Tailwind/shadcn).
- `src/migrations` adalah lokasi migration project ini. Jangan mengasumsikan `supabase/migrations`.
- `.env.example` hanya boleh berisi nama variable dan placeholder aman. Jangan expose nilai secret dari `.env`.

Jika dokumentasi dan source code berbeda, percaya source code dulu, lalu update dokumentasi yang relevan dalam task yang sama.

## ECC Workflow Overlay

CatatZ memakai subset Everything Claude Code (ECC) secara project-local untuk Codex dan Claude Code.

- `.agents/skills/` adalah canonical skill surface. `.claude/skills/` hanya berisi wrapper kompatibilitas yang menunjuk ke canonical skill.
- Gunakan skills `coding-standards`, `frontend-patterns`, `security-review`, `verification-loop`, `documentation-lookup`, `strategic-compact`, `agent-introspection-debugging`, `product-capability`, `tdd-workflow`, dan `e2e-testing` sesuai task.
- `nextjs-turbopack` adalah library reference, bukan aturan production build. Production CatatZ tetap memakai `next build --webpack` untuk Serwist.
- Specialized agents di `.codex/agents/` dan `.claude/agents/` bersifat read-only. Main agent adalah satu-satunya pihak yang boleh mengedit file.
- Hooks project memblokir command destruktif dan perubahan secret file, tetapi quality check hanya warning. CI tetap menjadi enforcement utama.
- Memory hook hanya boleh menyimpan metadata di `.ecc/runtime/`: path file, status verifikasi, timestamp, dan identifier sesi yang sudah di-hash. Jangan simpan prompt, tool arguments, isi file, secret, atau data finansial.
- MCP project hanya `chrome-devtools`. Jangan tambahkan Supabase, database production, GitHub, memory, atau connector lain tanpa review eksplisit.

Workflow default:

1. Fitur/refactor kompleks: planner -> TDD -> implementasi main agent.
2. Setelah perubahan: code review; tambahkan security review untuk auth, RLS, input, upload, Server Actions, secret, atau data sensitif.
3. Jalankan `npm run verify:quick` selama iterasi dan `npm run verify` sebelum PR jika browser Playwright tersedia.
4. Target 80% berlaku untuk kode baru/diubah dan modul pure yang dimasukkan ke coverage, bukan klaim coverage global legacy code.

## Design Contract

**WAJIB dibaca sebelum menyentuh file UI apapun.**

`DESIGN.md` adalah sistem desain institusional berbasis Coinbase brand. Setiap perubahan tampilan — komponen baru, styling baru, halaman baru — HARUS mengikuti aturan berikut:

### Token Warna

| Token CSS | Nilai | Gunakan untuk |
|---|---|---|
| `bg-primary` / `text-primary` | #0052ff | CTA utama, active state nav, accent link |
| `bg-surface-dark` | #0a0b0d | Dark hero card, editorial band |
| `bg-surface-dark-elevated` | #16181c | Card di atas dark background |
| `bg-surface-soft` | #f7f7f7 | Alternating band, muted section background |
| `bg-surface-strong` | #eef0f3 | Secondary button bg, badge bg, icon plate |
| `text-semantic-up` | #05b169 | Pemasukan / nilai positif — **text only, jangan pakai sebagai bg** |
| `text-semantic-down` | #cf202f | Pengeluaran / nilai negatif — **text only, jangan pakai sebagai bg** |
| `border-hairline` | #dee1e6 | Default border/divider pada surface terang |

### Shape Rules

- **Semua CTA button WAJIB `rounded-full` (pill).** Tidak ada pengecualian.
- **Card/container menggunakan `rounded-[24px]` atau `rounded-card`.** Bukan `rounded-xl` default shadcn.
- **Form input menggunakan `rounded-[12px]` atau `rounded-input`, height `h-12`.**
- **Badge/tag menggunakan `rounded-full` (pill).**
- Jangan pakai `rounded-none` (0px) pada komponen interaktif.

### Typography Rules

- **Font utama: Inter** (diimpor via `next/font/google`, variable `--font-inter`).
- **Font monospace/angka: Geist Mono** (variable `--font-geist-mono`) — gunakan `font-mono` untuk semua nominal keuangan.
- **Heading display (`h1`): `text-[32px] font-normal tracking-[-0.4px]`** — JANGAN `font-bold` untuk page title utama.
- **Section title: `text-lg font-semibold`** atau sesuai hirarki yang sudah ada.
- **Semua angka nominal keuangan WAJIB `font-mono`.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kupzed/catatz](https://github.com/kupzed/catatz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
