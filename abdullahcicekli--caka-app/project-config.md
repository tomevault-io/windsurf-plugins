---
trigger: always_on
description: Bu repo tek Cloudflare Worker'da çalışan bir link-in-bio uygulamasıdır
---

# AGENTS.md — Caka için ajan rehberi

Bu repo tek Cloudflare Worker'da çalışan bir link-in-bio uygulamasıdır
(Hono API + React Router v8 SSR, D1 + Drizzle, R2, Better Auth). Ürün **beş
dilde** yayındadır: `en`, `tr`, `es`, `pt-BR`, `de`. **Türkçe kanoniktir** —
yeni metin önce Türkçe yazılır, tip sözleşmesi ondan türer.

## Komutlar

| Komut | Not |
|---|---|
| `pnpm dev` | Vite + Miniflare (lokal D1/R2); port 5173 doluysa 5174'e kayar. **İşin bitince kapat** — bkz. "Arkada bırakma" |
| `pnpm typecheck` | Her değişiklikten sonra çalıştır; temiz olmalı |
| `pnpm test` | Vitest — `packages/shared` (saf kurallar) + `apps/web` (`tests/`, bugün Kur'an hattının kapsama sınaması) |
| `pnpm --filter @caka/web run deploy` | **Prod'a çıkışın tek yolu** (Değişmez #11): build + bekleyen D1 migration'larını `--remote` uygula + `wrangler deploy` → caka.app. **`run` sözcüğü şart:** pnpm'in rezerve `deploy` komutu script'i gölgeler; çıplak `pnpm deploy` kökte `ERR_PNPM_NOTHING_TO_DEPLOY` verir |
| `pnpm exec wrangler d1 migrations apply caka-db --local` | `apps/web` içinden; şema değişince |
| `pnpm exec wrangler types` | `wrangler.jsonc` değişince Env tiplerini yeniden üret |

## Karar kaynakları

- Yol haritası ve gerekçeler: `docs/plans/2026-08-15-001-feat-caka-mvp-plan.md`
  (R/KTD/U kimlikleri oradan gelir). Plan gövdesine ilerleme/durum YAZMA;
  ilerleme git'ten türetilir.
- Mimari özet: `ARCHITECTURE.md`. Çelişki görürsen plan kazanır.
- Ertelenmiş açık işler: `docs/backlog.md`. Bir madde bitince oradan **silinir**
  (durum git'ten okunur, "yapıldı" yazılmaz).
- **Lokalizasyon planı:** `docs/plans/2026-08-18-002-feat-lokalizasyon-plan.md`
  (`L`/`LKD` kimlikleri oradan gelir).
- Hukuki denetim kaydı: `docs/legal/` — `data-map.md` (hangi veri nerede, hangi
  sebeple), `cookie-inventory.md`, `vendor-register.md`, `trust-claims.md`
  (footer iddialarının dayanağı), `placeholders.md` (yayın kapısı). Yayındaki
  hukuki metinler bu dosyalardan yazıldı; çelişki görürsen **kod ve
  `docs/legal/` kazanır**, metin düzeltilir.

## Değişmezler (bozma)

1. **Rezerve isimler ↔ route'lar senkron:** Yeni bir top-level route eklediğinde
   `packages/shared/src/username.ts` içindeki `RESERVED_USERNAMES`'e ekle.
   `app/routes.ts`'te uygulama route'ları her zaman `:username` catch-all'undan
   önce durur.
2. **Sürüm pinleri:** `drizzle-orm@0.45.2` / `drizzle-kit@0.31.5` /
   `better-auth@1.6.28` bilinçli pinlidir. Drizzle v1 RC'ye GEÇME — klasörlü
   migration çıktısı `wrangler d1 migrations apply` ile uyumsuz. Better Auth
   yükseltmesi yapılırsa Google girişi elle smoke-test edilmeli.
3. **Migration'lar** yalnızca `drizzle-kit generate` ile üretilir
   (`packages/db/migrations/`, düz `.sql`); elle SQL dosyası ekleme.
4. **Tasarım token'ları** `apps/web/app/app.css` `@theme` bloğundadır
   (`zemin, murekkep, kirec, mavi, cam, kum, erik, mor, sinir` + türetilmiş
   `kirec-koyu`, `mor-acik`). Bileşenlere ham hex yazma; yeni renk gerekiyorsa
   önce token ekle. Radius: kılavuz 8/12/16 → `rounded-lg/xl/2xl` (shadcn'in
   oransal radius ölçeği bilinçli olarak kaldırıldı — geri koyma, kılavuzu ezer).
   shadcn'in semantik renkleri (`--popover`, `--accent`, `--border`,
   `--destructive` …) `:root`'ta marka token'larına bağlıdır; bileşen
   dosyalarındaki `bg-popover` gibi sınıflar bu yüzden markayla uyumlu.
   **İkon kütüphanesi `iconoir-react`** — lucide kaldırıldı, geri getirme.
   Iconoir'da `size` prop'u YOKTUR; ölçü `width`/`height` ile verilir.
5. **İçerik/görünüm ayrımı ve dil:** Kullanıcıya görünen her metin
   `apps/web/app/content/<alan>/{tr,en,es,pt-BR,de}.ts` kataloglarındadır;
   bileşene metin GÖMME. Türkçe dosya kanoniktir ve tipi verir
   (`export type XContent = typeof tr`), diğer dördü `satisfies XContent`
   der — bir anahtar eklenip bir dile eklenmezse `pnpm typecheck` kırılır.
   Kataloglardaki adresler Türkçe hâliyle yazılır (`/gizlilik`); render'da
   `useHref()` ile ziyaretçinin diline çevrilir. Paylaşılan katman
   (`packages/shared`) kullanıcıya görünen metin TAŞIMAZ: kimlik döndürür
   (`BlockIssueId` gibi), metni katalog kurar.
6. **Sır disiplini:** Gerçek anahtarlar yalnızca `apps/web/.dev.vars`
   (gitignore'da) ve `wrangler secret`'ta. `.dev.vars.example`'a asla gerçek
   değer yazma. Secrets: `BETTER_AUTH_SECRET`, `GOOGLE_CLIENT_ID/SECRET`,
   `APPLE_PRIVATE_KEY`. GitHub katkı grafiği sır İSTEMEZ: GitHub'ın genel
   katkı HTML parçası okunur (`server/github.ts`); parça çekilemez veya
   ayrıştırılamazsa özellik sessizce kapalıdır. Kod içinde loglara token/PII
   yazma.
7. **Auth kimliği** provider `sub`'ına bağlıdır; e-posta tabanlı account
   linking kapalıdır (`server/auth.ts`) — açma.
8. **Kullanıcı URL'leri** yalnız `http(s)` şemasıyla; kullanıcı metni meta'ya
   React Router meta API'si dışında bir yolla (string `<head>`,
   `dangerouslySetInnerHTML`) basılmaz.
9. **R2 anahtarları** düz UUID'dir (`asset.id` = R2 key); path segmentli anahtar
   üretme. Asset silme yalnızca hesap silmede yapılır (`server/account.ts`).
10. **Adres değişikliği semantiği:** eski adres 30 gün 302 yönlendirir ve
    kilitlidir (`username_redirect`). 301 KULLANMA (tarayıcı süresiz cache'ler).
    Aynı tablo silinmiş hesapların adını da 30 gün kilitler: `profile_id` NULL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abdullahcicekli/caka.app](https://github.com/abdullahcicekli/caka.app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
