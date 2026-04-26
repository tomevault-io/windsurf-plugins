---
trigger: always_on
description: > **Konsep:** Photobooth online dengan AI style transformation (Ghibli, Disney, Anime, dll).
---

@AGENTS.md

# 📸 AI Photobooth — Implementation Plan

> **Konsep:** Photobooth online dengan AI style transformation (Ghibli, Disney, Anime, dll).
> Kompetitor utama: Fremio.id. Unique value: foto langsung bisa di-transform ke berbagai art style via AI.

---

## 🗺️ Overview Arsitektur

```
Frontend (Next.js 14 App Router)
├── Halaman utama & landing
├── Photobooth Studio (capture + AI transform + frame)
└── Gallery & sharing

Backend (Next.js API Routes)
├── /api/transform   → AI image style transfer
├── /api/generate-frame  → AI frame generation
└── /api/upload      → Handling gambar

External Services
├── Google Gemini API (image-to-image style transfer)
├── Cloudinary atau Vercel Blob (image storage)
└── Vercel (deployment)
```

---

## 🛠️ Tech Stack

| Layer | Teknologi | Alasan |
|---|---|---|
| Framework | Next.js 14 (App Router) | SSR, API routes, file-based routing |
| Styling | Tailwind CSS + shadcn/ui | Cepat, konsisten, composable |
| AI Image | Google Gemini 2.0 Flash (`gemini-2.0-flash-preview-image-generation`) | Image-to-image support, $0.0195/output |
| Image Storage | Vercel Blob atau Cloudinary | Mudah integrasi dengan Next.js |
| Kamera | Browser MediaDevices API | Native, no library needed |
| Canvas | HTML5 Canvas API | Compositing foto + frame |
| State | Zustand | Ringan, cocok untuk wizard flow |
| Deploy | Vercel | Native Next.js support |

---

## 📁 Struktur Folder

```
ai-photobooth/
├── src/app/
│   ├── page.tsx                    # Landing page
│   ├── studio/
│   │   └── page.tsx                # Main photobooth studio
│   ├── gallery/
│   │   └── page.tsx                # Galeri hasil foto
│   └── api/
│       ├── transform/
│       │   └── route.ts            # AI style transformation endpoint
│       ├── generate-frame/
│       │   └── route.ts            # AI frame generation endpoint
│       └── upload/
│           └── route.ts            # Upload & storage
│
├── components/
│   ├── studio/
│   │   ├── CameraCapture.tsx       # Komponen kamera
│   │   ├── StyleSelector.tsx       # Pilih AI style
│   │   ├── FramePicker.tsx         # Pilih / generate frame
│   │   ├── PhotoCanvas.tsx         # Canvas compositing
│   │   ├── TransformProgress.tsx   # Loading state AI
│   │   └── DownloadShare.tsx       # Download & share
│   ├── landing/
│   │   ├── Hero.tsx
│   │   ├── FeatureShowcase.tsx
│   │   └── StylePreview.tsx
│   └── ui/                         # shadcn components
│
├── lib/
│   ├── gemini.ts                   # Gemini API client & helpers
│   ├── canvas.ts                   # Canvas compositing utilities
│   ├── styles.ts                   # Style definitions & prompts
│   └── storage.ts                  # Image upload/retrieve
│
├── store/
│   └── studio.ts                   # Zustand store untuk studio flow
│
├── public/
│   └── frames/                     # Default frame templates (PNG transparan)
│
└── types/
    └── index.ts                    # TypeScript types
```

---

## 🎯 Fase Development

### Fase 1 — Core Photobooth (Target: bisa foto & download)

**Goal:** User bisa buka kamera, foto, pilih frame, download. Belum ada AI.

#### 1.1 Setup Project

```bash
npx create-next-app@latest ai-photobooth --typescript --tailwind --app
cd ai-photobooth
npx shadcn@latest init
npx shadcn@latest add button card badge slider tabs
npm install zustand
npm install @google/generative-ai
```

#### 1.2 Landing Page (`app/page.tsx`)

Buat landing page dengan:
- Hero section: tagline + CTA button "Mulai Foto"
- Preview beberapa hasil AI transform (before/after)
- Section fitur utama
- Navigasi ke `/studio`

#### 1.3 Studio Layout (`app/studio/page.tsx`)

Wizard flow dengan 4 step:
```
Step 1: Capture  →  Step 2: AI Style  →  Step 3: Frame  →  Step 4: Download
```

Gunakan Zustand store untuk state antar step:

```typescript
// store/studio.ts
interface StudioStore {
  currentStep: 1 | 2 | 3 | 4;
  capturedPhoto: string | null;       // base64 original
  transformedPhoto: string | null;    // base64 after AI
  selectedStyle: AIStyle | null;
  selectedFrame: Frame | null;
  finalPhoto: string | null;          // composited result
  
  setStep: (step: number) => void;
  setCapturedPhoto: (photo: string) => void;
  setTransformedPhoto: (photo: string) => void;
  setSelectedStyle: (style: AIStyle) => void;
  setSelectedFrame: (frame: Frame) => void;
  setFinalPhoto: (photo: string) => void;
  reset: () => void;
}
```

#### 1.4 Komponen Kamera (`components/studio/CameraCapture.tsx`)

```typescript
// Fitur yang perlu diimplementasi:
// - Akses kamera via navigator.mediaDevices.getUserMedia
// - Preview live kamera dalam <video> element
// - Flip kamera (front/back) untuk mobile
// - Tombol capture → gambar ke canvas → simpan sebagai base64
// - Atau: upload dari galeri sebagai alternatif
// - Countdown timer 3-2-1 sebelum capture (opsional, tapi keren)
```

Spesifikasi teknis:
- Resolusi target: 1080x1080 atau 3:4 (portrait photobooth style)
- Mirror mode untuk selfie (transform: scaleX(-1))
- Error handling jika kamera tidak diizinkan

#### 1.5 Canvas Compositing (`lib/canvas.ts`)

```typescript
// Fungsi yang perlu dibuat:
// - composePhotoWithFrame(photoBase64, frameBase64): Promise<string>
//   Overlay frame transparan (PNG) di atas foto

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asrafmi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
