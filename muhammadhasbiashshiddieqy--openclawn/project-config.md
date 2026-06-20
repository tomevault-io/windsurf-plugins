---
trigger: always_on
description: > File ini adalah **panduan operasional** untuk agent coding (Claude Code / Sonnet / model lain) yang mengimplementasikan OpenCLAWN.
---

# CLAUDE.md — Panduan Implementasi OpenCLAWN

> File ini adalah **panduan operasional** untuk agent coding (Claude Code / Sonnet / model lain) yang mengimplementasikan OpenCLAWN.
>
> **Cara pakai:** Bawa file ini + `openclawn-core-spec-v0.3.md` ke repository baru. Dua file ini cukup untuk memberi konteks penuh tanpa membawa riwayat percakapan. Baca **kedua** file sebelum menulis kode apa pun.
>
> **Aturan emas:** Spec (`openclawn-core-spec-v0.3.md`) adalah *sumber kebenaran* untuk APA yang dibangun. File ini (`CLAUDE.md`) adalah sumber kebenaran untuk BAGAIMANA membangunnya — konvensi, urutan kerja, dan hal yang tidak boleh dilanggar.

---

## 0. TL;DR untuk agent yang baru masuk

Kamu sedang membangun **OpenCLAWN**: framework agent AI yang ringan, aman, self-improving, dan multi-role. Python 3.12, FastAPI + HTMX, SQLite, hybrid LLM (Ollama lokal + Claude API).

Yang membuat proyek ini berbeda adalah **4 inovasi inti**:
1. **Routing audit + self-calibration** — catat setiap keputusan routing + apakah terbukti tepat
2. **Skill decay** — skill yang jarang dipakai memudar dan ter-arsip
3. **Confidence-gated crystallization** — agent menilai kualitas solusinya sebelum menyimpannya sebagai skill
4. **Role output contracts** — handoff antar role tervalidasi dengan Pydantic

Empat inovasi ini bukan fitur tambahan — mereka adalah inti dari nilai proyek. Jangan pernah memangkasnya demi "menyederhanakan".

**Mulai dari Sprint 0** di §21 spec. Jangan loncat. Bangun fondasi (`infra/`) dulu, baru yang lain.

---

## 1. Prinsip yang tidak boleh dilanggar

Urut berdasarkan prioritas. Jika dua prinsip bertabrakan, yang lebih atas menang.

1. **Keamanan dulu.** `code_run` HARUS berjalan di dalam Docker sandbox (`--network none`, `--read-only`, non-root, timeout). Tidak ada eksekusi kode di host. Tidak pernah. Lihat spec §16.
2. **Credential tidak pernah masuk context/prompt.** Hanya diinjeksi saat outbound request via `Vault`. Jangan pernah log API key. Jangan pernah taruh di tabel DB.
3. **Setiap dependency eksternal punya kegagalan yang anggun.** LLM call → retry + fallback chain. Ollama offline ≠ agent mati. Lihat spec §8.
4. **Token-first.** Sebelum menambah apa pun ke context window, tanya: apakah ini perlu? Target < 28K token. Aktifkan prompt caching untuk bagian stabil.
5. **Tidak ada hardcoded domain/locale.** OpenCLAWN harus netral. Tidak ada "ServisIn", tidak ada "Depok", tidak ada Bahasa Indonesia yang dipaksakan di core. Locale via field `locale`, bukan di kode.
6. **Setiap inovasi = modul yang bisa diekstrak.** Tulis `skill_decay.py`, `audit.py`, `crystallizer.py`, `contracts.py` sedemikian rupa sehingga suatu hari bisa dijadikan paket terpisah. Jangan bocorkan ketergantungan spesifik OpenCLAWN ke dalamnya selain lewat interface yang jelas (`DatabaseManager`).

---

## 2. Konvensi kode (WAJIB diikuti)

### Bahasa & gaya
- **Python 3.12+**, gunakan fitur modern: `match`, `|` union types, `list[str]` bukan `List[str]`.
- **Type hints di semua fungsi publik.** Tidak ada `Any` kecuali benar-benar terpaksa.
- **`async`/`await` di semua I/O.** DB, HTTP, file besar. Tidak ada blocking call di event loop.
- **Format dengan `ruff format`. Lint dengan `ruff check`.** Jalankan sebelum menganggap tugas selesai.
- **Docstring singkat** untuk setiap kelas dan fungsi non-trivial. Bahasa Indonesia boleh, English boleh — konsisten dalam satu file.

### Penamaan
- File & modul: `snake_case.py`
- Kelas: `PascalCase`
- Fungsi & variabel: `snake_case`
- Konstanta: `UPPER_SNAKE_CASE` di puncak modul
- Private: prefix `_` (mis. `_load_soul_once`)

### Struktur
- Satu kelas utama per file modul. File pendukung kecil OK.
- Import absolut dari root proyek: `from infra.database import DatabaseManager`. Bukan relative.
- Konstanta konfigurasi di puncak file, bukan magic number tersebar.

### Komentar
- Jelaskan **mengapa**, bukan **apa**. Kode yang baik sudah menjelaskan "apa".
- Untuk setiap perbaikan dari audit, tinggalkan komentar referensi: `# Audit #4: evaluator minimal setara generator`. Ini memudahkan pelacakan.

---

## 3. Aturan khusus per modul

### `infra/` — bangun PERTAMA
Semua modul lain bergantung pada ini. Jangan tulis modul lain sebelum `infra/` jalan dan tertest.
- `config.py`: `AppConfig` adalah `frozen=True` dataclass. Semua angka ajaib (timeout, threshold, max tokens) ada di sini, bukan tersebar.
- `database.py`: `DatabaseManager` memegang SATU koneksi shared. Daftarkan `POWER(base, exp)` sebagai custom function (SQLite tidak punya bawaan — dibutuhkan exponential decay). Aktifkan WAL mode.
- `logging.py`: structlog JSON. Setiap error di background task HARUS ter-log.

### `core/llm_client.py` — bangun KEDUA
Ini fondasi semua interaksi LLM. Jangan ada modul yang call LLM langsung; semua lewat sini.
- `stream_with_fallback()` adalah satu-satunya entry point publik.
- Health check Ollama sebelum pakai. Anthropic asumsikan up, andalkan retry.
- Retry hanya untuk `httpx.HTTPError` (transient). Jangan retry error logika.
- Prompt caching: bungkus system prompt dengan `cache_control: ephemeral`.
- **Jangan pakai SDK Anthropic atau OpenAI.** Raw httpx. Ini disengaja untuk transparansi audit.

### `core/router.py` — soul-aware

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MuhammadHasbiAshshiddieqy/OpenClawn](https://github.com/MuhammadHasbiAshshiddieqy/OpenClawn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
