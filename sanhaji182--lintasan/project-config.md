---
trigger: always_on
description: > **Baca file ini dulu sebelum menyentuh kode apa pun.** Ini adalah peta sistem untuk AI agent (Claude Code, Codex, Cursor, Hermes, dll). Tujuannya: paham keseluruhan arsitektur, konvensi, dan cara kerja Lintasan tanpa harus meraba-raba.
---

# AGENTS.md — Lintasan Go

> **Baca file ini dulu sebelum menyentuh kode apa pun.** Ini adalah peta sistem untuk AI agent (Claude Code, Codex, Cursor, Hermes, dll). Tujuannya: paham keseluruhan arsitektur, konvensi, dan cara kerja Lintasan tanpa harus meraba-raba.

---

## 1. Apa itu Lintasan

Lintasan adalah **LLM gateway** — satu endpoint OpenAI-compatible yang merutekan request ke banyak provider AI (OpenAI, Anthropic, DeepSeek, Gemini, Groq, dll) dengan smart routing, failover, caching, token compression, observability, dan dashboard.

- **Backend:** Go (single binary, ~24MB)
- **Frontend:** SvelteKit 5 (dashboard, 26+ halaman)
- **Repo:** `github.com/sanhaji182/lintasan` (monorepo)
- **Filosofi:** "Setiap Koneksi Punya Jalannya."

> ⚠️ **Node.js v1 sudah DIHAPUS (May 2026).** Semua kode aktif ada di monorepo ini. Jangan cari/buat referensi ke `~/lintasan` lama.

---

## 2. Arsitektur Tingkat Tinggi

```
                 ┌─────────────────────────────────────────┐
   Browser  ───► │  nginx (lintasan.sans.biz.id, :443 TLS)  │
                 └────────────────────┬────────────────────┘
                                      │
                 / · /login · /dashboard · /api/* · /v1/* · /health
                                      ▼
                   ┌────────────────────────────────────────┐
                   │  Go backend  :20180   (lintasan start)  │
                   │  ── serves BOTH ──                       │
                   │   • embedded SPA dashboard (go:embed)    │
                   │   • API + OpenAI-compatible LLM gateway    │
                   └────────────────────┬───────────────────┘
                                        │
                                        ▼
            ┌───────────────────────────────────┐
            │ SQLite (data) + provider upstreams │
            └───────────────────────────────────┘
```

**Single binary (sejak v0.24.0):** Dashboard SvelteKit dikompilasi ke static SPA (`adapter-static`) lalu di-`go:embed` ke binary Go via package `internal/web`. Satu proses `lintasan start` di `:20180` menyajikan UI **dan** API — tidak ada proses Node terpisah.

**Pembagian routing nginx (sekarang sederhana):**
- Semua path → Go `:20180`. Backend yang membedakan: `/api/*` `/v1/*` `/mcp` `/health` ditangani handler; sisanya (`/`, `/login`, `/dashboard/*`, `/_app/*`, favicon) dilayani SPA embedded.
- Auth middleware: GET ke path UI/asset statis lewat tanpa auth (shell SPA tidak menyimpan secret; guard berjalan client-side). `/api/*` `/v1/*` `/mcp` **tetap** fail-closed 401 tanpa token. Dikunci 2 security boundary test.

Frontend memanggil API lewat path relatif (`/api/...`, `/v1/...`). **Jangan hardcode `localhost:20180` di frontend.**

> **Mode 2-service lama (deprecated):** SvelteKit `:5173` via `node build/index.js` + nginx split masih bisa jalan karena source `frontend/` utuh, tapi single-binary adalah jalur resmi. Prod sudah migrasi ke single-binary; service `lintasan-dashboard` di-disable.

---

## 3. Port & Service Map

| Service | Port | systemd unit | WorkingDir | ExecStart |
|---------|------|--------------|------------|-----------|
| Go backend (UI + API) | `20180` | `lintasan.service` | `/home/ubuntu/lintasan-go` | `lintasan start` |

Env penting backend: `PORT=20180`.

> Service `lintasan-dashboard` (SvelteKit Node `:5173`) sudah **di-stop + disable** sejak migrasi single-binary. Tidak perlu lagi — UI dilayani backend.

**Backend jalan sebagai systemd (Restart=always, PPID=1).** Jangan jalankan sebagai child process sesi — akan mati saat sesi putus.

Restart setelah deploy:
```bash
sudo systemctl restart lintasan
sudo systemctl is-active lintasan
```

Deploy binary baru (downtime ~0.2–0.3 detik):
```bash
make build                          # frontend → embed → dist-bin/lintasan (TIDAK menyentuh prod)
make deploy                         # stop → backup → cp ke ./lintasan → start → health check
```

> ⚠️ **`make build` TIDAK PERNAH menulis ke `./lintasan`.** Path itu yang
> dieksekusi systemd (`ExecStart=/home/ubuntu/lintasan-go/lintasan start`).
> Build yang menimpanya diam-diam men-stage kode ke produksi: restart
> berikutnya — karena sebab apa pun (reboot, crash, `systemctl restart` agent
> lain) — langsung menaikkannya tanpa langkah deploy dan tanpa persetujuan.
> Ini sudah pernah terjadi. 9 Agustus 2026 pukul 16:31:49 prod menyajikan
> `v0.29.3-17-g23bbc7c`; tujuh detik kemudian stop/start dari worktree lain
> menaikkan `v0.29.3-16-g389ec80` — commit yang **lebih lama** — karena build
> worktree itu meninggalkan binary-nya sendiri di path produksi. Tidak ada yang
> deploy; prod mundur diam-diam selama 100 detik sampai restart berikutnya.
> Semua build mendarat di `dist-bin/`; hanya `make deploy` yang boleh menyentuh
> path produksi, dan ia menyimpan backup bertimestamp lebih dulu.

Rollback (tanpa rebuild — backup dibuat otomatis oleh `make deploy`):
```bash
ls -t lintasan.bak-*                # pilih yang terakhir diketahui baik
sudo systemctl stop lintasan
cp lintasan.bak-<timestamp> lintasan
sudo systemctl start lintasan

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanhaji182/lintasan](https://github.com/sanhaji182/lintasan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
