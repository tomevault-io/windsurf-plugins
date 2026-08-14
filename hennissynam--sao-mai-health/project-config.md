---
trigger: always_on
description: Đọc file này TRƯỚC KHI làm bất cứ điều gì. Nó chứa mọi ngữ cảnh cần thiết để làm việc hiệu quả trên codebase này.
---

# CLAUDE.md — Sao Mai Health

Đọc file này TRƯỚC KHI làm bất cứ điều gì. Nó chứa mọi ngữ cảnh cần thiết để làm việc hiệu quả trên codebase này.

---

## Sản phẩm là gì

**Sao Mai Health** là nền tảng giám sát dịch tễ học và sức khỏe cộng đồng.

**Định vị thương mại đã chốt:** B2G platform bán cho CDC/Sở Y tế cấp tỉnh — KHÔNG phải consumer health app.

**Ba trụ cột tạo ra doanh thu:**
1. **Disease Surveillance Intelligence** — bản đồ ca bệnh + hotspot detection + báo cáo mẫu Bộ Y tế → bán cho Sở Y tế
2. **Fast Case Intake (30 giây)** — nhập ca bệnh offline-first → adoption driver cho nhân viên y tế tuyến cơ sở
3. **Environmental Stroke Risk** — AQI + GPS + thời tiết → rủi ro đột quỵ thời gian thực → differentiator với bảo hiểm và bệnh viện tư

**Flywheel logic:** FIELD (case intake miễn phí) → tạo data → RADAR (Sở Y tế trả tiền) → justify CLINIC (phòng khám trả tiền).

---

## Stack kỹ thuật

```
Frontend:  React 18 + TypeScript + Vite + Tailwind + shadcn/ui + Zustand
Backend:   Supabase (PostgreSQL + Auth + Realtime + Edge Functions)
Maps:      Mapbox GL + Leaflet + react-leaflet-cluster
Charts:    Recharts
3D:        Three.js + React Three Fiber
Offline:   Dexie (IndexedDB)
i18n:      i18next (vi + en)
```

**Dev commands:**
```bash
npm install    # cài dependencies
npm run dev    # dev server tại localhost:8080
npm run build  # production build
npm run lint   # ESLint check
```

**Supabase client:** `src/integrations/supabase/client.ts` — dùng `import { supabase } from "@/integrations/supabase/client"`.

---

## Cấu trúc thư mục quan trọng

```
src/
├── pages/              # Route pages — mỗi file = 1 trang
├── components/
│   ├── ui/            # shadcn/ui primitives — KHÔNG sửa trực tiếp
│   ├── biovault/      # Digital Twin components
│   ├── stroke/        # Stroke risk components
│   ├── surveillance/  # Map + hotspot components
│   └── dashboard/     # Dashboard widgets
├── hooks/             # 50+ custom hooks — logic nghiệp vụ ở đây
├── services/
│   ├── healthAPI.ts   # API wrapper + type definitions
│   └── dbService.ts   # DB-first service layer — dùng khi record user events
├── integrations/supabase/
│   ├── client.ts      # Supabase client (anon key)
│   └── types.ts       # Auto-generated DB types — KHÔNG sửa tay
└── i18n/locales/      # vi.json + en.json — thêm keys vào đây khi cần text mới
supabase/functions/    # 30+ Edge Functions (Deno/TypeScript)
```

---

## Patterns bắt buộc dùng

### 1. Supabase calls
```typescript
// Luôn import từ đây
import { supabase } from "@/integrations/supabase/client"

// RPC call
const { data, error } = await supabase.rpc('function_name', { param: value })
if (error) throw error

// Table insert
const { error } = await supabase.from('table_name').insert({ ... })
if (error) throw error

// Lấy user hiện tại
const { data: { user } } = await supabase.auth.getUser()
```

### 2. User event tracking (audit trail)
Dùng `dbService.ts` khi ghi user action — KHÔNG dùng console.log:
```typescript
import { recordEvent } from '@/services/dbService'
await recordEvent({ actionType: 'feature_use', payload: { feature: 'case_intake' } })
```

### 3. Offline-first pattern
```typescript
const { queueRecord, isOnline } = useOfflineStorage()
if (isOnline) {
  await supabase.rpc(...)  // online: lưu thẳng
} else {
  await queueRecord('table', 'insert', data)  // offline: queue
}
```

### 4. Toast notifications
```typescript
import { toast } from "sonner"
toast.success("Thành công")
toast.error("Lỗi: " + error.message)
```

### 5. Form validation
```typescript
import { z } from "zod"
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
// Định nghĩa schema → useForm với zodResolver → Form component từ shadcn
```

### 6. i18n
```typescript
import { useTranslation } from 'react-i18next'
const { t } = useTranslation()
// Dùng: t('nav.dashboard')
// Thêm key vào: src/i18n/locales/vi.json VÀ src/i18n/locales/en.json
```

---

## Database — bảng quan trọng nhất

| Bảng | Mục đích |
|------|---------|
| `case_events` | Ca bệnh báo cáo — source of truth cho surveillance |
| `daily_counts` | Thống kê ca bệnh theo ngày/quận/loại |
| `disease_hotspots` | Cụm bệnh được phát hiện bởi AI |
| `alerts` | Cảnh báo ngưỡng ca bệnh |
| `health_facilities` | Cơ sở y tế |
| `biovault_metrics` | Chỉ số sức khỏe cá nhân (lab results, sensors) |
| `user_events` | Audit trail user actions — `{ user_id, action_type, payload }` |
| `health_records` | Hồ sơ sức khỏe |
| `campaigns` | Chiến dịch tiêm chủng/vật tư |

**RPCs quan trọng:**
- `intake_case_fast(p_full_name, p_gender, p_birth_year, p_disease_code, p_facility_id, p_ward_id, p_district_id, p_onset_date, p_report_date, p_status, p_symptoms, p_lat, p_lng, p_mpi_hash, p_address_hash, p_phone_hash)` → ghi ca bệnh nhanh
- `metrics_json(in_date, in_metric)` → KPI metrics
- `get_points_available(p_user)` — điểm loyalty (ít dùng)

**Hashing PII (CCCD, phone, address):**
```typescript
btoa(unescape(encodeURIComponent(value))).slice(0, 32)
```

---

## Supabase Edge Functions — khi nào dùng cái nào

| Function | Dùng khi |
|---------|---------|
| `fetch-hcmc-data` | Lấy dữ liệu ca bệnh HCMC |
| `health-data-synthesis` | Tổng hợp KPI dashboard |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HENNISSYNAM/sao-mai-health](https://github.com/HENNISSYNAM/sao-mai-health) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
