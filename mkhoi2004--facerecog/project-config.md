---
trigger: always_on
description: **Agent & Developer Handbook** | Version 3.0 | Ngôn ngữ: Tiếng Việt + Code EN
---

# SKILL.md — FaceID Attendance System
**Agent & Developer Handbook** | Version 3.0 | Ngôn ngữ: Tiếng Việt + Code EN

> **📌 ĐỌC TRƯỚC KHI LÀM BẤT CỨ ĐIỀU GÌ**
> File này là nguồn sự thật duy nhất (single source of truth) của toàn bộ project.
> Agent: đọc hết file này trước khi tạo, sửa, hoặc xóa bất kỳ file nào.
> Developer mới: đọc §1 → §3 → §8 là đủ để bắt đầu làm việc.

---

## ═══ QUICK REFERENCE ═══

> Tìm nhanh — không cần đọc toàn bộ:

| Tôi cần... | Section | Ưu tiên |
| :--- | :--- | :--- |
| Hiểu project làm gì, tech stack | §1 | Đọc đầu tiên |
| Xem toàn bộ file và thư mục | §3 | Đọc đầu tiên |
| Build và chạy lần đầu | §8 | Đọc đầu tiên |
| Thêm field / sửa dữ liệu | §7.1 | Hay dùng nhất |
| Fix lỗi nhận diện mặt | §7.2 | Hay dùng nhất |
| Thêm tab/module UI mới | §7.3 | Hay dùng nhất |
| Viết SQL / query mẫu | §4.4 | Hay dùng nhất |
| Hiểu luồng nghiệp vụ chấm công | §5.3 | Quan trọng |
| Hiểu AI recognition pipeline | §5 | Quan trọng |
| Tra cứu màu sắc, font, GDI+ UI | §6 | Khi làm UI |
| Xem lỗi phổ biến và cách fix | §9 | Khi debug |
| Quy trình migration DB | §7.4 | Khi đổi schema |
| Checklist trước khi deploy | §10 | Trước khi release |
| Agent decision tree | §11 | Agent dùng |
| Lịch sử thay đổi | §12 | Tham khảo |

---

## §1 — PROJECT CONTEXT

### 1.1 Mục tiêu hệ thống

Hệ thống chấm công Windows chuyên nghiệp cho doanh nghiệp vừa và nhỏ. Nhận diện nhân viên qua webcam bằng deep learning (không cần thẻ từ, vân tay), lưu trữ toàn bộ trên SQLite với audit trail đầy đủ.

**Các chức năng cốt lõi**:
- Chấm công tự động qua nhận diện khuôn mặt real-time
- Quản lý nhân viên (thêm, sửa, xóa, gán khuôn mặt)
- Quản lý ca làm, phòng ban
- Xin và duyệt nghỉ phép
- Báo cáo chấm công theo ngày/tháng/nhân viên
- Audit log toàn bộ thao tác

### 1.2 Tech Stack

| Layer | Technology | Package NuGet | Version |
| :--- | :--- | :--- | :--- |
| UI | C# WinForms | — | .NET 4.6.1 |
| AI Recognition | Dlib ResNet-128 | `DlibDotNet` | 19.21.x |
| Computer Vision | OpenCV | `OpenCvSharp4` | 4.x |
| Database | SQLite | `System.Data.SQLite` | 6.x |
| Architecture | N-Tier (UI → Service → Repo) | — | — |

### 1.3 Ràng buộc bất di bất dịch (Non-negotiables)

```
⚠️  Platform:     x64 ONLY — Dlib native không có x86/AnyCPU build
⚠️  .NET:         Framework 4.6.1 — không phải .NET Core hay .NET 5+
⚠️  Model path:   PHẢI qua ModelsDirectoryResolver (xem §5.1 — Unicode bug)
⚠️  SQL:          Raw SQL trong Repository.cs — KHÔNG dùng ORM, Entity Framework
⚠️  UI threading: DB/AI calls KHÔNG được chạy trên UI thread
```

---

## §2 — SYSTEM ARCHITECTURE

### 2.1 Sơ đồ tổng quan

```
╔══════════════════════════════════════════════════════════════════╗
║                     FaceIDApp — WinForms UI                      ║
║  ┌──────────────┬───────────────────┬──────────┬──────────────┐  ║
║  │ UCAttendance │UCEmployeeManagement│ UCReports│UCLeaveRequest│  ║
║  └──────────────┴───────────────────┴──────────┴──────────────┘  ║
╚═══════════╤══════════════════════════════╤═══════════════════════╝
            │ gọi Service                  │ gọi Service
            ▼                              ▼
╔═══════════════════════════╗   ╔══════════════════════════════╗
║  FaceRecognitionService   ║   ║    WebcamCaptureService       ║
║  - MatchFace()            ║   ║    - StartCapture()           ║
║  - RegisterFace()         ║   ║    - GetCurrentFrame()        ║
║  - GetAttendanceStatus()  ║   ║    (OpenCvSharp4)             ║
╚═══════════╤═══════════════╝   ╚══════════════════════════════╝
            │ gọi AI Lib + Repo
     ┌──────┴──────┐
     ▼             ▼
╔══════════════╗  ╔══════════════════════════════════════╗
║ Repository   ║  ║  FaceRecog Lib  (src/FaceRecog/)     ║
║ .cs          ║  ║  - FaceRecognition.cs                ║
║ Raw SQL      ║  ║  - Wrapper cho DlibDotNet P/Invoke   ║
╚══════╤═══════╝  ╚══════════════════════════════════════╝
       │
       ▼
╔══════════════════╗
║  SQLite DB   ║
║  face_attendance ║
╚══════════════════╝
```

### 2.2 Nguyên tắc kiến trúc (Architecture Rules)

```
Rule 1: UI không được gọi Repository trực tiếp.
        UI → Service → Repository (bắt buộc đi qua Service layer)

Rule 2: Service không chứa SQL.
        SQL chỉ được viết trong Repository.cs

Rule 3: Repository không chứa business logic.
        Chỉ CRUD thuần túy — không if/else nghiệp vụ ở đây

Rule 4: DTO là vật trung gian duy nhất giữa các layer.
        Không truyền SqlDataReader, System.Data.SQLiteDataReader ra ngoài Repository

Rule 5: Mọi exception phải được log trước khi bubble up.
        Không bao giờ để exception âm thầm bị nuốt (catch không xử lý)
```

### 2.3 Luồng dữ liệu — Chấm công (Happy Path)

```
[Webcam] → frame (Bitmap)
    │
    ▼
WebcamCaptureService.GetCurrentFrame()
    │
    ▼
FaceRecognitionService.MatchFace(bitmap)
    │
    ├─► FaceRecogLib.LocateFaces(bitmap, upsampling=1)
    │       └─ [Nếu rỗng] retry upsampling=2
    │
    ├─► FaceRecogLib.GetFaceEncodings(bitmap, locations)
    │       └─ Trả về List<double[128]>
    │
    ├─► Repository.GetAllFaceEncodings()
    │       └─ Trả về Dict<employeeId, List<double[128]>>
    │
    ├─► FaceRecogLib.FaceDistance(candidate, known) cho từng encoding
    │       └─ Tìm distance nhỏ nhất, so với tolerance (0.6)
    │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkhoi2004/FaceRecog](https://github.com/mkhoi2004/FaceRecog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
