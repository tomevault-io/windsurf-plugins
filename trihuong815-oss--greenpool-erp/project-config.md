---
trigger: always_on
description: > Auto-updated: 2026-06-11 | Người dùng: Nguyễn Văn Hướng (QUẢN TRỊ VIÊN HỆ THỐNG)
---

# CLAUDE.md — Green Pool ERP Working Guide
> Auto-updated: 2026-06-11 | Người dùng: Nguyễn Văn Hướng (QUẢN TRỊ VIÊN HỆ THỐNG)
> Repo: https://github.com/trihuong815-oss/greenpool-erp
> Production: https://greenpool-erp.vercel.app (Firebase App Hosting — asia-southeast1)

---

## 1. THÔNG TIN DỰ ÁN

### Tổng quan
- **Tên**: Green Pool ERP — Hệ thống quản lý vận hành 5 cơ sở bể bơi
- **Stack**: Next.js 16 + React 18 + TypeScript + Tailwind CSS + Firebase (Auth + Firestore + Storage + FCM)
- **Deploy**: Firebase App Hosting (apphosting.yaml) — NOT Vercel deploy, chỉ domain vercel.app
- **Font**: Inter (Google Fonts) + system font fallback — import trong globals.css

### 5 Cơ sở (BranchId)
| ID | Tên đầy đủ | Màu |
|----|-----------|-----|
| HM | Green Pool Hoàng Mai | #10b981 |
| TK | Green Pool 20 Thuỷ Khuê | #06b6d4 |
| CTT | Green Pool Cung Thể Thao Mã | #8b5cf6 |
| 24 | Green Pool 24 Nguyễn Công Trứ | #f59e0b |
| TT | Green Pool Thanh Trì | #ef4444 |

### Cấu trúc tổ chức (Roles)
Các role function check: `isTopAdmin`, `isCEO`, `isAdminSystem`, `isGD`, `isTP`, `isQLCS`, `isWriteAdmin`, `canSeeAllFacilities`, `hasRole`
(source: lib/auth/roles.ts — SINGLE SOURCE OF TRUTH)

---

## 2. KIẾN TRÚC CODE

### Cấu trúc thư mục chính
```
app/
  (app)/              # protected routes — yêu cầu login
    dashboard/        # DashboardContent.tsx, KTDashboardSection.tsx
    giao-viec/        # GiaoViecClient.tsx, TaskCreateModal.tsx, TaskDetailModal.tsx
    doanh-so/         # NhapClient.tsx, PackagesClient.tsx, ManageSalesModal.tsx
    ky-thuat/         # GiaoViecClient.tsx (KT), HoaChatClient.tsx, MayClient.tsx
    checklist-v2/     # ChecklistV2Client.tsx, SupervisorView.tsx, ChecklistHeatmap.tsx
    cong-viec-ca-nhan/ # PersonalWorkClient.tsx + AIPanel, GoalsPanel, HabitsPanel, JournalPanel
    tin-nhan/         # TinNhanClient.tsx + ChatAttachments, MessageThread, Modals
    sodo/             # OrgChartClient.tsx, FlowView.tsx, OrgTreeView.tsx
    bao-cao/          # page.tsx (báo cáo tự động)
    bao-mat/          # SecurityClient.tsx (bảo mật & thông báo)
    users/            # UsersClient.tsx, PermissionGrantPanel.tsx
    quan-ly-sale/     # QuanLySaleClient.tsx
    settings-packages/ # page.tsx
  api/
    tasks/            # 8 route handlers (CRUD + comments + attachments + approval)
    sales/            # 5 route handlers
    ky-thuat/         # 5 route handlers
    chat/             # 8 route handlers
    personal/         # 19 route handlers
    cron/             # 5 scheduled jobs
    admin/            # 5 admin routes
  globals.css         # @import Inter, @tailwind base/components/utilities
  layout.tsx          # root layout — body: antialiased, bg-slate-50 text-slate-800
components/
  AppShell.tsx        # layout shell: sidebar + topbar + main
  Sidebar.tsx         # navigation sidebar
  AppTopBar.tsx       # top navigation bar
  ui/                 # Button, Card, Input, Badge, Toast, EmptyState, Skeleton...
lib/
  types/
    index.ts          # BARREL — import tất cả types từ đây
    tasks.ts          # re-export từ services/tasks/api-client.ts
    users.ts          # UserDoc, UserPublic, FcmDevice, CallerProfile
    branches.ts       # BranchId, BranchMeta, BRANCHES, BRANCH_BY_ID
  auth/
    roles.ts          # SINGLE SOURCE — isTopAdmin, isCEO, isGD, isTP, isQLCS...
    can.ts            # permission helpers
  firebase/
    collections.ts    # SINGLE SOURCE — tên collection Firestore
    admin.ts          # Firebase Admin SDK
    client.ts         # Firebase Client SDK
  services/
    tasks/api-client.ts      # Task CRUD, TaskCreate, Task interface
    sales/api-client.ts      # Sales CRUD
    ky-thuat/work-api-client.ts  # KT work tasks
    chat/api-client.ts       # Chat/messaging
  permissions.ts      # permission logic
  branches.ts         # branch data
  navigation/routes.ts # ALL_ROUTES definition
```

---

## 3. MENU / NAVIGATION

### Sections và routes (theo thứ tự sidebar)
**Tổng quan**: dashboard | tin-nhan | cong-viec-ca-nhan
**Vận hành**: doanh-so | doanh-so/nhap | ky-thuat | checklist-v2 | quy-trinh | giao-viec
**Nhân sự**: sodo | luong
**Báo cáo**: bao-cao | daotao | mkt
**Cài đặt**: bao-mat | doanh-so/packages | users | doi-mat-khau

---

## 4. MODULE ĐIỀU PHỐI CÔNG VIỆC (/giao-viec) — TRUNG TÂM HỆ THỐNG

### Files
- `app/(app)/giao-viec/GiaoViecClient.tsx` — main client component
- `app/(app)/giao-viec/TaskCreateModal.tsx` — form tạo task/proposal
- `app/(app)/giao-viec/TaskDetailModal.tsx` — chi tiết + timeline + actions
- `app/(app)/giao-viec/page.tsx` — server component wrapper

### Tabs
- **Tôi phụ trách** (my-tasks): tasks giao cho mình
- **Tôi giao** (assigned-by-me): tasks mình tạo
- **Liên khối** (cross-block): chỉ CEO/Admin
- **Chờ phản hồi** (pending-response): cần action
- **Quá hạn** (overdue): deadline đã qua

### KPI Header (5 cards)
Đang xử lý | Chờ phản hồi | Chờ duyệt | Quá hạn | Hoàn thành

### 3 Panels (header section)
Công việc theo khối | Tắc nghẽn hiện tại | Công việc quá hạn

### Table View (9 cột)
\# | Công việc | Loại | Khối chủ trì | Phối hợp | Trạng thái | Tiến độ | Đang chờ | Deadline

### Task Interface (Task type — lib/services/tasks/api-client.ts)
```typescript
interface Task {
  id: string;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trihuong815-oss/greenpool-erp](https://github.com/trihuong815-oss/greenpool-erp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
