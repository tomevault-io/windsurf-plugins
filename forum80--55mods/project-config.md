---
trigger: always_on
description: Oyun modu paylaşım platformu. Kullanıcılar mod yükler, indirir, yorum yapar; creator'lar stüdyo üzerinden içerik yönetir; adminler moderasyon ve site yönetimi yapar.
---

# 55Mods – Project Brain (CLAUDE.md)

Oyun modu paylaşım platformu. Kullanıcılar mod yükler, indirir, yorum yapar; creator'lar stüdyo üzerinden içerik yönetir; adminler moderasyon ve site yönetimi yapar.

---

## Tech Stack

### Backend
- **PHP 8.2+** — Custom DDD mimarisi (Laravel DEĞİL)
- **MySQL 8.0** — InnoDB, utf8mb4, veritabanı adı: `55mods`
- **JWT** — `firebase/php-jwt` ile access + refresh token sistemi
- **PSR-7** — `nyholm/psr7` HTTP mesaj katmanı
- **vlucas/phpdotenv** — Ortam değişkenleri

### Frontend
- **Vue 3.3** — Composition API, `<script setup>` syntax
- **TypeScript 5.3**
- **Vite 5** — Build tool, output: `../public`
- **Pinia 2** — State management
- **Vue Router 4** — 95+ route
- **Tailwind CSS 3.3** — Ana CSS framework (dinamik tema sistemi var)
- **Headless UI** — Erişilebilir unstyled bileşenler
- **Heroicons** — İkon sistemi (FontAwesome KULLANMA, kaldırılıyor)
- **Axios** — API iletişimi, 30s timeout, otomatik token refresh
- **Chart.js** — Admin dashboard grafikleri
- **DOMPurify** — HTML sanitizasyonu
- **vuedraggable** — Drag & drop
- **vite-plugin-pwa** — PWA + Service Worker

### Geliştirme Ortamı
- **Docker Compose** — MySQL + PHP/Apache + phpMyAdmin
  - MySQL: `localhost:3306`
  - PHP/Apache: `localhost:8080`
  - phpMyAdmin: `localhost:8081`
- **Vite Dev Server** — `localhost:5173` (frontend)
- **Context7 MCP** — Güncel kütüphane dokümantasyonu

---

## Geliştirme Ortamı Kurulumu

### Docker (MySQL + PHP Backend)
```bash
# 1. Docker Desktop kur: https://www.docker.com/products/docker-desktop/
# 2. Servisleri başlat
docker compose up -d

# 3. XAMPP'tan DB'yi taşı
bash scripts/db-export.sh
# Çıktıdaki komutu çalıştır

# 4. .env güncelle (DB_HOST=mysql, docker için)
```

### Frontend (Vite)
```bash
cd frontend
npm install
npm run dev   # localhost:5173
```

### XAMPP ile çalışmaya devam (geçiş dönemi)
```
DB_HOST=localhost
DB_PORT=3306
APP_URL=http://localhost/55Mods
```

---

## Proje Yol Haritası

### PHASE 0: Altyapı & Araçlar
| Görev | Durum | Notlar |
|-------|-------|--------|
| CLAUDE.md oluşturuldu | ✅ Tamamlandı | |
| Context7 MCP kuruldu | ✅ Tamamlandı | ~/.claude/settings.json |
| Docker Compose hazırlandı | ✅ Tamamlandı | 3 servis: mysql, php, phpmyadmin |
| .env güvenlik temizliği | ✅ Tamamlandı | SSH bilgileri kaldırıldı |
| .env.example güncellendi | ✅ Tamamlandı | |
| DB export scripti | ✅ Tamamlandı | scripts/db-export.sh |
| Docker Desktop kurulumu | ✅ Tamamlandı | v29.3.1 |
| XAMPP→Docker DB geçişi | ✅ Tamamlandı | 3.1MB, tüm tablolar aktarıldı |

### PHASE 1: i18n Sistemi Yenileme ✅ Tamamlandı (2026-03-28)
| Görev | Durum | Notlar |
|-------|-------|--------|
| vue-i18n v9 kurulumu | ✅ Tamamlandı | `npm install vue-i18n@9` |
| Mevcut JSON dosyaları dönüştürme | ✅ Tamamlandı | public/languages/ — format zaten uyumlu, taşıma gerekmedi |
| Custom store/composable kaldırma | ✅ Tamamlandı | 485 satır custom impl → vue-i18n wrapper (~100 satır) |
| main.ts: plugin + initI18n() | ✅ Tamamlandı | `useLocaleStore` kaldırıldı, `initI18n()` kullanılıyor |
| utils/dateHelper.ts güncelleme | ✅ Tamamlandı | `useLocaleStore()` → `i18n.global` |
| TypeScript tip güvenliği | ✅ Tamamlandı | i18n kaynaklı 0 hata, build başarılı |
| Tolgee kurulumu (opsiyonel) | ⬜ Bekliyor | Görsel çeviri editörü |

#### Phase 1 Mimari Kararları
- `src/plugins/i18n.ts` — vue-i18n instance, `loadLocale()`, `setLocale()`, `initI18n()`
- `src/composables/useI18n.ts` — ince wrapper; eski API birebir korundu (t, te, tm, locale, setLocale, formatNumber, formatDate, formatRelativeTime)
- `src/stores/i18n.ts` — geriye-uyumluluk re-export'u, silinmedi
- JSON dosyaları `public/languages/` — dinamik fetch, bundle'a dahil değil
- Fallback: EN → TR sırasıyla yüklenir

### PHASE 2: Kalite Kontrolü — Frontend ✅ Tamamlandı (2026-03-28)
| # | Parça | Durum | Notlar |
|---|-------|-------|--------|
| 2.1 | Auth & Güvenlik | ✅ Tamamlandı | JWT, guards, token refresh — solid, değişiklik gerekmedi |
| 2.2 | Tema Sistemi | ✅ Tamamlandı | Tailwind CSS vars + API entegreli theme store — solid |
| 2.3 | Frontend Mimari | ✅ Tamamlandı | 0 TS hatası: User.stats, ApiResponse.meta, img callable, DataWidgets, WeeklyChoice, vb. |
| 2.4 | Mod Sistemi | ✅ Tamamlandı | fetchComments → api.mods.getComments(); pagination meta fix |
| 2.5 | WIP Sistemi | ✅ Tamamlandı | Backend routes doğrulandı, 0 TS hatası |
| 2.6 | Pack Sistemi | ✅ Tamamlandı | hasMoreComments pagination fix; 0 TS hatası |
| 2.7 | Oyun/Kategori/Marka | ✅ Tamamlandı | 0 TS hatası, solid |
| 2.8 | Kullanıcı & Sosyal | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.9 | Creator Studio | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.10 | Admin Paneli | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.11 | Arama & Keşif | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.12 | Chat & Mesajlaşma | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.13 | Bildirimler | ✅ Tamamlandı | 0 TS hatası — solid |

#### Phase 2.3 Yapılanlar (2026-03-28)
- `types/index.ts`: User.stats, User.points, User.stats.total_mods/packs/wips, ApiResponse.meta, ModRequest.comment_count, ModCollection.collection_type genişletildi, ModCollection.tags, WeeklyWinner.entity_title/category/nominee

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forum80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
