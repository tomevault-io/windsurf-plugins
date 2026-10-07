---
trigger: always_on
description: Dokumen ini adalah acuan **Wajib & Mengikat** bagi AI Agent (dan developer) dalam membuat, mengubah, memelihara, dan mengembangkan codebase aplikasi ini.
---

# GEMINI.md - Panduan Utama & Rules Coding AI Agent

Dokumen ini adalah acuan **Wajib & Mengikat** bagi AI Agent (dan developer) dalam membuat, mengubah, memelihara, dan mengembangkan codebase aplikasi ini.

---

## 🚨 MANDATORY WORKFLOW FOR AI AGENTS

Setiap kali AI Agent memulai atau mengerjakan tugas pada aplikasi ini, AI Agent **WAJIB** mengecek, merujuk, dan memperbarui seluruh file dokumentasi berikut secara disiplin:

| File Dokumentasi | Fungsi & Peran | Kewajiban AI Agent |
| :--- | :--- | :--- |
| [GEMINI.md](file:///Users/tnoerman/Herd/hajitrackers/GEMINI.md) | Technical Documentation, Rule Coding, Standard Code & Routing. | **Wajib dibaca** untuk memastikan kepatuhan standar teknis. |
| [docs/PRD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/PRD.md) | Product Requirement Document (Acuan awal fitur & tujuan aplikasi). | **Wajib diverifikasi** sebelum membuat fitur baru. |
| [docs/STRUKTUR_MENU.md](file:///Users/tnoerman/Herd/hajitrackers/docs/STRUKTUR_MENU.md) | Acuan RBAC (Role-Based Access Control) & Menu tiap Role. | **Wajib disesuaikan** jika ada penambahan halaman/menu/role. |
| [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md) | Relasi & Skema Database (Entity Relationship Diagram). | **Wajib diupdate** setiap kali ada migrasi / perubahan tabel. |
| [docs/FITUR_APLIKASI.md](file:///Users/tnoerman/Herd/hajitrackers/docs/FITUR_APLIKASI.md) | Pendataan fitur lengkap + Diagram ASCII + Alur Proses + Perubahan Status Data. | **Wajib dibuat / diupdate** setiap ada fitur baru / perubahan alur. |
| [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) | Standar UI/UX, Skema Warna, Pagination, Cards, Filter, CDN (SweetAlert, Icons). | **Wajib diikuti** dalam pembuatan komponen Blade / Frontend. |
| [docs/DOKUMENTASI_PENGGUNAAN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DOKUMENTASI_PENGGUNAAN.md) | Manual Book / Panduan Penggunaan Bahasa Awam (Step-by-Step). | **Wajib diperbarui** agar pengguna awam dapat menggunakan fitur baru. |
| [docs/UAT.md](file:///Users/tnoerman/Herd/hajitrackers/docs/UAT.md) | Skenario Pengujian (User Acceptance Testing) untuk pengujian pengguna. | **Wajib ditambahkan** skenario pengujian baru saat penambahan fitur. |
| [docs/TASK_DONE.md](file:///Users/tnoerman/Herd/hajitrackers/docs/TASK_DONE.md) | Log Perubahan & Audit Trail Pekerjaan AI Agent. | **Wajib dicatat** setiap selesai melakukan perubahan code. |

---

## 🛠️ SPESIFIKASI TEKNIS APLIKASI

| Teknologi | Versi / Detail | Keterangan |
| :--- | :--- | :--- |
| **PHP** | `8.3` | Runtime utama |
| **Framework** | `Laravel 13` | Backend framework |
| **Database** | `PostgreSQL` | Database utama (bukan MySQL/SQLite) |
| **Template Engine** | `Blade (Full)` | 100% Blade — **DILARANG** menggunakan Livewire, Inertia, Vue, React |
| **CSS Framework** | `Tailwind CSS v4` | Sesuai standar [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) |
| **JavaScript** | `jQuery 3.7.1` | Untuk Select2 dan DOM manipulation |
| **Enhanced Select** | `Select2 4.1.0-rc.0` | Dropdown select yang lebih baik |
| **Interaktivitas** | `Alpine.js v3` | **Gunakan seminimal mungkin** — hanya untuk dropdown/modal toggle jika tidak bisa pakai jQuery |
| **Notifikasi** | `SweetAlert2 v11` | Toast & confirmation dialog |
| **Icon** | `Heroicons (SVG inline)` + `RemixIcon (CSS class)` | Sesuai [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) |
| **Spreadsheet** | `PhpSpreadsheet` | Import & Export data (Excel/CSV) |
| **Testing** | `Pest 4` | Unit & Feature testing |

> **PENTING — Alpine.js:** Gunakan Alpine.js **seminimal mungkin**. Jika sebuah interaksi (show/hide, toggle, dll) bisa diselesaikan dengan jQuery atau vanilla JS, maka **JANGAN** menggunakan Alpine.js. Alpine.js hanya boleh digunakan jika memang tidak ada alternatif lain yang lebih sederhana (misalnya: `x-cloak` untuk anti-flash, atau reactive binding yang sangat kompleks).

---

## 📐 ATURAN CODING & ARSITEKTUR APLIKASI

### 1. Filosofi Coding — Eksplisit, Sederhana, Mudah Dibaca

> **Posisikan diri sebagai senior programmer yang menulis code agar mudah dibaca oleh junior programmer yang masih awam.**

- **Tulis code secara eksplisit** — hindari penggunaan magic function, implicit binding, atau shortcut yang menyembunyikan logika.
- **Jangan terlalu modular** — jangan memecah code ke banyak file/class/service jika tidak perlu. Lebih baik satu file controller yang jelas daripada 5 file yang saling panggil.
- **Setiap fungsi ditulis jelas & to the point** — tidak berbelit-belit. Nama fungsi dan variabel menggunakan **bahasa Inggris** yang deskriptif.
- **Setiap baris code diberi komentar singkat dalam bahasa Indonesia** — menjelaskan "fungsi ini / proses ini melakukan apa". Tidak perlu terlalu detail, tetapi tetap menjelaskan proses.

**Contoh komentar yang benar:**
```php
// Ambil semua data jamaah beserta relasi paketnya
$jamaahList = Jamaah::with('jamaahPakets.paketHaji')->paginate(10);

// Validasi input dari form tambah jamaah
$validated = $request->validate([
    'nik' => 'required|string|size:16|unique:jamaahs,nik',
    'nama_lengkap' => 'required|string|max:255',
]);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aliwafa16/safarstrack](https://github.com/aliwafa16/safarstrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
