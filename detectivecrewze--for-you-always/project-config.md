---
trigger: always_on
description: **Aldo** adalah Solo Founder dari **For you, Always.** — sebuah brand digital yang menjual produk kado & surat interaktif premium. Aldo mengatur seluruh visi, arah produk, dan estetika desain.
---

# 🤖 PANDUAN ONBOARDING — AI AGENT
### Platform: **For you, Always.** — Digital Atelier
### Dokumen ini wajib dibaca tuntas sebelum menyentuh kode apapun.

---

## 1. IDENTITAS & ALUR KERJA

### Siapa yang kamu bantu?
**Aldo** adalah Solo Founder dari **For you, Always.** — sebuah brand digital yang menjual produk kado & surat interaktif premium. Aldo mengatur seluruh visi, arah produk, dan estetika desain.

**Aldo tidak melakukan coding sendiri.** Tugas kamu sebagai AI Agent adalah murni sebagai **eksekutor teknis**: membaca instruksi Aldo, memahami konteks codebase, lalu menulis/mengubah kode sesuai permintaan tanpa merusak hal lain.

### Prinsip Kerja Wajib
- JANGAN menghapus atau merombak kode yang tidak diminta.
- JANGAN mengambil keputusan besar (arsitektur, hapus produk) tanpa konfirmasi eksplisit dari Aldo.
- SELALU cek `git log` sebelum membuat perubahan besar — Aldo sangat memperhatikan git history dan sering minta revert ke commit tertentu.
- SELALU lakukan `git add . && git commit -m "..." && git push origin main` setelah setiap perubahan selesai.
- Jika ada kebingungan atau instruksi ambigu, tanya dulu — jangan asumsi sendiri.
- Pastikan perubahan yang diminta sudah benar-benar terefleksi di file sebelum commit.

### Gaya Komunikasi
Aldo berbicara dalam **Bahasa Indonesia** santai. Balas juga dalam Bahasa Indonesia, ringkas, dan langsung ke poin.

---

## 2. ARSITEKTUR PLATFORM — GAMBARAN BESAR

Platform ini terdiri dari **dua lapisan utama**:

```
+----------------------------------------------------------+
|  LAYER 1: Valentine-Platform (Next.js)                   |
|  Landing page, katalog produk, checkout flow             |
|  Domain: for-you-always.my.id                            |
+---------------------------+------------------------------+
                            | user membeli -> redirect ke produk
                            v
+----------------------------------------------------------+
|  LAYER 2: Produk Individual (Vanilla JS / Next.js)       |
|  Setiap produk punya repo & subdomain sendiri            |
|  Contoh: letter.for-you-always.my.id                     |
+----------------------------------------------------------+
                            ^
                            |
+----------------------------------------------------------+
|  pakasir-gateway (Cloudflare Worker)                     |
|  Payment gateway terpusat - menghubungkan semua produk   |
+----------------------------------------------------------+
```

---

## 3. PEMETAAN FOLDER PROJECT

Semua folder proyek berada di: `C:\Users\aldor\OneDrive\Desktop\`

---

### [Valentine-Platform/] — INDUK UTAMA (Next.js 14, TypeScript)
> INI ADALAH REPO YANG PALING SERING DIUBAH. Fokus utama kamu.

Deploy  : for-you-always.my.id
Stack   : Next.js 14 (App Router), TypeScript, Vanilla CSS, Vercel

#### Struktur Internal:

```
Valentine-Platform/
├── app/
│   ├── layout.tsx              <- Root layout: font, metadata SEO, schema.org, analytics
│   ├── globals.css             <- Global CSS variables & resets
│   ├── robots.ts               <- SEO robots config
│   ├── sitemap.ts              <- Sitemap otomatis
│   ├── (landing)/              <- Semua halaman publik (route group, tidak muncul di URL)
│   │   ├── page.tsx            <- HOMEPAGE UTAMA — sangat besar (~1100 baris)
│   │   ├── landing.css         <- CSS khusus landing page
│   │   ├── layout.tsx          <- Layout wrapper untuk semua halaman landing
│   │   ├── LetterEnvelopePreview.tsx  <- Komponen preview amplop interaktif
│   │   ├── catalog/
│   │   │   ├── page.tsx        <- Halaman /catalog — grid semua produk
│   │   │   ├── letter/         <- Halaman detail: Letter Edition
│   │   │   ├── voices/         <- Halaman detail: Voices Gift
│   │   │   ├── mixtape/        <- Halaman detail: Mixtape Edition
│   │   │   ├── invitation/     <- Halaman detail: Invitation Edition
│   │   │   ├── arcade/         <- Halaman detail: Arcade Edition
│   │   │   ├── retro/          <- Halaman detail: Retro Edition
│   │   │   ├── wrapped/        <- Halaman detail: Wrapped Edition
│   │   │   └── memoria/        <- Halaman detail: Memoria (Premium)
│   │   ├── letter/             <- Landing page khusus Letter Edition (bukan katalog)
│   │   ├── voices/             <- Landing page khusus Voices Gift
│   │   ├── arcade/             <- Landing page khusus Arcade Edition
│   │   ├── wrapped/            <- Landing page khusus Wrapped Edition
│   │   ├── bundle/             <- Halaman paket bundle produk
│   │   └── order-status/       <- Halaman konfirmasi status pembayaran pasca checkout
│   ├── components/             <- Komponen React reusable
│   │   ├── LandscapeProductCard.tsx  <- Kartu produk landscape (TERKOMPLEKS, ~800 baris)
│   │   ├── CompactProductCard.tsx    <- Kartu compact untuk grid katalog
│   │   ├── CarouselProductCard.tsx   <- Kartu format carousel
│   │   ├── CheckoutModal.tsx         <- Modal popup checkout & payment
│   │   ├── Navbar.tsx                <- Navbar floating
│   │   └── ProductCarousel.tsx       <- Auto-scroll carousel produk

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [detectivecrewze/for-you-always](https://github.com/detectivecrewze/for-you-always) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
