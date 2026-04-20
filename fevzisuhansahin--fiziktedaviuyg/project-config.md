---
trigger: always_on
description: Web tabanlı, uygulama indirmeye gerek duymayan, **tamamen istemci tarafında (Edge AI)** çalışan fizik tedavi egzersiz takip platformu. Video verisi **asla sunucuya gönderilmez**; cihazda işlenerek yalnızca biyomekanik JSON metrikleri iletilir.
---

# CLAUDE.md — Edge AI Fizik Tedavi Asistanı

## Proje Özeti

Web tabanlı, uygulama indirmeye gerek duymayan, **tamamen istemci tarafında (Edge AI)** çalışan fizik tedavi egzersiz takip platformu. Video verisi **asla sunucuya gönderilmez**; cihazda işlenerek yalnızca biyomekanik JSON metrikleri iletilir.

---

## Teknoloji Stack'i

### Frontend / Edge AI
- **Pose Estimation:** `@mediapipe/tasks-vision` → BlazePose (WorldLandmarks, 3D)
- **GPU Hızlandırma:** WebGPU (birincil) → WebGL 2.0 (fallback)
- **Threading:** Web Workers (inference main thread'den ayrı çalışır)
- **Wasm:** SIMD + Pthread optimizasyonları aktif
- **Biyomekanik:** Kosinüs teoremi ile ROM hesabı, DTW ile jitter giderme
- **Framework:** React + TypeScript + Vite

### Backend
- **Runtime:** Node.js (TypeScript) veya Python (FastAPI) — karar aşamasında
- **Realtime:** WebSocket (eş zamanlı izleme senaryoları için)
- **REST:** HTTP/2 + TLS 1.3
- **Standart:** HL7 FHIR R4 (Observation, Procedure resource modelleri)

### Veritabanı
- **İlişkisel:** PostgreSQL (hasta, klinisyen, seans kayıtları)
- **Zaman serisi:** TimescaleDB veya InfluxDB (ROM trend verileri)
- **Şifreleme:** AES-256 (data at rest)

### Güvenlik & Uyumluluk
- HIPAA + KVKK (Privacy by Design)
- TLS 1.3 (data in transit)
- Video frame'ler RAM'de işlendikten milisaniyeler içinde imha edilir
- Sunucuya **asla** piksel/video/fotoğraf gönderilmez

---

## Klasör Yapısı

```
/
├── CLAUDE.md                        ← Bu dosya
├── packages/
│   ├── client/                      ← React + TypeScript frontend
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── workers/
│   │   │   │   └── pose.worker.ts   ← MediaPipe inference (Web Worker)
│   │   │   ├── engine/
│   │   │   │   ├── poseEngine.ts    ← WebGPU/WebGL delegate yönetimi
│   │   │   │   ├── biomechanics.ts  ← ROM hesabı, kosinüs teoremi
│   │   │   │   ├── occlusion.ts     ← Visibility skoru & fallback mantığı
│   │   │   │   └── repCounter.ts    ← DTW tabanlı tekrar sayımı
│   │   │   ├── components/
│   │   │   │   ├── Camera/          ← Video akışı & canvas overlay
│   │   │   │   ├── ExerciseView/    ← Egzersiz UI
│   │   │   │   └── Dashboard/       ← Hasta özet ekranı
│   │   │   ├── hooks/
│   │   │   │   ├── usePoseDetection.ts
│   │   │   │   └── useWebSocket.ts
│   │   │   ├── transport/
│   │   │   │   └── wsClient.ts      ← JSON metrik gönderimi (WS/WebRTC)
│   │   │   └── types/
│   │   │       ├── biomechanics.ts
│   │   │       └── fhir.ts
│   │   └── vite.config.ts
│   │
│   ├── server/                      ← Backend API
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── sessions.ts      ← Seans CRUD
│   │   │   │   ├── metrics.ts       ← Biyomekanik veri alımı
│   │   │   │   ├── fhir.ts          ← FHIR Observation/Procedure endpoint'leri
│   │   │   │   └── clinician.ts     ← Klinisyen dashboard API
│   │   │   ├── ws/
│   │   │   │   └── metricsSocket.ts ← WebSocket sunucu tarafı
│   │   │   ├── db/
│   │   │   │   ├── schema.sql        ← PostgreSQL şeması
│   │   │   │   └── migrations/
│   │   │   ├── fhir/
│   │   │   │   ├── observation.ts   ← ROM → FHIR Observation builder
│   │   │   │   └── procedure.ts     ← Egzersiz → FHIR Procedure builder
│   │   │   └── security/
│   │   │       ├── encryption.ts    ← AES-256 yardımcıları
│   │   │       └── tls.ts
│   │   └── tsconfig.json
│   │
│   └── shared/                      ← Ortak tipler (client + server)
│       └── src/
│           ├── MetricPayload.ts     ← WebSocket/REST JSON şeması
│           └── FHIRTypes.ts
│
├── docs/
│   ├── architecture/
│   │   ├── system-diagram.md        ← Deliverable 1: Sistem mimarisi
│   │   └── sequence-diagrams.md     ← Deliverable 2: Sekans diyagramları
│   ├── database/
│   │   └── schema-design.md         ← Deliverable 3: DB şeması & FHIR modeli
│   └── edge-ai/
│       └── webgpu-workers-guide.md  ← Deliverable 4: WebGPU/Workers/Wasm kılavuzu
│
└── docker-compose.yml
```

---

## Kritik Mimari Kararlar (Değiştirilmez)

### 1. Video Asla Sunucuya Gönderilmez
```
Kamera → RAM (frame) → MediaPipe (inference) → JSON metrikler → Backend
                ↑
        Frame burada imha edilir
```
Sunucuya giden tek veri: `{ patientId, timestamp, metrics: { knee_flexion_deg, ... } }`

### 2. Web Worker Zorunluluğu
MediaPipe inference **mutlaka** `pose.worker.ts` içinde çalışmalıdır.
Main thread'de inference yapılması UI freeze'e neden olur → **kesinlikle yasaktır**.

### 3. WebGPU → WebGL Fallback Sırası
```typescript
// poseEngine.ts içinde bu sıra korunmalıdır
const delegate = await detectDelegate(); // 'GPU' | 'WEBGL' | 'CPU'
```
WebGPU yoksa WebGL 2.0, o da yoksa CPU fallback devreye girer.

### 4. Oklüzyon Güvenlik Sınırı
`visibility < 0.5` olan eklem noktaları için açı hesabı **durdurulur**, önceki geçerli değer korunur.
Halüsinasyon (hatalı açı) üretimi engellenmiş olur.

### 5. FHIR Uyumu
ROM verileri `FHIR Observation` kaynağına, egzersiz seansları `FHIR Procedure` kaynağına map'lenmelidir.
USBS / e-Nabız entegrasyonu bu standart üzerinden sağlanır.

---

## Veri Akışı (Özet)

```
[Kamera]
   │  video frame (RAM only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fevzisuhansahin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
