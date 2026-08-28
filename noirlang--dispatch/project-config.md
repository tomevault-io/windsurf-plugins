---
trigger: always_on
description: > Bu dosya, **Dispatch** projesinin tüm mimari kararlarını, özelliklerini, geliştirme kurallarını ve agent talimatlarını içerir. Projeye katkıda bulunan her agent ve geliştirici bu dosyayı referans almalıdır.
---

# AGENTS.md — Dispatch Project Blueprint

> Bu dosya, **Dispatch** projesinin tüm mimari kararlarını, özelliklerini, geliştirme kurallarını ve agent talimatlarını içerir. Projeye katkıda bulunan her agent ve geliştirici bu dosyayı referans almalıdır.

---

## 📌 Proje Genel Bakış

**Dispatch**, EmailWiz (Postfix + Dovecot) altyapısı üzerine inşa edilmiş, modern ve akıllı bir e-posta istemcisidir. Klasik e-posta özelliklerinin ötesinde; yapay zeka entegrasyonu, RSS akışları, akıllı takvim ve güvenlik odaklı özelliklerle donatılmıştır.

| Özellik | Değer |
|---|---|
| **Proje Adı** | Dispatch |
| **E-posta Altyapısı** | EmailWiz (Postfix + Dovecot) |
| **Backend** | Ruby on Rails 8 (API modu) |
| **Frontend** | Vite + React 18 + TypeScript |
| **Veritabanı** | PostgreSQL |
| **Cache / Queue** | Redis + Sidekiq |
| **IMAP Bridge** | mail_room gem |
| **Stil** | Tailwind CSS (siyah/beyaz tema) |
| **Test Ortamı** | Docker Compose (domain gerekmez) |

---

## 🏗️ Mimari

```
┌─────────────────────────────────────────────────────┐
│                     DISPATCH                         │
│                                                      │
│  ┌──────────┐    ┌────────────┐    ┌─────────────┐  │
│  │  React   │◄──►│ Rails API  │◄──►│  Postfix /  │  │
│  │  (Vite)  │    │  (JSON)    │    │   Dovecot   │  │
│  └──────────┘    └─────┬──────┘    └─────────────┘  │
│                        │                             │
│            ┌───────────┼───────────┐                 │
│            ▼           ▼           ▼                 │
│       PostgreSQL     Redis      Sidekiq              │
│                     (Cache)    (Workers)             │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │              AI Layer (Optional)             │   │
│  │   Gemini API | Claude API | OpenAI API       │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### E-posta Akışı

```
Gelen Mail (Postfix)
        │
        ▼
   mail_room (IMAP polling)
        │
        ▼
  ActionMailbox (Rails)
        │
        ▼
  DispatchMailboxRouter
        │
        ├── Yeni gönderici? ──► Approval Queue (IMAP: Approvals klasörü)
        │
        ├── Onaylı gönderici? ──► Inbox
        │
        ├── Speakeasy Code içeriyor? ──► Inbox (trusted)
        │
        └── Reddedilmiş? ──► Trash / Block
```

---

## 📁 Dizin Yapısı

```
dispatch/
├── emailwiz/                  # EmailWiz kurulum scripti (mevcut)
├── backend/                   # Ruby on Rails API
│   ├── app/
│   │   ├── mailboxes/         # ActionMailbox işleyicileri
│   │   ├── models/            # Email, Contact, Label, RssFeed, CalendarEvent...
│   │   ├── workers/           # Sidekiq workers (RSS, AI processing)
│   │   ├── services/
│   │   │   ├── ai/            # Gemini, Claude, OpenAI adaptörleri
│   │   │   ├── email/         # IMAP, SMTP, spy pixel proxy
│   │   │   └── rss/           # RSS fetcher servisi
│   │   └── controllers/api/v1/
│   ├── config/
│   └── Gemfile
├── frontend/                  # Vite + React
│   ├── src/
│   │   ├── views/
│   │   │   ├── Email/         # E-posta modülü (Thunderbird tarzı)
│   │   │   ├── Calendar/      # Dikey takvim modülü
│   │   │   ├── Feed/          # RSS akış modülü
│   │   │   ├── Dashboard/     # AI pano (notlar, kodlar, kargo vs.)
│   │   │   ├── Auth/          # Kayıt / Giriş (tam ekran)
│   │   │   └── Settings/      # Ayarlar sayfaları
│   │   ├── components/
│   │   └── store/             # Zustand state management
│   └── package.json
├── docker/
│   ├── docker-compose.yml     # Lokal test ortamı
│   ├── mailserver/            # docker-mailserver config
│   └── nginx/                 # Reverse proxy + image proxy config
├── AGENTS.md                  # Bu dosya
└── README.md
```

---

## 🎨 UI/UX Tasarım Sistemi

### Tema: Siyah/Beyaz Minimalist

```css
/* Renk Paleti */
--bg-primary:     #0a0a0a;   /* Ana arka plan */
--bg-secondary:   #111111;   /* Panel arka planı */
--bg-tertiary:    #1a1a1a;   /* Kart/hover arka planı */
--border:         #222222;   /* Kenarlıklar */
--text-primary:   #ffffff;   /* Ana metin */
--text-secondary: #999999;   /* İkincil metin */
--text-muted:     #444444;   /* Soluk metin */
--accent:         #ffffff;   /* Vurgu (beyaz buton) */
--accent-hover:   #e0e0e0;   /* Buton hover */
--danger:         #ff4444;   /* Silme / red */
--success:        #44ff88;   /* Onay / kabul */
--warning:        #ffaa00;   /* Uyarı / önemli */
```

### Navigasyon Yapısı

```
┌──────────────────────────────────────────────────────┐
│  📅 Calendar    │    📧 Email    │    📰 Feed (RSS)   │
│   (sol, ~25%)   │  (orta, ~50%) │    (sağ, ~25%)     │
└──────────────────────────────────────────────────────┘
```

- Üst menüden 3 ana bölüme geçiş yapılabilir
- Varsayılan görünüm: Email ortada büyük, diğerleri küçük panel
- Tam ekran modunda tek bölüm görünebilir

---

## 📧 E-posta Modülü

### Temel Özellikler (Thunderbird Benzeri)
- Gelen Kutusu, Gönderilenler, Taslaklar, Çöp Kutusu
- Mail okuma, yazma, yanıtlama, yönlendirme

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noirlang/dispatch](https://github.com/noirlang/dispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
