---
trigger: always_on
description: Briefing untuk agent AI yang mengerjakan repo ini. Baca sebelum menyentuh kode.
---

# AGENTS.md

Briefing untuk agent AI yang mengerjakan repo ini. Baca sebelum menyentuh kode.

`README.md` menjelaskan cara memakai library, dan `CONTRIBUTING.md` menjelaskan alur kontribusi serta code style. Berkas ini mencatat perintah, batasan keras, jebakan domain, dan keputusan yang mahal bila ditemukan ulang.

Instruksi langsung pengguna di percakapan selalu mengalahkan berkas ini.

---

## 1. Orientasi

`MerchantID` adalah toolkit payment-provider **tidak resmi** untuk merchant Indonesia. Implementasi saat ini mendukung:

- **GoPay Merchant / GoBiz**: OTP GoID, refresh token, discovery merchant/outlet/QRIS, dan feed transaksi offset-based.
- **Shopee Merchant / ShopeePay**: OTP fetch-only, cookie/session persistence, discovery merchant/store, dan feed transaksi cursor-based. QRIS statis diberikan manual.

Dua pekerjaan domain bersama:

1. Mengubah QRIS statis merchant menjadi QRIS dinamis per pesanan dengan menyuntikkan nominal ke tag EMV.
2. Mendeteksi settlement dari feed provider dan mencocokkannya ke pesanan berdasarkan nominal unik, waktu, status, dan `PaymentScope`.

API privat provider tidak memberi order reference milik aplikasi. Nominal tetap menjadi pembeda utama. Setiap pembayaran baru juga membawa scope provider/account/merchant-store agar transaksi satu provider atau store tidak pernah melunasi pembayaran lain.

`MerchantID` adalah registry/composition root. Ia tidak memaksakan login universal. Auth, session, merchant/store discovery, pagination, dan normalisasi tetap dimiliki adapter provider.

`GopayProvider` dan `ShopeeProvider` adalah adapter konkret yang diekspor package. API publik hanya mengekspor nama kanonis MerchantID dan adapter provider.

**Library ini memindahkan uang sungguhan di akun merchant orang lain.** Bug rekonsiliasi dapat membuat pembeli sudah membayar tetapi pesanan tidak terkirim, atau pesanan lain terkirim. Ambang ketelitiannya lebih tinggi dari proyek biasa.

Lingkungan pengguna adalah **Windows + PowerShell**. Pisahkan perintah dengan `;`, bukan `&&`.

---

## 2. Perintah

Empat quality gate berikut harus lolos:

```bash
npm run typecheck   # tsc --noEmit untuk src/ DAN test/
npm run lint        # eslint src/**/*.ts
npm test            # vitest run
npm run build       # tsup: index, satu CLI, ESM/CJS/d.ts
```

Development lab multi-provider memakai package root lewat `merchantid: file:../..`, bukan registry npm:

```powershell
npm run build
Set-Location .example/development/web
npm install
npm run dev
```

`.example/development/web` adalah utility TanStack Start + Tailwind live-only dengan satu halaman dan tab GoPay/Shopee, ter-link ke root repo lewat `merchantid: file:../../..`. `.example/production` adalah konsol dua-route (kasir + riwayat) yang mengonsumsi paket `merchantid` terpublikasi dari npm. Pengiriman OTP, discovery, refresh sesi, dan rekonsiliasi dapat mengirim request provider nyata; gunakan hanya akun merchant milik sendiri. Login provider satu-satunya lewat OTP (GoPay: nomor→OTP; Shopee: nomor+password→OTP), tidak ada jalur paste cookie. Session, token, cookie, OTP challenge, provider instance, dan static QRIS harus tetap di modul server; client hanya menerima DTO tersunting serta SVG QR hasil render.

State runtime schema v2 berada di `.example/*/data/` dan diabaikan Git. State atau payment schema lama dihapus saat migrasi agar tidak dianggap sebagai data live. `JsonPaymentStore` hanya untuk satu proses development, bukan store production. Rekonsiliasi sengaja manual agar hot reload tidak menggandakan polling interval.

Quality gate website dijalankan setelah root package dibuild dan dependency lokal terpasang:

```powershell
Set-Location .example/development/web
npm run typecheck
npm test
npm run build
```

Jangan jalankan `npm run dev` sebagai validasi otomatis karena prosesnya tidak berhenti sendiri.

CLI source:

```bash
npx tsx src/cli.ts login gopay
npx tsx src/cli.ts login shopee
npx tsx src/cli.ts session gopay
npx tsx src/cli.ts session shopee --reveal
npx tsx src/cli.ts merchants gopay
npx tsx src/cli.ts stores shopee
npx tsx src/cli.ts set-provider shopee
npx tsx src/cli.ts set-merchant <merchantId> --provider gopay
npx tsx src/cli.ts set-store <storeId>
npx tsx src/cli.ts set-qris shopee
npx tsx src/cli.ts whoami
```

Build menghasilkan satu binary:

- `merchantid` -> `dist/cli.cjs`, CLI multi-provider.

Config default adalah `~/.merchantid/config.json`, dengan env `MERCHANTID_CONFIG`. Schema aktif:

```jsonc
{
  "version": 1,
  "defaultProvider": "gopay",
  "providers": {
    "gopay": { "session": {}, "merchants": [], "defaultMerchantId": "..." },
    "shopee": { "session": {}, "staticQris": "..." },
  },
}
```

Set `MERCHANTID_DEBUG` untuk diagnostik umum. Debug tidak boleh mencetak token, cookie, OTP, atau QRIS.

**Jangan jalankan proses yang tidak berhenti sendiri** seperti `npm run dev`, `npm run test:watch`, monitor, atau `tsup --watch`. Berikan command agar pengguna menjalankannya sendiri di terminal.

---

## 3. Aturan yang tidak bisa dinegosiasi

1. **Nol dependency runtime.** `dependencies` di root `package.json` harus kosong/tidak ada. Pakai `fetch` global dan primitive Web API.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alhifnywahid/merchantid](https://github.com/alhifnywahid/merchantid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
