---
trigger: always_on
description: React SPA konsumsi REST API dari `G:\New folder\simgos-backend` (Laravel modular, 474 modul).
---

# SIMGOS Frontend

React SPA konsumsi REST API dari `G:\New folder\simgos-backend` (Laravel modular, 474 modul).

## Stack
- Vite + React 19 + TypeScript, React Router, axios
- Tailwind v4 + shadcn-style components (`src/components/ui/`)
- Auth: Laravel Sanctum, Bearer token (bukan cookie), persist di localStorage key `simgos_token`

## Struktur
`src/features/<Modul>` 1:1 mapping ke `Modules/<Modul>` backend. Tiap modul: `api.ts`, `types.ts`, `pages/`.

## Tema
Brand color: primary `#039992`, secondary `#0d5cad`, accent `#0e96ae`. Didefinisikan di `src/index.css`, soft light+dark mode, radius `0.75rem`.

## Urutan pengerjaan
Ikuti urutan build backend (git log oldest-first di `simgos-backend`). Fase-fase ada di rencana (lihat memory `project_simgos_frontend_react`).

## Open questions (cek tiap modul baru, jangan asumsi seragam)
- Format response API per modul (JsonResource wrapping `{data: ...}` vs custom)
- Konvensi versioning route (`v1` dst)

---

## Progress Log

Entry baru ditambah paling atas.

### 2026-08-18 — Fase 1 selesai (audit penuh vs legacy) + mulai Fase 2 (Pendaftaran)
**Fase 1 diaudit ulang lewat Playwright ke app legacy asli** (`http://192.168.56.101/apps/SIMpel/`, kredensial ada di memory) — ketemu banyak gap dari build awal, semua diperbaiki:
- Employee/Patient form: tambah Kontak (list, banyak nomor), Kartu Identitas (list), Employee smf_id jadi dropdown Spesialis. Patient: 7 field lookup (Pendidikan/Pekerjaan/Status Kawin/Gol Darah/Kewarganegaraan/Suku/Bahasa) jadi dropdown asli, tambah Keluarga (list, nested Kontak+Kartu Identitas per anggota), umur otomatis, tempat lahir dapat autocomplete.
- Ward/Room: tambah dropdown Jenis Ruangan/Jenis Kunjungan/Kelas yang sebelumnya gak pernah dipakai walau field-nya ada.
- **Fitur baru: Kelola Penugasan Ruangan** (`/wards/:id/assignments`) — tab Dokter/Spesialis/Paramedis/Staff/Tindakan per ward, generic `WardAssignmentList` component. Backend 6 modul (GeneralDoctor/Nurse/StaffMember + 3 WardAssignment) dipindah dari unauthenticated ke `auth:sanctum`+`v1`, ditambah kolom `is_active`, filter `?ward_id=`.
- Data referensi (Agama/Pendidikan/Pekerjaan/Golongan Darah/Status Kawin/SHDK/Jenis Kontak/Jenis Kartu Identitas/Profesi/Spesialis) diverifikasi satu-satu vs layar "Master > Referensi" legacy, seeder diperbaiki biar PERSIS sama (bukan generate bebas) — termasuk koreksi Spesialis dari 19→36 item yang kepotong scroll di percobaan pertama.
- Enum `contact_type`/`id_type` diperluas ke jumlah pilihan legacy (8 jenis kontak, 7 jenis kartu identitas), frontend+backend disinkronkan.
- Ditemukan & diperbaiki: bug nested-`<form>`-di-dalam-`<form>` (submit nyasar ke form induk) di beberapa shared component (`ContactList`, `IdentityCardList`, `FamilyList`) — semua diganti div+button biasa.

**Fase 2 dimulai — modul pertama sesuai urutan git log backend (`PendaftaranRegistration`+`PendaftaranVisit`)**:
- Keduanya modul matang (auth+v1, paginated, FormRequest, filter by parent id).
- Frontend: `src/features/PendaftaranRegistration` (Registrasi pasien, no. auto `REG-{tahun}-{seq}`, `PatientPicker` search-select baru di shared/components), `src/features/PendaftaranVisit` (Kunjungan/rawat, no. auto `KJ-{tahun}-{seq}`, cascade Ward→Room→Bed, picker dokter dari Employee).
- Nav sidebar baru "Pendaftaran" (Registrasi, Kunjungan).
- Tested end-to-end lewat Playwright: create Registration → create Visit terhubung — sukses persist.

**Ponytail/gap yang sengaja dilewatin** (didokumentasikan, bukan lupa):
- Tab "Barang" di Ruangan legacy — gak ketemu modul backend yang cocok, butuh keputusan desain dulu.
- Menu "Tempat Tidur" (papan okupansi live legacy) — sekarang **memungkinkan** dibangun (PendaftaranVisit punya `bed_id`+`ward_id`+status), tapi backend belum punya endpoint filter `?ward_id=`/`?status=active` di Visit index, jadi occupancy board harus fetch-all-lalu-filter client-side untuk sementara, atau minta endpoint baru dulu.
- Registration form: field `admission_diagnosis_id`/`referral_id`/`package_id` dilewatin (modul GeneralDiagnosisCode/PendaftaranReferral/GeneralPackage belum ada picker-nya di frontend).
- Doctor/Nurse/StaffMember data masih dummy Faker (nama random), belum di-link ke pegawai asli RS.

**Skala backend**: 474 modul total. Urutan build (git log, ascending) setelah Ward/Room/Bed: PendaftaranRegistration/Visit (✅ baru dibangun) → GeneralCountry/Education/Ethnicity/Language/MaritalStatus/Occupation/KemkesBloodType (✅ sudah, dari audit Fase 1) → PembayaranInvoice/InvoiceItem/Payment → GeneralService/ServiceTariff (Service ✅) → MedicalRecordClinicalNote → GeneralDiagnosisCode/MedicalRecordDiagnosis → LayananPrescription/Item → InventoryItem → GeneralRoomClass (✅) → PendaftaranGuarantor → LayananLabOrder/LabResult → dst (lihat `git log --diff-filter=A --name-only --pretty=format:"%h|%ad" --date=short --reverse -- "Modules/*/composer.json"` di `simgos-backend` buat urutan lengkap).

**`PembayaranInvoice`/`InvoiceItem`/`Payment` (billing) — selesai juga di sesi yang sama**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dhrigo/RME-Frontend](https://github.com/Dhrigo/RME-Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
