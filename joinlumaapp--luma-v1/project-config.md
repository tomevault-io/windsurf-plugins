---
trigger: always_on
description: LUMA is a compatibility-first social discovery platform that combines meaningful matching with social features. It merges the best of Instagram (social feed), Tinder (card swipe discovery), and Omegle (random live video) into a single dating & friendship app.
---

# LUMA V1 -- Project Instructions for Claude

## Project Overview
LUMA is a compatibility-first social discovery platform that combines meaningful matching with social features. It merges the best of Instagram (social feed), Tinder (card swipe discovery), and Omegle (random live video) into a single dating & friendship app.

**Core Philosophy:** "Gercek uyum icin kendin ol." -- Compatibility is the foundation. Every feature serves the goal of connecting compatible people.

### Locked V1 Architecture
- **5 Main Tabs**: Akis, Kesfet, Canli, Eslesme, Profil
- **3 Packages**: Ucretsiz, Premium, Supreme (NO Gold, Pro, or Reserved)
- **20+5 Questions**: 20 uyum (mandatory) + 5 kisilik (optional)
- **4 Relationship Types**: Takip, Arkadas, Eslesme, Super Begeni
- **5 Hedefler**: Evlenmek, Iliski bulmak, Sohbet/Arkadas, Kulturleri ogrenmek, Dunyayi gezmek
- **Jeton Economy**: In-app currency for premium actions
- **Uyum Score Range**: 47-97% (90+ = Super Uyum)
- **Photos**: Min 2, Max 9

## Tech Stack
- **Mobile**: React Native + TypeScript (apps/mobile/)
- **Backend**: Node.js + NestJS + TypeScript (apps/backend/)
- **Database**: PostgreSQL + Redis + Elasticsearch
- **Shared Types**: @luma/shared (packages/shared/)
- **Infrastructure**: AWS (Terraform in infrastructure/)
- **CI/CD**: GitHub Actions (.github/workflows/)
- **Real-time**: WebSocket (Socket.io) for messaging, live video, notifications
- **Video**: WebRTC for Canli (live) random video matching
- **Push Notifications**: Firebase Cloud Messaging (FCM)
- **Ads**: Google AdMob (rewarded ads for free users)

## Language Rules
- All code, comments, and technical documentation: **English**
- All user-facing strings and responses to the founder: **Turkish**
- Agent prompts are in English, output always in Turkish

## Code Standards
- TypeScript strict mode everywhere
- No `any` types
- All API endpoints must have input validation
- All business logic must have unit tests
- Follow NestJS module pattern for backend
- Follow React Navigation + Zustand for mobile
- Use Prisma ORM for database operations
- All shared types go in `packages/shared/`
- All API routes defined in `packages/shared/src/constants/api.ts`
- All WebSocket events defined in `packages/shared/src/constants/api.ts`

## Design Direction
- **Reference**: Bumpy dating app (animations, gradients, modern UI)
- Smooth card swipe transitions in Kesfet
- Konfeti/kalp animasyonu on match
- Gradient backgrounds (pink-to-peach light theme, purple-to-dark premium theme)
- Soft, rounded UI elements
- Micro-animations on interactions (like, follow, boost)

---

## APP ARCHITECTURE -- 5 Main Tabs

### Tab 1: Akis (Feed)
Instagram-like social feed with stories and posts.

**Top Section -- Stories:**
- Horizontal scrollable story circles
- First circle: user's own story with "+" to add new
- Only stories from matched/followed users visible
- Stories expire after 24 hours
- Story creation limits by package (Ucretsiz: 1/gun, Premium: 5/gun, Supreme: Sinirsiz)
- Supreme users' stories appear first (Hikaye Onde Gosterim)

**Post Creation:**
- "Gonderini paylas..." input area
- 3 post types: Fotograf, Video, Yazi
- Each post shows: user avatar, name, verification badge, city, time ago, hedef tag

**Feed Tabs:**
- **Populer**: Posts from nearby users sorted by distance + uyum score + engagement. Visible to everyone.
- **Takip**: Posts only from users you follow.

**Interactions on posts:**
- Begen (like with heart animation)
- Yorum (comment)
- Takip et (follow from post)
- Profili ziyaret et (visit profile)

### Tab 2: Kesfet (Discover)
Card-based swipe system for finding compatible people.

**Card Display:**
- Full-screen profile cards with photos
- Shows: name, age, city, uyum yuzdesi (compatibility %), hedef tag, kisilik tipi tag
- Swipe right = Begen, Swipe left = Pas gec
- Swipe up = Super Begeni (costs jeton)

**Top Right Controls:**
- Boost button (profilini one cikar)
- Filter button (yas, mesafe, cinsiyet, gelismis filtreler)

**Matching Logic:**
- Karsilikli begeni (mutual like) = Eslesme -> mesajlasma acilir
- Tek tarafli begeni -> Eslesme bolumunde "Begeniler" tabinda blurlu gorunur
- Begeni gorunurlugu: Ucretsiz (1-2 blurlu), Premium (sinirli net), Supreme (sinirsiz net)

**Filters (package-tiered):**
- Ucretsiz: Yas araligi, cinsiyet, mesafe (temel)
- Premium: + Ilgi alanlari, egitim, yasam tarzi
- Supreme: Tum filtreler acik + gelismis kombinasyonlar

### Tab 3: Canli (Live)
Omegle-style random video matching for instant connections.

**How It Works:**
1. User taps "Baglan"
2. System finds a compatible user (based on uyum score + filters)
3. Live video chat begins (jeton consumed)
4. At end of chat, options appear: Takip Et, Begen, Sonraki
5. Karsilikli takip = Arkadas olur

**Jeton Cost:**
- Each Canli session costs jeton
- Ucretsiz users get limited daily sessions
- Supreme users: unlimited

**Important:** Voice/video calls between matched/friended users happen in the MESSAGING section, NOT in Canli tab. Canli is ONLY for random discovery.

### Tab 4: Eslesme (Matches)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joinlumaapp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
