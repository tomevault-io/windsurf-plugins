---
trigger: always_on
description: Bu rehber, Claude Code (ve diğer AI destekli araçlar) ile bu depoda çalışırken doğru bağlamı kurmak için hazırlanmıştır. İnsan geliştiriciler için de geçerli bir mimari özetidir.
---

# CLAUDE.md — Bancho Cafe (Brew & Bloom) Kod Tabanı Rehberi

Bu rehber, Claude Code (ve diğer AI destekli araçlar) ile bu depoda çalışırken doğru bağlamı kurmak için hazırlanmıştır. İnsan geliştiriciler için de geçerli bir mimari özetidir.

---

## Komutlar

```bash
npm run dev          # Geliştirme sunucusu (tsx server.ts) — Vite HMR gömülü, :3000
npm run dev:full     # db:start + dev
npm run db:start     # Taşınabilir yerel MongoDB (.local-mongo/) başlat
npm run db:stop      # Yerel MongoDB'yi durdur
npm run db:clean     # Yerel MongoDB verisini sil (tehlikeli)
npm run lint         # tsc --noEmit — tip kontrolü (lint yok, bu bir alias)
npm test             # vitest run — birim testleri (tek seferlik)
npm run test:watch   # vitest — izleme modu
npm run build        # vite build → dist/
npm run docker:up    # docker compose up -d --build (app + mongodb + nginx)
```

- **Testler Vitest ile** çalışır (`vitest.config.ts`): `src/**/*.{test,spec}.ts` desenini tarar, node ortamı, globals açık.
- **"lint" aslında tip kontrolüdür** (`tsc --noEmit`); ESLint yapılandırması bağımlılıklarda mevcut ama `lint` script'ine bağlı değildir. CI (`.github/workflows/ci.yml`) npm ci → lint → test → build sırasını çalıştırır (Node 20).

## Mimari — Tek Süreçli Tam Yığın

`server.ts` tek giriş noktasıdır ve Express + Vite'ı **tek HTTP sunucusunda** birleştirir:

1. `dotenv.config()` → `JWT_SECRET` yoksa **anında çıkış** (kod 1).
2. `connectDatabase()` → Mongoose ile MongoDB'ye bağlanır (5 sn seçim zaman aşımı).
   - Geliştirme modunda başarısızsa: gömülü `MongoMemoryServer` yedek devreye girer (kalıcı değil, uyarılır).
   - Üretim modunda başarısızsa: **sistem çıkar** — yedek yok.
3. `checkAndSeedInitialData()` + `backfillNotificationEvents()` → ilk veri tohumlama (`autoSeed` servisi, `ENABLE_SEED` ile kontrol edilir; `BOOTSTRAP_ADMIN_*` env'lerinden yönetici hesabı açar).
4. Middleware zinciri (sıra önemlidir):
   - `trust proxy 1` (Nginx arkası gerçek IP)
   - **Helmet** — CSP *report-only* modda (SPA/kamera/QR akışlarını bozmamak için; raporlar temizlenince enforcing'e geçirilecek)
   - **CORS** — `ALLOWED_ORIGINS` (virgülle ayrılmış). Üretimde joker `*` veya boş liste → açılışta çıkış. Kimlik bilgileri kapalı (Bearer header modeli, çerez yok).
   - **express-rate-limit** — `/api/auth` 30 istek/15 dk; genel `/api` 600/15 dk.
   - `express.json({ limit: "10mb" })` (base64 görsel yüklemeleri için)
   - `/uploads` → `express.static(getUploadRoot())` (yönetilen görseller)
5. `/api` → `src/server/routes/api.js` (çekirdek) + içinden `new-features.js` mount edilir.
6. Geliştirme: Vite **middleware mode** (HMR; `DISABLE_HMR=true` ile kapatılır). Üretim: `express.static(dist)` + her şey `index.html`'e düşer (SPA fallback).

### Yol Haritası (route dosyaları)

- **`src/server/routes/api.ts` (~3350 satır)** — çekirdek uç noktalar:
  - Kimlik: `/api/auth/*` (login, register, me, logout, reset-password, session-role)
  - Katalog: `/api/products`, `/api/categories`, `/api/ingredients`
  - Sipariş: `/api/orders`, `/api/orders/:id/status`
  - Masa: `/api/tables`, `/api/table-sessions/*` (join-or-create, pay, poll-status, force-close…)
  - Sadakat: `/api/loyalty/qr`, `/api/loyalty/scan/*`
  - Bildirim/Push: `/api/notifications/*`, `/api/push/*` (VAPID)
  - Yönetim: `/api/users*`, `/api/staff*`, `/api/campaigns*`, `/api/analytics/dashboard`, `/api/uploads/image`, `/api/system/reset`, `/api/health`
- **`src/server/routes/new-features.ts` (~1400 satır)** — yeni modüller, `api.ts` içinden mount edilir:
  - Kuponlar (`/api/coupons*`), rezervasyon (`/api/reservations*`), envanter (`/api/inventory*`), sohbet (`/api/chat/*`), arkadaşlık (`/api/friends*`), hediyeler (`/api/gifts*`), değerlendirme (`/api/reviews*`), liderlik tablosu (`/api/leaderboard*`), garson çağrıları (`/api/waiter-calls*`), abonelikler (`/api/subscriptions*`), sistem metinleri (`/api/system-texts*`)

Yeni bir REST uç noktası eklerken: önce doğru dosyayı seç (çekirdek mi yeni modül mü), sonra `attachAuth`/`attachOptionalAuth` + `restrictTo("role")` zincirini uygula.

### Modeller (`src/server/models/`)

23 Mongoose şeması. Hepsi default export eder. Önemliler:

`User` (rol, sessionRole, sadakat puanları, bakiye), `Order`, `Product`, `Category`, `Ingredient`, `Campaign`, `Coupon`, `Staff`, `Table`, `TableSession` (ortak masa oturumu), `Reservation`, `WaiterCall`, `ChatRoom`, `Friend`, `Gift`, `Review`, `Subscription` (+ `SubscriptionPlanModel` named export), `InventoryItem`, `Notification`, `PushSubscription`, `SystemText` (panelden düzenlenebilir metinler), `ChangeLog`, `BalanceTopUp`.

Şemaları değiştirirken migration yok — autoSeed yalnızca eksik belge oluşturur, mevcut alanları güncellemez. Alan eklerken eski belgelerde `undefined` kalacağını hesaba kat.

### Servisler (`src/server/services/`)

| Servis | Sorumluluk |
|---|---|
| `autoSeed.ts` | İlk açılış veri tohumlama + yönetici bootstrap + bildirim geri-doldurma |
| `loyalty.ts` | Puan toplama/harcama, sadakat QR token üretimi/doğrulaması, kampanya ödülü uygulama |
| `coupon.ts` | Kupon doğrulama/kullanma (testli) |
| `pushNotification.ts` | Web Push gönderimi (kullanıcıya/siparişe/role/aboneliğe) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusuf-konuk/saas-coffee-react](https://github.com/yusuf-konuk/saas-coffee-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
