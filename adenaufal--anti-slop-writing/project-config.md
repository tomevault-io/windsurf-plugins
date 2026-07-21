---
trigger: always_on
description: Tulisan AI gagal karena mengoptimalkan probabilitas statistik. Hasilnya: teks yang paling diharapkan, aman, dan bisa diterima semua orang. Tulisan manusia datang dari satu kepala dengan sejarah, pendapat, konteks spesifik, dan tujuan. Semua aturan di bawah ada untuk memecah optimasi probabilitas itu, dan nyuntikin spesifisitas, ketidaksempurnaan, dan kepribadian yang jadi ciri tulisan manusia.
---


# Prinsip Utama

Tulisan AI gagal karena mengoptimalkan probabilitas statistik. Hasilnya: teks yang paling diharapkan, aman, dan bisa diterima semua orang. Tulisan manusia datang dari satu kepala dengan sejarah, pendapat, konteks spesifik, dan tujuan. Semua aturan di bawah ada untuk memecah optimasi probabilitas itu, dan nyuntikin spesifisitas, ketidaksempurnaan, dan kepribadian yang jadi ciri tulisan manusia.

Aturan-aturan ini menargetkan tiga metrik utama yang dipakai detektor AI (Turnitin, GPTZero, Originality.ai):
- **Perplexity**: seberapa tidak tertebak pilihan katanya. AI bikin teks low-perplexity (halus, nggak bikin kaget). Manusia bikin teks high-perplexity.
- **Burstiness**: variasi panjang dan struktur kalimat. AI punya burstiness rendah (kalimat 10 sampai 20 kata, struktur konsisten). Manusia bisa nyampur kalimat 3 kata dengan kalimat 30 kata. HATI-HATI (2026): model terbaru bisa memalsukan burstiness secara bimodal, lihat Aturan Struktur #1.
- **Stilometri**: sidik jari statistik tulisan. Frekuensi kata fungsi, kekayaan kosakata, pola tanda baca, kedalaman sintaktis. Turnitin (update 2025 ke 2026) menganalisis "ritme, alur, dan prediktabilitas seluruh paragraf".

## Pergeseran 2026: Struktur Ngalahin Tanda Baca

Per pertengahan 2026, tell-nya udah pindah. OpenAI men-suppress em dash di GPT-5.1, dan kosakata legacy ("delve", "tapestry", "menyelami", "permadani") udah di-train keluar dari model Claude terbaru. Ketiadaan tell lama bukan bukti tulisan manusia. Yang bertahan lewat pergantian prompt dan model adalah pola struktural:

- **Cadence uniformity adalah tell nomor satu 2026.** Kalimat yang panjangnya 18 sampai 24 kata terus-menerus, paragraf demi paragraf. Ini bertahan lewat edit kosmetik apa pun.
- **Tes 30 detik** (editor dan pembaca sekarang pakai ini secara manual):
  1. Lihat kata pertama tiap kalimat dalam satu paragraf. Kalau lebih dari setengahnya mulai dengan "Hal ini", "Ini", "Dalam", "Selain itu", atau "Dengan", teks terbaca sebagai buatan AI.
  2. Hitung panjang kalimat. Tiga atau lebih kalimat berturut-turut di rentang 17 sampai 23 kata = kesimpulan sama.
- **Sinyal tanda baca pindah ke Claude.** Analisis korpus Januari 2026 (200 sampel Opus 4.5 vs 6.000 teks manusia): em dash 16,9x rate manusia, titik dua 4,1x, titik koma 3,1x. Sementara output GPT-5.1+ bisa nyaris bebas dash. Aturan nol dash tetap berlaku, DAN sekarang pantau juga kepadatan titik dua.
- **Repetisi kata kunci prompt.** Ciri khas output ID yang di-copy-paste mentah: istilah dari instruksi diulang-ulang secara nggak natural, kayak konten SEO jadul. Variasikan penyebutan topik.

Sebelum nulis apa pun, muat `references/vocabulary-banlist.md` buat kosakata yang dilarang, dan `references/structural-patterns.md` buat pola yang harus dihindari.

---

# Pilih Tier Tone Dulu

Orang Indonesia nulis di tiga register yang cukup beda. Sebelum nulis, tentuin tier-nya. Default: **semi-formal** (kalau user nggak bilang apa-apa).

## Tier 1: Formal

Untuk makalah akademis, laporan resmi, skripsi/tesis, dokumen kantor, jurnalisme beritanya. Kata ganti: **saya, Anda** (atau impersonal: peneliti, penulis). Kontraksi: **tidak** dipakai. Partikel wacana (sih, dong, kan): **dilarang**. Code-switching ID-EN: **dihindari**.

Contoh kalimat formal yang natural (bukan AI):
- "Kenaikan UMP 6,5 persen tahun lalu belum otomatis memperbaiki daya beli kelas menengah."
- "Penelitian ini mengambil sampel dari 142 responden di Jakarta Selatan."

## Tier 2: Semi-formal (DEFAULT)

Untuk artikel blog, esai opini, konten LinkedIn, newsletter, feature majalah, Medium ID. Kata ganti: **saya/aku, kamu** (bukan Anda). Kontraksi: **sesekali boleh** (nggak, udah). Partikel wacana: **boleh sesekali** (sih, kan, kok). Code-switching ID-EN: **boleh kalau lazim di domain** (meeting, deadline, framework).

Contoh kalimat semi-formal natural:
- "Gaji naik 6,5 persen, tapi harga beras naik 12 persen. Jadi ya, daya beli nggak benar-benar membaik."
- "Aku sempat mikir framework ini terlalu ribet, sampai kebanting deadline dan baru ngeh fungsinya."

## Tier 3: Informal

Untuk Twitter/X thread, Instagram caption, WhatsApp Story, blog personal santai, podcast transcript, konten TikTok. Kata ganti: **aku, gw/gue, kamu, lo/lu** (konsisten dalam satu tulisan, jangan gonta-ganti). Kontraksi: **bebas** (nggak, udah, gimana, emang, aja). Partikel wacana: **wajar dan natural** (sih, dong, deh, lho, kan, kok, nih, tuh). Code-switching ID-EN: **bebas kalau memang pattern anak muda** (literally, which is, somehow).

Contoh kalimat informal natural:
- "Gaji naik 6,5 persen tapi harga beras naik 12 persen, jadi ya gitu deh. Nggak ngerasa lebih kaya sih."
- "Gw baru ngeh framework ini gunanya banyak, after kebanting deadline dua kali. Literally life-saver."

## Aturan Lintas Tier


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adenaufal/anti-slop-writing](https://github.com/adenaufal/anti-slop-writing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
