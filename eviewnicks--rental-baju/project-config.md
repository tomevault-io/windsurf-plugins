---
trigger: always_on
description: Dokumen ini berisi protokol standar untuk proses debugging pada aplikasi Maguru, khususnya untuk fitur-fitur kompleks seperti auto-save draft dan toggle mode editor. Protokol ini dirancang untuk membantu developer mengidentifikasi, melacak, dan menyelesaikan masalah secara sistematis.
---

# Protokol Debugging Maguru

## 1. Pendahuluan

Dokumen ini berisi protokol standar untuk proses debugging pada aplikasi Maguru, khususnya untuk fitur-fitur kompleks seperti auto-save draft dan toggle mode editor. Protokol ini dirancang untuk membantu developer mengidentifikasi, melacak, dan menyelesaikan masalah secara sistematis.

## 2. Persiapan Debugging

### 2.1 Konfigurasi Logger

Sebelum memulai debugging, pastikan sistem logging sudah dikonfigurasi dengan benar:

```typescript
// Pastikan level logging sesuai kebutuhan debugging
logger.setLevel('debug');
```

### 2.2 Tools Debugging yang Diperlukan

- Browser DevTools (Chrome/Firefox)
- React DevTools Extension
- Redux DevTools (jika menggunakan Redux)
- Network Monitor pada DevTools
- Winston Logger yang sudah diimplementasikan

## 3. Protokol Debugging untuk Toggle Mode Editor

### 3.1 Identifikasi Masalah

1. **Persiapan Lingkungan**:
   - Buka halaman modul yang memiliki masalah
   - Buka DevTools (F12) dan pilih tab Console
   - Aktifkan "Preserve log" pada Console untuk menyimpan log saat navigasi
   - Filter log dengan string "DocumentHeader" atau "ModuleDraftPageContext"

2. **Reproduksi Masalah**:
   - Catat status awal: `editorMode`, `activePage.status`
   - Klik tombol toggle mode (Edit/View)
   - Amati perubahan UI dan log yang muncul

3. **Analisis Log**:
   - Cari log dengan format `=== TOGGLE MODE START: Dari [mode] ke [mode] ===`
   - Periksa urutan eksekusi langkah-langkah dalam `handleToggleMode`
   - Identifikasi langkah mana yang gagal atau tidak berjalan sesuai ekspektasi

### 3.2 Pemeriksaan State dan Props

1. **Gunakan React DevTools**:
   - Pilih komponen `DocumentHeader`
   - Periksa props `editorMode` dan `activePage`
   - Pilih komponen `RichTextEditor`
   - Periksa props `readOnly` dan state internal editor

2. **Periksa Context Value**:
   - Pilih provider `ModuleDraftPageContext.Provider`
   - Periksa nilai `editorMode` dan fungsi `toggleEditorMode`
   - Bandingkan dengan nilai yang diharapkan

### 3.3 Debugging Toggle Mode Step-by-Step

1. **Verifikasi Alur Data**:
   ```
   Klik tombol Edit/View
   → handleToggleMode() di DocumentHeader
   → toggleEditorMode() di ModuleDraftPageContext
   → Perubahan status halaman di database
   → Perubahan state React (editorMode)
   → Perubahan editor.isEditable di RichTextEditor
   ```

2. **Periksa Race Condition**:
   - Cek apakah ada flag `isTogglingMode` di sessionStorage
   - Periksa timing antara perubahan state React dan perubahan status halaman
   - Cek apakah ada multiple request yang berjalan bersamaan

3. **Debugging Spesifik untuk Toggle Mode**:
   - Tambahkan breakpoint di fungsi `handleToggleMode`
   - Tambahkan breakpoint di fungsi `toggleEditorMode`
   - Tambahkan breakpoint di useEffect yang merespons perubahan `editorMode` di RichTextEditor
   - Step through kode untuk melihat alur eksekusi

4. **Pemeriksaan Database**:
   - Verifikasi bahwa status halaman di database berubah sesuai harapan
   - Gunakan Network tab untuk melihat response dari API
   - Periksa payload request dan response

## 4. Protokol Debugging untuk Auto-save Draft

### 4.1 Identifikasi Masalah Auto-save

1. **Persiapan Monitoring**:
   - Filter log dengan string "useRichTextAutosave" atau "DraftFeedbackService"
   - Aktifkan Network tab dan filter untuk request ke endpoint `/api/module/*/pages/*/draft`

2. **Reproduksi Masalah**:
   - Buat perubahan pada editor
   - Tunggu 5 detik (waktu debounce default)
   - Amati apakah request auto-save dikirim
   - Periksa response dan status yang ditampilkan di UI

3. **Analisis Status Auto-save**:
   - Periksa nilai `draftSaveStatus` di context
   - Cek apakah status berubah dari 'idle' → 'saving' → 'saved'
   - Periksa apakah `lastSavedAt` diperbarui

### 4.2 Debugging Auto-save Step-by-Step

1. **Verifikasi Trigger Auto-save**:
   - Tambahkan log di event handler editor yang memicu auto-save
   - Periksa apakah debounce berfungsi dengan benar
   - Cek apakah throttle membatasi request sesuai konfigurasi

2. **Periksa Proses Penyimpanan**:
   - Tambahkan breakpoint di fungsi `forceSave` atau handler auto-save
   - Periksa payload yang dikirim ke server
   - Verifikasi bahwa content editor diambil dengan benar

3. **Analisis Response Server**:
   - Periksa response dari endpoint `/api/module/*/pages/*/draft`
   - Cek status code dan body response
   - Verifikasi bahwa response diproses dengan benar di client

4. **Pemeriksaan Status Draft di Database**:
   - Verifikasi bahwa `draftData` tersimpan di database
   - Periksa nilai `hasUnpublishedChanges` dan `draftSavedAt`

## 5. Debugging Integrasi RichTextEditor

### 5.1 Verifikasi Sinkronisasi Mode dan Content

1. **Periksa Konsistensi Mode**:
   - Log nilai `editorMode` dari context
   - Log nilai `editor.isEditable` dari instance editor
   - Verifikasi bahwa keduanya konsisten (edit mode → editable: true)

2. **Verifikasi Content yang Ditampilkan**:
   - Log hasil dari `getDraftOrPublishedContent`
   - Bandingkan dengan content yang ditampilkan di editor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EviewNicks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
