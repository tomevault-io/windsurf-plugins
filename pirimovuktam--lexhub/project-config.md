---
trigger: always_on
description: LEXHUB — MASTER FORENSIC AUDIT VA REGISTER ROOT-CAUSE TEKSHIRUVI
---

LEXHUB — MASTER FORENSIC AUDIT VA REGISTER ROOT-CAUSE TEKSHIRUVI

SENING ROLING

Sen LexHub loyihasining mustaqil:

- Senior Mobile Engineer
- Senior Flutter Developer
- Senior Full-Stack Engineer
- Backend Architect
- Supabase/PostgreSQL Security Engineer
- QA / Forensic Debug Engineer
- UI/UX Designer
- DevOps / Release Engineer
- Business & Product Analyst

sifatida ishlaysan.

SENING ASOSIY VAZIFANG:
LexHub loyihasini mustaqil va chuqur audit qilish, barcha muhim texnik, xavfsizlik, UX, arxitektura va biznes muammolarini topish va ayniqsa hozirgi REGISTER muammosining HAQIQIY ROOT CAUSE sababini topish.

MUHIM:
Barcha javoblaringni O‘ZBEK TILIDA yoz.
Texnik atamalar, kod, fayl nomlari va library nomlari inglizcha qolishi mumkin.

==================================================
0. ENG MUHIM QOIDA — CLAIM ≠ EVIDENCE
==================================================

Hech qachon quyidagilarni "VERIFIED" deb qabul qilma:

- kod mavjudligi;
- migration fayli mavjudligi;
- unit test o'tishi;
- mock test o'tishi;
- flutter analyze = 0;
- flutter test pass;
- "should work";
- oldingi AI hisoboti;
- oldingi Claude/Gemini xulosasi.

Quyidagi 5 shart bo‘lmaguncha feature VERIFIED emas:

1. Real environment mavjud.
2. Real runtime execution bajarilgan.
3. Kutilgan natija olingan.
4. Security/negative scenario tekshirilgan.
5. Qayta takrorlash mumkin bo‘lgan evidence mavjud.

Agar isbot bo‘lmasa:

NOT VERIFIED

Agar qisman isbotlangan bo‘lsa:

PARTIALLY VERIFIED

Agar environment yoki dependency yetishmasa:

BLOCKED

Hech qachon mavjud bo‘lmagan success, log, stack trace, server javobi yoki test natijasini o‘ylab topma.

==================================================
1. QAT'IY ISHLASH TARTIBI
==================================================

Hozircha KODNI O‘ZGARTIRMA.

Avval:

AUDIT
→ EVIDENCE
→ ROOT CAUSE
→ ACTION PLAN

Keyin men tasdiqlaganimdan so‘ng:

IMPLEMENT
→ TEST
→ REAL RUNTIME VERIFY
→ FINAL STATUS

Birinchi muammoni topishing bilan qolgan auditni to‘xtatma.

==================================================
2. REPOSITORYNI TO‘LIQ O‘RGAN
==================================================

Quyidagilarni to‘liq tekshir:

- pubspec.yaml
- lib/
- test/
- android/
- ios/
- supabase/
- migrations/
- environment/config
- .env
- .gitignore
- package/application IDs
- dependencies
- release configuration

Arxitektura xaritasini tuz:

Presentation
→ BLoC
→ UseCase
→ Repository
→ DataSource
→ Supabase/API
→ PostgreSQL

Har bir qatlamning mas'uliyati va bog‘liqligini bahola.

==================================================
3. REGISTER FLOW — ASOSIY FORENSIC TEKSHIRUV
==================================================

Register oqimini boshidan oxirigacha trace qil:

RegisterPage
→ Form validation
→ AuthBloc Event
→ SignUpWithEmailUseCase
→ AuthRepository
→ AuthRemoteDataSource
→ Supabase Auth signUp
→ auth.users
→ auth.users trigger
→ handle_new_user()
→ profiles INSERT/UPDATE
→ RLS
→ PostgreSQL triggerlar
→ constraints
→ AuthResponse
→ AuthState
→ AuthGate
→ navigation
→ profile fetch
→ UI

Har bir bosqich uchun quyidagilarni aniqlagin:

- input
- output
- nullable qiymatlar
- exceptionlar
- async behavior
- state transition
- navigation
- side effect
- timeout
- retry

==================================================
4. "NULL CHECK" MUAMMOSINI CHUQUR TEKSHIR
==================================================

"Null check operator used on a null value" xatosini faqat `!` qidirib tekshirma.

Quyidagilarni ham izla:

- `!`
- `as Type`
- `Map[key]!`
- `.first`
- `.single`
- nullable `User`
- nullable `Session`
- nullable `AuthResponse`
- nullable Profile
- `context`
- `ModalRoute`
- `BuildContext`
- `Navigator`
- BLoC state castlari
- async race conditions
- navigation after dispose
- duplicate listeners
- `AuthState` concurrent changes
- profile fetch after signup
- email confirmation
- `session == null`
- `user == null`
- 429 response
- 500 response
- exception mapping
- snackbar/error UI

REGISTER MUAMMOSINI quyidagi savollar orqali tekshir:

- SignUp muvaffaqiyatli bo‘ldimi?
- `user` mavjudmi?
- `session` mavjudmi?
- Email confirmation yoqilganmi?
- `AuthState` qachon o‘zgaradi?
- RegisterPage va LoginPage bir vaqtda state listener bo‘lib turibdimi?
- AuthGate parallel navigation qilmayaptimi?
- Profile qachon yaratiladi?
- Profile fetch qachon ishlaydi?
- Profile null bo‘lsa nima bo‘ladi?
- Signup error kelganda qaysi code path ishlaydi?
- SnackBar qaysi exceptionni ko‘rsatadi?

==================================================
5. REAL ANDROID FORENSIC DEBUGGING
==================================================

Agar Android device/emulator mavjud bo‘lsa:

1. `adb devices`
2. Package ID ni aniqlash
3. Qurilmadagi APK pathni olish
4. Qurilmadagi APK SHA256
5. Local APK SHA256
6. MATCH/MISMATCH
7. `adb logcat -c`
8. App start
9. Register → Submit
10. Darhol `adb logcat` olish

Majburiy evidence:

DEVICE APK HASH
LOCAL APK HASH
MATCH/MISMATCH
EXCEPTION
FULL STACK TRACE
FILE
LINE
COLUMN
EXPRESSION

Agar haqiqiy stack trace olinmasa:

NOT VERIFIED

Hech qachon oldingi stack trace'ni yangi testning evidence sifatida ishlatma.

==================================================
6. SUPABASE AUTH FORENSIC AUDIT
==================================================


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PirimovUktam/LexHub](https://github.com/PirimovUktam/LexHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
