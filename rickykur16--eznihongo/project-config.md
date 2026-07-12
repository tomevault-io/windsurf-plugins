---
trigger: always_on
description: - [ ] **Offsite backup ke Cloudflare R2** — `RCLONE_REMOTE` di
---

# EzNihongo — Catatan untuk Claude

## Pending ops / infra (jadwal: minggu ini)

- [ ] **Offsite backup ke Cloudflare R2** — `RCLONE_REMOTE` di
  `/var/www/eznihongo/backend/.env` masih kosong, jadi `backup.sh` cuma
  nge-dump lokal di `/var/backups/eznihongo/`. Risiko: kalau VPS hilang
  (disk corrupt / akun suspended / hacker `rm -rf`), backup ikut hilang.
  Setup: rclone config (s3, provider Cloudflare) → `RCLONE_REMOTE=r2:eznihongo-backups`
  di `.env` → test `sudo /var/www/eznihongo/backend/deploy/backup.sh` dan
  `sudo rclone ls r2:eznihongo-backups`. Lihat session sebelumnya untuk
  step lengkapnya.

- [ ] **GPG encryption pada dump** sebelum di-upload offsite. Dump berisi
  email user + raw webhook Midtrans (PII + payment data). Tambahkan
  `gpg --symmetric --cipher-algo AES256` di `backup.sh` sebelum
  `rclone copy`. Passphrase simpan di password manager, bukan di repo.
  Hanya relevan setelah offsite hidup.

- [ ] **`pg_dumpall --globals-only`** terpisah untuk role / grant. Saat ini
  `backup.sh` cuma dump database `eznihongo` — kalau VPS rebuild dari nol,
  role `eznihongo_app` + grant-nya harus dibikin manual dulu.

- [ ] **Test restore ke staging** — tulisan ini ga akan jadi backup beneran
  sampai pernah dicoba di-restore. Minimal sekali per bulan ke Postgres
  staging / container.

- [ ] **Ownership tabel kanji** di production. Migration 008 di-bypass
  via existence-gating (`pg_indexes` / `pg_trigger`), tapi root cause
  ownership masih ada — `kanji_users`, `kanji_sessions`, `subscriptions`,
  `kanji_progress` owned oleh role lain dari `eznihongo_app`.
  Sekali waktu, sebagai superuser:
  ```sql
  ALTER TABLE kanji_users      OWNER TO eznihongo_app;
  ALTER TABLE kanji_sessions   OWNER TO eznihongo_app;
  ALTER TABLE subscriptions    OWNER TO eznihongo_app;
  ALTER TABLE kanji_progress   OWNER TO eznihongo_app;
  ```
  Tanpa ini, migrasi masa depan yang butuh `ALTER TABLE` beneran (bukan
  no-op) bakal kena "must be owner of table" lagi.

- [ ] **SSH masih sebagai `root`** di pipeline deploy
  (`.github/workflows/deploy.yml`). User `deploy` + sudoers terbatas =
  hardening berikutnya, butuh perubahan di sisi VPS.

## Konvensi penting

- **Admin authz**: env `ADMIN_EMAILS` (bootstrap, anti-lockout, read-only dari
  UI) ∪ tabel `admin_emails` (migration 035, dikelola via tombol "Kelola Admin"
  di header `admin.html` — tambah/hapus tanpa edit `.env`/restart). Cek lewat
  `isAdminEmail()` (`backend/src/auth.js`) yang sekarang **async** (DB + cache
  TTL 15s, throw-safe: DB error → fallback env-only) — call site baru WAJIB
  `await` (Promise truthy = semua orang lolos cek). Endpoint:
  `GET/POST /api/admin/admins`, `DELETE /api/admin/admins/:email` (env admin &
  diri sendiri tidak bisa dihapus). Provisioning password co-admin tetap via
  `set-password` (email harus sudah admin).
- **Env**: produksi pakai `backend/.env`, semua var lewat `DATABASE_URL`
  (bukan `DB_PASSWORD` terpisah). systemd service + migration runner +
  backup script semua source dari file yang sama.
- **Migrasi**: tambah file SQL baru di `backend/migrations/`, runner
  (`run.js`) jalan dalam transaction per file, skip kalau sudah
  ke-record di `schema_migrations`. Untuk objek yang mungkin sudah
  ada dari `schema.sql` bootstrap, gate setiap `CREATE INDEX` /
  `CREATE TRIGGER` via `pg_indexes` / `pg_trigger` untuk hindari
  ownership check (lihat 008 sebagai contoh).
- **Pipeline deploy**: push ke `main` → CI parse-check → SSH ke VPS →
  `git reset --hard` + `npm ci --omit=dev` + `npm run migrate` +
  `systemctl restart eznihongo-api` + healthcheck loop ke `/api/health`.
- **Branch konvensi**: `claude/<topic>-<short-id>` untuk fitur Claude.
  PR ke `main`, tidak push langsung.
- **Sync progres lintas device (main site)** — progres "lesson selesai"
  (`ez_progress`) + skor kuis (`ez_quiz_scores`) di `welcome.html` disimpan di
  localStorage **dan** di-mirror ke server sebagai blob JSONB per user (tabel
  `user_learning_state`, migration 026), pola sama seperti `kanji_progress`.
  Endpoint `GET/PUT /api/learning-state` (`backend/src/routes/learning-state.js`,
  `requireAuth`). Frontend: `setProgress` + tulis skor kuis memicu
  `_scheduleCloudPush()` (debounce 1.2s → `PUT`); boot memanggil
  `syncLearningStateFromServer()` yang union-merge cloud↔local (completion
  monoton) sebelum render. XP/streak TIDAK ikut blob — tetap server-authoritative
  via `/api/stats/me`. Tabel relational `user_progress` (lesson UUID) sengaja
  tidak dipakai karena progres frontend di-key slug `"<moduleId>:<lessonId>"`.
- **Tipe pelajaran `deck`** (kosakata interaktif, migration 009): lesson
  bertipe `deck` punya kartu kosakata yang dipilih dari bank
  (`module_vocabulary`, bisa `lesson_id` NULL untuk item bank murni) lewat join
  `lesson_deck_items`; tiap kata punya `vocabulary_examples` (contoh kalimat,
  disimpan polos + kolom `highlight` + `reading` kana penuh per kalimat
  [migration 036, nullable tanpa backfill — frontend sembunyikan baris kana
  kalau kosong; generate-vocab-examples ikut mengisi `reading`; backfill contoh
  lama via tombol "✨ Generate kana (AI)" di Kelola Deck →
  `POST /api/admin/lessons/:id/generate-deck-readings` (Claude `ANTHROPIC_GEN_MODEL`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rickykur16/EzNihongo](https://github.com/Rickykur16/EzNihongo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
