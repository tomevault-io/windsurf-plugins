---
trigger: always_on
description: File ini adalah instruksi kerja untuk agen AI (ChatGPT Codex, Claude Code, atau
---

# Panduan agen: update WhatsApp Web untuk Elaina Baileys

File ini adalah instruksi kerja untuk agen AI (ChatGPT Codex, Claude Code, atau
sejenisnya) yang diminta memeriksa dan menerapkan update WhatsApp Web ke fork
ini. Kalau kamu memakai ChatGPT versi chat biasa, salin seluruh isi file ini
sebagai *system prompt* / *custom instruction*, lalu berikan output perintah di
bawah sebagai bahan analisis.

Repo ini adalah **distribusi Baileys yang dipelihara sendiri**. Jangan pernah
membandingkannya dengan WhiskeySockets/Baileys atau fork lain, jangan menyalin
kode dari sana, dan jangan berasumsi perilaku upstream berlaku di sini. Satu-
satunya sumber kebenaran adalah **bundle JavaScript WhatsApp Web yang live** dan
kode di repo ini.

---

## 1. Aturan keras

Langgar salah satu dari ini dan hasil kerjamu tidak bisa dipakai.

1. **Jangan pernah mengarang temuan.** Setiap klaim tentang WhatsApp Web harus
   berasal dari string yang benar-benar kamu temukan di bundle. Kalau kamu tidak
   menemukannya, katakan "tidak ditemukan", bukan "kemungkinan besar".
2. **Ekstraktor yang menghasilkan nol baris bukan bukti bahwa tidak ada
   perubahan.** Sebelum menyimpulkan "tidak ada X baru", pastikan dulu ekstraktor
   X memang menghasilkan data pada kedua bundle. Laporan dari `wa:update` sudah
   mencantumkan `Total sekarang` untuk setiap permukaan — kalau totalnya 0,
   ekstraktornya rusak, bukan WhatsApp-nya yang diam.
3. **Jangan bump versi kalau `verify:proto` gagal.** Round-trip encoder adalah
   pengaman terakhir terhadap field protobuf yang salah nomor atau salah tipe.
4. **Jangan menambah field protobuf secara manual.** Pakai `npm run sync:proto`.
   Skrip itu aditif dan idempoten; tangan manusia pernah membuat field ganda
   (`faviconMmsMetadata` vs `faviconMMSMetadata` di field 33) yang hanya
   ketahuan karena round-trip test. Sejak revisi `1047020237` skrip ini juga
   membangkitkan anggota baru `Message` — dulu ia menyerah di situ dan menyuruh
   regenerasi manual. Setiap anggota `Message` adalah proto3 optional, yaitu
   oneof sintetis beranggota satu, bentuk yang sama persis dengan field
   opsional di tipe lain; yang dulu kurang cuma spec-nya, karena `diffBundle`
   melaporkan field `Message` hanya sebagai nama dan nomor. Sekarang spec itu
   diambil dari `bundle.specs.get('Message')` lalu masuk ke codegen yang sama.
5. **Kode yang kamu tulis tidak boleh memakai komentar `//` atau `/** */`**
   kecuali komentar itu menjelaskan sesuatu yang benar-benar tidak terbaca dari
   kodenya. Dokumen markdown berbahasa Indonesia bebas dari aturan ini.
6. **Commit langsung ke `main`.** Jangan membuat branch baru, jangan membuka PR,
   kecuali diminta eksplisit.
7. **Identitas commit** harus milik pemilik repo, bukan milik agen:
   ```
   git -c commit.gpgsign=false \
       -c user.name="RexxHayanasi Pengen S.Kom" \
       -c user.email="150516773+rexxzyid@users.noreply.github.com" \
       commit -am "<pesan>"
   ```
   Tanpa trailer `Co-Authored-By`, tanpa menyebut model atau alat apa pun di
   pesan commit.

---

## 2. Alur kerja: satu perintah

```bash
npm run wa:update
```

Skrip ini melakukan seluruh rantai kerja:

1. Membaca revisi yang terpasang di `lib/Defaults/index.js`.
2. Mengambil revisi live dari `https://web.whatsapp.com/sw.js`.
3. Mengunduh seluruh chunk bundle (±540 file, ±80 MB) ke `.wa-bundle/<revisi>/`.
   Kalau snapshot revisi itu sudah ada, dipakai ulang tanpa mengunduh.
4. Mem-parse spesifikasi protobuf dari bundle dan membandingkannya dengan
   `WAProto/index.d.ts`.
5. Membandingkan snapshot baru dengan snapshot revisi sebelumnya di seluruh
   permukaan protokol.
6. Menjalankan round-trip encoder atas ±2870 field.
7. Menulis `.wa-bundle/report.md` dan `.wa-bundle/report.json`.

Opsi:

| Opsi | Arti |
|---|---|
| `--apply` | Bump revisi terpasang, tapi hanya kalau kesimpulannya `bump-only` atau `bump-and-review` |
| `--cache <dir>` | Lokasi cache snapshot (default `.wa-bundle/`) |
| `--out <dir>` | Lokasi file laporan |
| `--keep <n>` | Jumlah snapshot yang disimpan (minimum 2, default 3) |

Perintah pendukung:

| Perintah | Kegunaan |
|---|---|
| `npm run wa:diff -- <dir-lama> <dir-baru>` | Diff dua snapshot saja |
| `npm run check:proto` | Cek celah protobuf saja |
| `npm run sync:proto` | Tambahkan field protobuf yang hilang ke WAProto |
| `npm run verify:proto` | Round-trip encoder saja |
| `npm run fetch:bundle -- <dir>` | Unduh bundle mentah ke direktori |

Variabel lingkungan `PROTO_BUNDLE_DIR` membuat semua skrip membaca dari
direktori lokal, dan `PROTO_OFFLINE=1` melewati pengambilan revisi live —
berguna kalau jaringan diblokir.

> **Catatan rollout.** `sw.js` dan CDN chunk bisa berbeda pendapat saat rilis
> sedang digelar bertahap: `sw.js` sudah mengumumkan revisi baru sementara
> chunk yang dilayani masih revisi lama. Laporan membedakan keduanya lewat
> `Revisi live` (yang diumumkan) dan `Revisi yang dianalisis` (yang benar-benar
> terunduh). Semua kesimpulan — termasuk `--apply` — memakai revisi yang
> dianalisis, karena revisi yang bundle-nya belum pernah dibaca tidak boleh
> ikut dibump.

> **Catatan jaringan.** Di sebagian lingkungan, `fetch` bawaan Node ditolak
> `403` oleh web.whatsapp.com sementara `curl` lolos. `script/protobundle.js`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rexxzyid/elaina-baileys](https://github.com/rexxzyid/elaina-baileys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
