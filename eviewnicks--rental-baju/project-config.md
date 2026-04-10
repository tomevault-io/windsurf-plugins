---
trigger: always_on
description: Dokumen ini berisi panduan standar untuk format dokumentasi hasil implementasi task di project Maguru. Panduan ini mengacu pada best practice JIRA Agile Scrum dan memastikan konsistensi pelaporan hasil pengembangan.
---


# Panduan Dokumentasi Hasil (result-ops-xxx.md)

## Tujuan Panduan

Dokumen ini berisi panduan standar untuk format dokumentasi hasil implementasi task di project Maguru. Panduan ini mengacu pada best practice JIRA Agile Scrum dan memastikan konsistensi pelaporan hasil pengembangan.

# [OPS-XXX] Hasil Implementasi [Nama Fitur]

**Status**: [🟢 Complete | 🟡 Partial | 🔴 Blocked]  
**Diimplementasikan**: [Tanggal Mulai] - [Tanggal Selesai]  
**Developer**: [Nama Developer]  
**Reviewer**: [Nama Reviewer]  
**PR**: [Link Pull Request]

---

## Daftar Isi

1. [Ringkasan Implementasi](mdc:#ringkasan-implementasi)
2. [Perubahan dari Rencana Awal](mdc:#perubahan-dari-rencana-awal)
3. [Status Acceptance Criteria](mdc:#status-acceptance-criteria)
4. [Detail Implementasi](mdc:#detail-implementasi)
5. [Kendala dan Solusi](mdc:#kendala-dan-solusi)
6. [Rekomendasi Selanjutnya](mdc:#rekomendasi-selanjutnya)

## Ringkasan Implementasi (opsional Jika berhbungan dengan pembuatan fitur)

[Ringkasan singkat (3-5 kalimat) tentang fitur yang diimplementasikan, highlight utama, dan nilai bisnis yang dihasilkan]

### Ruang Lingkup

[Deskripsi singkat tentang apa yang tercakup dalam implementasi dan apa yang tidak]

#### 1. React Components

**Server Components**:

- [Nama Komponen]: [Deskripsi singkat]
- ...

**Client Components**:

- [Nama Komponen]: [Deskripsi singkat]
- ...

#### 2. State Management

**Context Providers**:

- [Nama Context]: [Deskripsi singkat]
- ...

**React Query/State**:

- [Nama Hook/Query]: [Deskripsi singkat]
- ...

#### 3. Custom Hooks

**Feature Hooks**:

- [Nama Hook]: [Deskripsi singkat]
- ...

**Utility Hooks**:

- [Nama Hook]: [Deskripsi singkat]
- ...

#### 4. Data Access

**Adapters**:

- [Nama Adapter]: [Deskripsi singkat]
- ...

**API Endpoints**:

- `[METHOD] /api/[path]` - [Deskripsi singkat]
- ...

#### 5. Server-side

**Services**:

- [Nama Service]: [Deskripsi singkat]
- ...

**Database Schema**:

- [Nama Model/Perubahan]: [Deskripsi singkat]
- ...

#### 6. Cross-cutting Concerns

**Types**:

- [Nama Type]: [Deskripsi singkat]
- ...

**Utils**:

- [Nama Util]: [Deskripsi singkat]
- ...

## Perubahan dari Rencana Awal

[Deskripsi perubahan signifikan dari rencana awal (task-ops-xxx.md) dan justifikasinya]

### Perubahan Desain

| Komponen/Fitur  | Rencana Awal | Implementasi Aktual | Justifikasi |
| --------------- | ------------ | ------------------- | ----------- |
| [Nama Komponen] | [Deskripsi]  | [Deskripsi]         | [Alasan]    |
| [API Endpoint]  | [Deskripsi]  | [Deskripsi]         | [Alasan]    |

### Perubahan Teknis

| Aspek           | Rencana Awal | Implementasi Aktual | Justifikasi |
| --------------- | ------------ | ------------------- | ----------- |
| [Struktur Data] | [Deskripsi]  | [Deskripsi]         | [Alasan]    |
| [Teknologi]     | [Deskripsi]  | [Deskripsi]         | [Alasan]    |

## Status Acceptance Criteria

| Kriteria     | Status | Keterangan                  |
| ------------ | ------ | --------------------------- |
| [Kriteria 1] | ✅     | [Keterangan jika perlu]     |
| [Kriteria 2] | ⚠️     | [Keterangan keterbatasan]   |
| [Kriteria 3] | ❌     | [Alasan tidak implementasi] |

## Detail Implementasi

> **⚠️ PENTING**: Dokumentasi ini harus fokus pada detail implementasi yang jelas dan ringkas. **HINDARI MENAMPILKAN PSEUDOCODE ATAU IMPLEMENTASI KODE YANG RUMIT**. Berikan penjelasan tingkat tinggi tentang pendekatan yang diambil, pola yang digunakan, dan alasan di balik keputusan teknis. Pengecualian hanya untuk Schema Database dan TypeScript Interface yang perlu ditampilkan sebagaimana diimplementasikan.

### Arsitektur Folder

Implementasi mengikuti struktur folder standar yang didefinisikan dalam arsitektur Maguru:

```
/features/[feature-name]/
├── components/         # Komponen React
│   ├── ComponentName/  # Komponen dengan sub-komponen
│   │   └── ...
│   └── ...
├── context/            # React Context Providers
│   └── ...
├── hooks/              # Custom React Hooks
│   ├── feature/        # Feature-specific hooks
│   └── shared/         # Shared hooks
├── adapters/           # Data access adapters (API calls)
│   └── ...
├── services/           # Business logic
│   └── ...
├── types/              # TypeScript type definitions
│   └── ...
└── utils/              # Utility functions
    └── ...
```

> **Catatan**: Jika implementasi Anda menyimpang dari struktur standar di atas, jelaskan alasannya di sini.

### Komponen Utama

#### [Nama Komponen]

**File**: `/path/to/component.tsx`

**Deskripsi**:
[Deskripsi fungsi dan tanggung jawab komponen]

**Pattern yang Digunakan**:

- [Pattern 1]: [Deskripsi]
- [Pattern 2]: [Deskripsi]

### Alur Data

[Jelaskan alur data dari API ke UI atau sebaliknya dengan fokus pada:

1. Bagaimana data dimuat dari backend
2. Bagaimana state dikelola (React Query, Context, dll)
3. Bagaimana state mempengaruhi UI
4. Bagaimana mutasi/update data dilakukan]

### Database Schema

[Jika ada perubahan skema database, jelaskan di sini]

```prisma
model UpdatedModel {
  // Skema yang diimplementasikan
}
```

### API Implementation

#### [Endpoint URL]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EviewNicks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
