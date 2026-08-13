---
trigger: always_on
description: ეს ფაილი Claude Code-ს (claude.ai/code) აძლევს კონტექსტს ამ repo-ში მუშაობისას.
---

# CLAUDE.md

ეს ფაილი Claude Code-ს (claude.ai/code) აძლევს კონტექსტს ამ repo-ში მუშაობისას.

## პროექტი

**Cycle Care** (მარკეტინგული სახელი "Qalis Sivrce") — ქართულენოვანი (ka-GE) მენსტრუალური ციკლის თრექინგის Expo/React Native აპი, პარალელური ორსულობის რეჟიმით და OpenAI-ზე დაფუძნებული AI ასისტენტით. Bundle ID `com.mishvela.kalebi`, EAS slug `kalebi`.

Backend: Supabase (Auth, DB, Edge Functions). AI: OpenAI-ს იძახებს მხოლოდ `supabase/functions/ai-assistant/index.ts` — OpenAI key ყოველთვის სერვერზეა, არასდროს frontend-ში (იხ. `AI_SETUP.md`).

---

## ბოლოს რა გავაკეთეთ (ბოლო სესიის snapshot)

> ეს სექცია ყოველთვის უნდა აჩვენებდეს ყველაზე ახალ მდგომარეობას — ახალი სესიის დაწყებისას აქედან დაიწყე, `git log`/`git status`-ის თავიდან აწარმოების ნაცვლად. საჭიროებისამებრ განაახლე.

**🔐 AI ბიუჯეტი გამოწერას მიჰყვება + `premium_override` დაიბლოკა (2026-08-01 — committed `f7d06fc`, pushed, **migration + edge function deployed ✅**)**: აუდიტის ბოლო ორი ხვრელი. ⚠️ **OTA არ სჭირდება** — ორივე სერვერზეა.
- **`ai-assistant/index.ts`** (**deployed ✅**): chat-ის ლიმიტი ადრე მხოლოდ `pregnancy_mode`-ს უყურებდა → ვადაგასულ გამომწერელს (ან ვინც დროშას თვითონ ჩაიწერდა) **10 კითხვა რჩებოდა 1-ის ნაცვლად** = რეალური OpenAI-ს ხარჯი. ახალი `hasPregnancyAccess()` (`resolvePregnancyAccessFromProfile`-ის სარკე) + `pregnancyActive = pregnancy_mode && hasPregnancyAccess(profile)`. select-ს დაემატა `has_pregnancy_subscription, pregnancy_until`. fail-open ქცევა profile-ის წაკითხვის შეცდომაზე **შენარჩუნებულია**.
- **migration `20260801_lock_premium_override.sql`** (**remote ✅**): 🛡️ trigger `guard_premium_override` on `profiles` (BEFORE INSERT OR UPDATE, SECURITY DEFINER). მიზეზი: RLS **სვეტს ვერ ზღუდავს**, `premium_override`-ს კი `ThemeContext` **არასდროს ამოწმებს** RevenueCat-თან (ადრევე ჩერდება) → user-ს საკუთარ row-ში ჩაწერით **სამუდამო Prime** შეეძლო. ახლა: `auth.uid() is null` (service_role/migration/SQL editor) → გაატარებს; admin-ის JWT email → გაატარებს; სხვა შემთხვევაში UPDATE-ზე **`raise exception` (42501)**, INSERT-ზე `false`-ზე აიძულებს.
- ⚠️ **მხოლოდ ეს ერთი სვეტია დაბლოკილი.** `is_premium`/`has_pregnancy_subscription` კლიენტიდან უნდა იწერებოდეს (purchases.js), და ისინი ისედაც **ყოველ გახსნაზე გადაიწერება** store-იდან, ე.ი. თვით-მინიჭება ვერ გადარჩება. სრული დაბლოკვა მოითხოვს ჩაწერის სერვერზე გადატანას — ცალკე სამუშაოა.
- ✅ **ტესტირებული**: 12/12 — რეალური გამომწერელი, legacy `until=null`, გაუქმებული ფასიან პერიოდში → **ბიუჯეტი უცვლელი**; ვადაგასული, თვით-ჩაწერილი, გატეხილი თარიღი → **1/დღეში**. Prime ორსულობას ჯობნის, `premium_override` ისევ მუშაობს.
- ✅ **ხელით გადამოწმებულია live-ზე (user-მა, 2026-08-01)**: admin პანელის „Admin Prime" გადამრთველი მუშაობს (trigger admin-ის JWT-ს ატარებს) და AI ჩატიც პასუხობს.
- ↩️ **rollback**: `drop trigger guard_premium_override on public.profiles;`

**🎯 guard დავიწროვდა — `admin_grant` = ხელით მიცემული წვდომა (2026-08-01 — committed `a38e169`, pushed, ⚠️ OTA ჯერ არ გასულა)**: ⛔ **მთავარი კონტექსტი: Android საერთოდ არ არის გაშვებული — აპი მხოლოდ iOS-ზეა.** ამიტომ ეტაპ 1-ის guard-ი ("დაიცავი ყველაფერი, რასაც RevenueCat ვერ ხედავს") აზრს კარგავდა: Android-ის მყიდველები არ არსებობენ, ხოლო `profiles`-ზე user-ს **საკუთარი row-ის სრული UPDATE უფლება აქვს** (RLS გადამოწმებულია — `Users can update their profile`, სვეტების შეზღუდვის გარეშე), ე.ი. უწყარო `has_pregnancy_subscription=true` თვითონვე შეიძლება ჩაეწერა.
- **`purchases.js`** — ახალი `MANUAL_PREGNANCY_GRANT_SOURCES = ["admin_grant"]`. guard ახლა **მხოლოდ** `pregnancy_source = 'admin_grant'`-ს იცავს; ყველა დანარჩენი (`null`-ის ჩათვლით) ჩვეულებრივ ექვემდებარება RevenueCat-ის პასუხს.
- 💡 **ახალი შესაძლებლობა**: წვდომის ხელით მიცემა **ახლა მუშაობს** (ადრე აპი წაშლიდა — იხ. ძველი გაკვეთილი ქვემოთ): `update profiles set has_pregnancy_subscription=true, pregnancy_source='admin_grant', pregnancy_until='2026-12-31' where email='...'` (`pregnancy_until=null` = უვადო). ვადა მაინც მოქმედებს — გასულ `admin_grant`-საც წვდომა ეკეტება.
- 📊 **გადამოწმებულია რეალურ მონაცემებზე** (RevenueCat CSV export × `profiles`): **16 ორსულობის გამომწერელი** (+9 Pro = 25), **ყველა 16 ემთხვევა profile row-ს `app_user_id`-ით** → არცერთი მათგანი guard-ზე არ არის დამოკიდებული (აქტიური entitlement მიცემის გზაზე მიდის, სადაც guard არ ერევა). ბაზაში დროშა **21**-ს ჰქონდა → **5 არ იხდის**, აქედან **3 დეველოპერის სატესტოა** (`isTestAccountEmail`-ით ისედაც აქვთ). 2 რეალურ user-ს user-ის გადაწყვეტილებით წვდომა ეხსნება.
- ⚠️ **11 გადამხდელს row ჯერ არ ჰქონდა დამარკირებული** (`pregnancy_source=null`) — ეს ეტაპ 1-მდე არსებული სინქრონის დანაკლისია; აპის გახსნისთანავე თავისით სწორდება.
- ✅ **ეტაპი 1-ის დადასტურება ველზე**: export-ში ერთი გამომწერელი 21 ივლისს გაუქმებულია, წვდომა 9 აგვისტოს 18:15-ზე ეწურება — ბაზის `pregnancy_until` **წამში ემთხვევა**. ძველი კოდით სამუდამოდ შეუნარჩუნდებოდა.
- ✅ 9/9 ახალი ტესტი (რეალური სცენარებიდან) + წინა 29 რეგრესიაზე.
- ⚠️ **დარჩენილი ნაპრალი**: ვინც გააუქმებს და აპს **აღარასდროს გახსნის**, row არ განახლდება. მხოლოდ webhook-ით წყდება — 16 გამომწერელზე რისკი მცირეა.

**🧾 გადახდის ისტორია + ანგარიშის გამოცვლა (2026-08-01 — committed `286bb16`, pushed, ⚠️ OTA ჯერ არ გასულა)**: აუდიტის "ეტაპები 3-4", ორივე პატარა.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mishvelidzeu-code/kalebi](https://github.com/mishvelidzeu-code/kalebi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
