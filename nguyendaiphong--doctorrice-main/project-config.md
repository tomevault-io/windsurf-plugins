---
trigger: always_on
description: Bộ quy tắc phát triển ứng dụng React Native + Expo cho dự án "Bác sĩ Lúa".
---

@description
Bộ quy tắc phát triển ứng dụng React Native + Expo cho dự án "Bác sĩ Lúa". 
Mục tiêu: đảm bảo project có cấu trúc chuẩn (/src), hiệu năng tối ưu trên Android, quản lý i18n chuẩn, quyền runtime rõ ràng, flow chụp ảnh + watermark thông qua backend, và backend Node.js + MongoDB nhẹ, ổn định, có Swagger và cơ chế wake-on-sleep cho Render. 
@end

@context
Ngôn ngữ: TypeScript (ưu tiên) / JavaScript (hàng phụ)
Framework: Expo Managed Workflow (create-expo-app)
Mục tiêu triển khai: Android-first tối ưu, tương thích iOS, code có thể scale, dễ maintain, CI-friendly, documentation luôn cập nhật.
@end

@rules
{
  "always": {
    "description": "Những quy tắc bắt buộc toàn cục.",
    "rules": [
       "Trước khi CHẠY CODE: luôn hỏi user xác nhận rõ ràng những câu hỏi bạn chưa hiểu (ví dụ: 'Tôi cần làm rõ vấn đề abc này và tôi gợi ý dùng phương án này, bạn có đồng ý không?').",
       "Trước khi XOÁ hoặc CHỈNH SỬA dữ liệu: luôn hỏi xác nhận với user trước khi thực hiện, và hiển thị tóm tắt dữ liệu sẽ bị ảnh hưởng nếu có thể.",
       "SAU MỖI LẦN user gửi yêu cầu: trước khi thực hiện hành động chính được yêu cầu, luôn xác nhận lại rằng hành động đó đúng với ý định của user.",
      "Luôn dùng TypeScript cho business logic; file UI có thể là .tsx. Nếu dùng JS, phải có JSDoc cho mọi function export.",
      "Project root phải có file `.env-example` chứa tất cả biến môi trường dùng trong app và backend (ví dụ: API_URL, GOOGLE_CLIENT_ID, FACEBOOK_APP_ID, SENTRY_DSN nếu dùng). Không commit `.env` thật vào VCS.",
      "Không để `console.log`, `debugger` trong bản build production; dùng logger controllable (ví dụ `debug` hoặc Sentry) và tắt theo biến môi trường.",
      "Mọi API network call phải thông qua module riêng `/src/services/api.ts` và sử dụng axios instance có timeout, retry logic (3 lần), và cancel token support."
    ]
  },

  "projectStructure": {
    "description": "Cấu trúc Hybrid với Expo Router + /src cho business logic.",
    "rules": [
      "Root chứa: /app (Expo Router screens), /src (business logic), và các file cấu hình (package.json, app.json, tsconfig.json, .eslintrc, .prettierrc, .env-example)",
      "Cấu trúc Hybrid:",
      "  /app - Expo Router screens (file-based routing): (tabs), modal, _layout.tsx. Screens chỉ chứa UI layout và import logic từ /src",
      "  /src/components - các component tái sử dụng (Header, Button, Modal, Avatar, WatermarkPreview, SkeletonLoader, etc.)",
      "  /src/hooks - custom hooks (useAuth, usePermissions, useI18n, useCameraFlow, useFetch)",
      "  /src/services - api clients, auth service, storage service",
      "  /src/constants - colors, spacing, fonts, keys",
      "  /src/i18n - cấu hình i18n và files locales (vi.json, en.json)",
      "  /src/utils - helper functions, formatters, validators",
      "  /src/types - TypeScript types và interfaces global",
      "  /src/assets - hình, icon, fonts (tối ưu kích thước & webp nếu phù hợp)",
      "Import xuyên folder phải dùng path aliases (ví dụ: @/components, @/hooks, @/services) được cấu hình trong tsconfig.json. Không dùng nhiều `../../..`.",
      "Screens trong /app KHÔNG chứa business logic - chỉ import hooks/components từ /src và render UI."
    ]
  },

  "appJsonAndBuild": {
    "description": "Cấu hình app.json để tên hiển thị là \"Bác sĩ Lúa\" và build tối ưu.",
    "rules": [
      "app.json / app.config.js phải đặt `name` và `displayName` phù hợp: displayName = \"Bác sĩ Lúa\".",
      "Sử dụng `expo-asset` và `expo-font` để preload assets và fonts trước khi show root screen.",
      "Dùng `eas build` cho production; cấu hình eas.json để có profile 'production' và 'development'.",
      "Prioritize minimal bundle: only install packages cần thiết, tránh native modules không hỗ trợ Expo Managed Workflow.",
      "Kiểm tra và test `expo start --clear` trước khi build release.",
      "Định nghĩa App version và buildNumber trong app.json và cập nhật mỗi release."
    ]
  },

  "authAndAccounts": {
    "description": "Đăng nhập/Đăng ký qua email, số điện thoại, Google, Facebook.",
    "rules": [
      "Xây module auth tại `/src/services/auth` xử lý: login, register, socialLogin, refreshToken, logout.",
      "Email/password và phone/SMS OTP đều phải xác thực trên backend. Trên app: validate form trước khi gửi (email regex, phone format).",
      "Social login (Google, Facebook) sử dụng `expo-auth-session` hoặc SDK tương thích Expo; token exchange phải thực hiện an toàn qua backend (backend verify token với Google/Facebook và trả JWT của hệ thống).",
      "Không lưu mật khẩu ở client. Lưu token/refreshToken trong SecureStore (Expo SecureStore) hoặc encrypted storage.",
      "Thêm flow must_change_password, forgot password và verify phone flow (OTP)."
    ]
  },

  "permissionsAndPrivacy": {
    "description": "Quy tắc xử lý quyền runtime và privacy.",
    "rules": [
      "Khi app lần đầu chạy, show permission rationale UI rõ ràng trước khi gọi requestPermissions (camera, location).",
      "Đề xuất quyền: CAMERA và LOCATION (coarse/fine) — chỉ request khi user cần tính năng tương ứng (không request cùng lúc nếu chưa dùng).",
      "Dùng `expo-permissions` / `expo-location` / `expo-camera` để kiểm tra & request quyền.",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NguyenDaiPhong/DoctorRice-main](https://github.com/NguyenDaiPhong/DoctorRice-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
