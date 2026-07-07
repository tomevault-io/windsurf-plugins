---
trigger: always_on
description: **Project**: Multi-location tire shop POS and management system with React + TypeScript + Firebase
---

# Copilot Instructions for tire-plan

**Project**: Multi-location tire shop POS and management system with React + TypeScript + Firebase

## Architecture Overview

### Tech Stack
- **Frontend**: React 19 + TypeScript + Vite + TailwindCSS
- **Backend**: Firebase (Firestore, Authentication, Cloud Functions)
- **State Management**: React hooks + Firebase Realtime Listeners
- **Charts**: Recharts
- **Styling**: TailwindCSS + Tailwind Merge

### Project Structure
```
src/
├── components/      # Feature-based components (Dashboard, POS, Inventory, etc)
├── utils/          # Firestore service layer, formatting utilities
├── hooks/          # Custom React hooks (useMenuAccess)
├── types.ts        # Centralized TypeScript interfaces
├── firebase.ts     # Firebase initialization
└── App.tsx         # Main app with role-based routing (~2566 lines)

functions/
├── auth.ts         # Firebase Auth functions
├── index.ts        # Cloud Functions entry
└── subscription.ts # Subscription management
```

## Critical Patterns

### 1. Type Imports vs Value Imports
```typescript
// ✅ Types use 'type' keyword
import type { Sale, Customer, Shift } from './types';

// ✅ Runtime values (enums, constants) don't use 'type'
import { PaymentMethod } from './types';
```
**Why**: PaymentMethod is an object constant used at runtime for checks like `s.paymentMethod === PaymentMethod.CARD`. Other interfaces are pure types for compile-time only.

### 2. Firestore Real-time Data Flow
Most list data (sales, shifts, staff) use `subscribeToQuery()` with `onSnapshot()` listeners:
```typescript
const unsub = subscribeToQuery<Shift>(COLLECTIONS.SHIFTS, constraints, (data) => {
  const sorted = [...data].sort((a, b) => ...);
  setShifts(sorted);
});
// Auto-unsubscribe on cleanup
return () => unsub?.();
```
**Key**: Date range queries in `App.tsx` control which data loads. Calendar and schedule views request different date ranges dynamically.

### 3. Date Handling: Timezone Pitfalls
ISO string dates can cause midnight shifts when converting to Date objects:
```typescript
// ✅ Safe: Extract date part directly from ISO string
const isoToLocalDate = (iso: string) => iso.split('T')[0];  // Returns "2026-01-03"

// ❌ Risky: new Date("2026-01-03") creates midnight UTC, not local
```
**Apply to**: Any date comparison in filters (src/components/ScheduleAndLeave.tsx, Dashboard.tsx)

### 4. Firestore Datetime Fields Store ISO Strings
Sales, shifts, and leave requests store `date: string` in ISO format ("2026-01-03T12:00:00Z").
- Always normalize with `isoToLocalDate()` before comparing
- Use `dateToLocalString()` for creating new dates: `new Date(...).toISOString()`

### 5. Sale Item Calculations
Each `Sale` has `items: SalesItem[]` where each item has `quantity: number`:
```typescript
// Sum tire quantities across all items in a sale
const tireQuantity = sale.items?.reduce((sum, item) => sum + item.quantity, 0) || 0;
```
**Used in**: Dashboard tire count displays, inventory tracking

### 6. Role-Based Access Control
Three roles with distinct permissions:
- `SUPER_ADMIN` (ID: '999999'): Master user, all features
- `STORE_ADMIN` (Owner): Own store's data, staff management
- `STAFF`: View-only, single store assigned

Implemented in `App.tsx` via conditional routing and in components via `useMenuAccess()` hook.

### 7. Multi-Store Context
Each sale, shift, and inventory record has `storeId`. Dashboard and reports filter by `selectedStoreId`. When `selectedStoreId === 'ALL'`, data aggregates across all visible stores.

## Development Workflows

### Start Development
```bash
npm run dev        # Vite server on localhost:5173
```
Hot Module Replacement auto-refreshes on file changes.

### Build & Deploy
```bash
npm run build      # TypeScript + Vite build
npm run lint       # ESLint check
```
Deployment uses GitHub Actions to Lightsail (see `LIGHTSAIL_DEPLOYMENT_GUIDE.md`).

### Testing
End-to-end tests use Playwright:
```bash
scripts/e2e-login-test.js  # Login flow verification
```

## Deployment Workflow (CRITICAL - READ FIRST)

### ⚠️ NEVER Skip These Steps

**배포 전 필수 체크리스트:**
1. ✅ **로컬 빌드 먼저 실행**: `npm run build` (서버에서 빌드하지 말 것!)
2. ✅ **dist 폴더 확인**: 빌드 후 `dist/assets/` 파일명 해시 변경 확인
3. ✅ **GitHub Actions 우선**: 수동 SSH 배포는 최후의 수단
4. ✅ **변경사항만 배포**: `deploy-to-lightsail.sh`는 이미 최적화됨 (dist만 전송)

### Deployment Methods

#### 1️⃣ **GitHub Actions (권장 방법)**
```bash
# 코드 변경 후 커밋만 하면 자동 배포
git add .
git commit -m "feat: 기능명"
git push origin main
# → GitHub Actions가 자동으로 빌드 + 배포
```

**워크플로우 위치**: `.github/workflows/deploy-lightsail.yml`
- 자동 빌드 (npm run build)
- SSH 키로 Lightsail 접속 (GitHub Secrets 저장됨)
- dist 폴더만 전송 (60초 이내 완료)
- nginx 자동 재시작

**GitHub Secrets 등록 상태** (이미 설정 완료):
- `LIGHTSAIL_SSH_KEY`: SSH private key (~/Downloads/LightsailDefaultKey-ap-northeast-2.pem)
- `LIGHTSAIL_HOST`: 52.78.72.19
- `LIGHTSAIL_USER`: ubuntu

#### 2️⃣ **수동 배포 (긴급 시에만)**
```bash
# Step 1: 로컬 빌드 (필수!)
npm run build

# Step 2: 배포 스크립트 실행
bash deploy-to-lightsail.sh 52.78.72.19 ~/Downloads/LightsailDefaultKey-ap-northeast-2.pem
```

**⚠️ 배포 스크립트는 이미 최적화됨:**
- ✅ 로컬에서 사전 빌드된 dist만 전송
- ✅ SSH 호스트 키 검증 비활성화 (`StrictHostKeyChecking=no`)
- ✅ 서버에서 npm install/build 실행 안 함 (시간 절약)
- ✅ PM2 자동 재시작 (또는 nginx 서빙)

### SSH Key 정보 (인식 필수)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mytireplan/tire-plan](https://github.com/mytireplan/tire-plan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
