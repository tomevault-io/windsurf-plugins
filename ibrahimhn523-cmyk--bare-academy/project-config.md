---
trigger: always_on
description: > ملف مرجعي يصف المشروع بشكل دقيق ليستخدمه Claude في الجلسات القادمة. يُحدَّث عند تغيير الهيكلة أو اتخاذ قرارات معمارية.
---

# CLAUDE.md — دليل المشروع لـ Claude Code

> ملف مرجعي يصف المشروع بشكل دقيق ليستخدمه Claude في الجلسات القادمة. يُحدَّث عند تغيير الهيكلة أو اتخاذ قرارات معمارية.

---

## ١. بنية المشروع الحالية

مشروع **بوابة أكاديمية بارع** — منصة ويب لإدارة برامج أكاديمية رياضية تعليمية في بريدة، السعودية.

### الملفات وأدوارها

```
bare-academy/
├── index.html            (568 سطر) — الصفحة العامة + نموذج التسجيل
├── dashboard.html        (744 سطر) — لوحة الإدارة
├── dashboard.css         (365 سطر) — تنسيقات لوحة الإدارة
├── dashboard.js          (2454 سطر) — منطق لوحة الإدارة
├── portal.html           (920 سطر) — بوابة الموظفين متعددة الصلاحيات
├── portal.css            (1025 سطر) — تنسيقات البوابة
├── portal.js             (3092 سطر) — منطق البوابة
├── leaderboard.html      (408 سطر) — لوحة صدارة عامة (HTML + JS مدمج)
├── tournament-view.html  (567 سطر) — عرض البطولة العام (HTML + JS مدمج)
├── README.md             (170 سطر) — توثيق عام (⚠️ متقادم: يذكر Google Sheets)
├── image/                — الشعارات (bare-logo.png, bare-logo-blue.png)
├── fonts/                — خط TheYearofHandicrafts (5 أوزان)
└── .claude/
    └── settings.local.json — صلاحيات Claude Code المحلية
```

### التطبيقات الأربعة

| التطبيق | الجمهور | الوصف |
|---|---|---|
| `index.html` | العامة | صفحة هبوط + نموذج تسجيل طالب → يكتب في جدول `students` |
| `dashboard.html` | المدير العام (كلمة سر واحدة) | إدارة الطلاب، البرامج، الاشتراكات، الرسوم، التواصل، الإحصائيات، السجل |
| `portal.html` | موظفون متعددون (users + permissions) | تحضير، نقاط، مسابقات ثقافية، بطولات رياضية، صدارة، إدارة المستخدمين |
| `leaderboard.html` + `tournament-view.html` | العامة (قراءة فقط) | عرض صدارة برنامج معيّن وعرض بطولة (يُمرَّر `?prog=` أو `?id=`) |

### ملاحظة عن الملفات المفقودة المذكورة في README
- `registration.html` — **غير موجود**؛ نموذج التسجيل مدمج في `index.html`.
- `google-sheets-api.js` — **غير موجود**؛ تمت الهجرة إلى Supabase.
- `package.json` — **غير موجود**؛ لا يوجد build step.

---

## ٢. الـ Stack المستخدم

### Frontend
- **Vanilla HTML / CSS / JavaScript** — بدون framework، بدون build step، بدون bundler.
- **CDN خارجي:**
  - SheetJS `xlsx-0.20.1` (في `dashboard.html`) — لاستيراد/تصدير Excel.
  - Font Awesome 6.4.0 (في `index.html` فقط).
- **خط محلي:** `TheYearofHandicrafts` (5 أوزان) في مجلد `fonts/`.

### Backend / Data
- **Supabase** (PostgREST API):
  - URL: `https://oytfhgqhibbcsqbnvwyv.supabase.co/rest/v1`
  - مفتاح: `anon` JWT — مكشوف في الكود client-side في كل الملفات.
- **النداء مباشر** من المتصفح إلى Supabase REST عبر `fetch()` — لا يوجد خادم تطبيق وسيط.
- **localStorage** للـ session (`portal_user`) ولقوالب WhatsApp (`wa_templates`).

### الجداول في Supabase
| الجدول | الاستخدام |
|---|---|
| `students` | الطلاب |
| `programs` | البرامج التعليمية |
| `subscriptions` | اشتراكات الطلاب في البرامج |
| `payments` | الدفعات |
| `settings` | إعدادات عامة |
| `logs` | سجل العمليات (آخر 150) |
| `users` | مستخدمو البوابة (مع `password`, `role`, `permissions`) |
| `attendance`, `attendance_log` | التحضير |
| `points`, `point_reasons` | نظام النقاط |
| `cultural_competitions`, `cultural_participants` | المسابقات الثقافية |
| `sports_tournaments`, `sports_teams`, `sports_matches`, `sports_stats` | البطولات الرياضية |

### الاستضافة
- **غير محددة في الكود** — لا `vercel.json`، لا `netlify.toml`، لا Dockerfile.
- يعمل كـ **static site** على أي خادم HTTP بسيط (Python, Node http-server حسب README).

### Google Sheets
- **مذكور في README فقط — غير مستخدم في الكود.**
- المشروع كان مرتبطاً بـ Google Sheets سابقاً ثم هاجر إلى Supabase. كلمة "sheet" في الكود تشير لـ Excel sheets فقط (مكتبة SheetJS).

### المصادقة
- **لا يوجد Supabase Auth.**
- المصادقة يدوية: جدول `users` مع `password` plaintext، فحص بمقارنة نصية، حفظ في `localStorage`.
- `dashboard.html` يستخدم كلمة مرور واحدة ثابتة (موثّقة سابقاً في README).

---

## ٣. المشاكل الحرجة المعروفة

### 🔴 C1 — كلمات المرور مخزّنة plaintext
- **المكان:** [portal.js:144](portal.js:144)
- **الوصف:** `if (u.password !== password)` — مقارنة نصية، يعني الـ DB تحتوي passwords بدون hashing/salting.
- **الأثر:** أي تسرّب لـ DB = كشف كل كلمات المرور دفعة واحدة. كثير من المستخدمين يكررون كلمات المرور في حسابات أخرى.

### 🔴 C2 — مصادقة client-side عبر localStorage فقط
- **المكان:** [portal.js:146-162](portal.js:146)
- **الوصف:** بعد `login()` يُحفظ كائن المستخدم (مع `role` و `permissions`) في `localStorage`. عند إعادة التحميل يُقرأ منه مباشرة بدون أي تحقق من السيرفر.
- **الأثر:** أي مستخدم يفتح Console ويعدّل `localStorage.portal_user` ليصير `role:'super_admin'`، ثم يحدّث الصفحة → صلاحية كاملة بدون استدعاء سيرفر.

### 🔴 C3 — Anon key مكشوف للعمليات الحساسة
- **المكان:** [dashboard.js:5](dashboard.js:5), [portal.js:6](portal.js:6), [index.html:519](index.html:519), [leaderboard.html:219](leaderboard.html:219), [tournament-view.html:206](tournament-view.html:206)
- **الوصف:** نفس الـ anon JWT يُستخدم لـ INSERT/UPDATE/DELETE على كل الجداول من المتصفح.
- **الأثر:** كل الأمان يعتمد على Supabase **RLS policies** (غير مرئية في الكود). لو RLS غير محكمة → أي زائر يقدر يعمل `DELETE /students` أو `UPDATE /payments` مباشرة.

### 🔴 C4 — لا rate limiting على login
- **المكان:** [portal.js:133-151](portal.js:133)
- **الوصف:** `login()` يقبل عدداً لا نهائياً من المحاولات.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ibrahimhn523-cmyk/bare-academy](https://github.com/ibrahimhn523-cmyk/bare-academy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
