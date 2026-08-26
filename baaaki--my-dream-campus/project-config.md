---
trigger: always_on
description: Universite yonetim sistemi. Full-stack monorepo: Go moduler monolith (`new-backend/`) + React+Vite web + React Native (Expo) mobil. Eski mikroservis kodu main'den cikarildi — `v0-microservices` git tag'inde arsivli.
---

# MyDreamCampus — AI Asistanı Talimatları

Universite yonetim sistemi. Full-stack monorepo: Go moduler monolith (`new-backend/`) + React+Vite web + React Native (Expo) mobil. Eski mikroservis kodu main'den cikarildi — `v0-microservices` git tag'inde arsivli.

> Bu dosya **AI'a** talimattir. Kullanici dokumanlari icin bkz. `README.md`.

---

## 1. Cakisma Hiyerarsisi

Cakisma durumunda **yukaridan asagiya** dogru oncelik (1 en yuksek):

1. **Kullanici prompt'u** — en yuksek oncelik
2. **Ilgili `skills.md`** — `new-backend/skills.md`, `frontend/skills.md`, `mobile/skills.md`
3. **Bu dosya (CLAUDE.md)** — proje genel kurallari
4. **Memory** — gecmis konusmalardan sabitler

---

## 2. Zorunlu Okuma Kosullari

Gorev baslamadan **mutlaka oku**:

| Eger suraya dokunacaksan… | Once oku |
|---|---|
| `new-backend/monolith/**` | `new-backend/skills.md` |
| `new-backend/services/**`, `new-backend/shared/**` | `new-backend/skills.md` |
| `frontend/src/**` | `frontend/skills.md` |
| `mobile/app/**`, `mobile/services/**`, `mobile/hooks/**` | `mobile/skills.md` |
| Migration / SQL / sqlc | `new-backend/skills.md` §3-4 (make komutlari + workflow) |

Birden fazla katmanda degisiklik varsa **hepsini** oku.

---

## 3. Konusma Dili & Kod Dili

- **Konusma dili (kullaniciyla)**: Turkce
- **Kod, degisken, dosya adi, commit mesaji**: Ingilizce
- **Kullaniciya hata mesaji (UI text)**: Turkce
- **Log mesaji**: Ingilizce

**YAPMA:** Turkce degisken adi (`kullaniciAdi` ❌, `username` ✅).
**YAPMA:** Ingilizce kullaniciya hata mesaji (`"User not found"` ❌, `"Kullanici bulunamadi"` ✅ — UI'da).

---

## 4. Paket Yoneticisi (kritik — yanlis kullanma)

| Dizin | Komut | YAPMA | Neden |
|---|---|---|---|
| `frontend/` | `bun add`, `bun run`, `bun tsc`, `bunx --bun <x>` | `npm`, `npx`, `yarn` | `bun.lock` source-of-truth; `package-lock.json` yok, npm bagimliliklari farkli cozuyor |
| `mobile/` | `npm install`, `npm run`, `npx expo`, `npx jest` | `bun` (eskiden vardi, kaldirildi) | Expo prebuild scriptleri npm assumption ile yazilmis, `package-lock.json` source |
| `new-backend/monolith/` | `go mod`, `make sqlc-<module>`, `make migrate-up-<module>` (Makefile bu dizinde) | dogrudan `goose`, `sqlc generate` | Makefile `.env` ve modul basina `sqlc.yaml` cozumlemesi yapiyor; ciplak komut config bulamaz |

---

## 5. Docker / sudo Kurali

- Docker komutlari `sudo` gerektiriyor (kullanici `docker` grubunda degil).
- **Sandbox `sudo` calistirmaz** — komutu **kullaniciya kopyala-yapistir** olarak goster, kendin calistirma.

```bash
# Bunu sen calistirma — kullaniciya goster:
sudo docker compose -f new-backend/infrastructure/docker-compose.yml up -d
sudo docker exec mydreamcampus-postgres psql -U postgres -d mydreamcampus -c "SELECT email FROM auth_users;"
sudo docker logs -f mydreamcampus-monolith
```

---

## 6. Onaysiz Yapilabilecekler vs. Sorulacaklar

### SORMA, dogrudan uygula:
- HTTP status code secimi (REST standardi: 200/201/204/400/401/403/404/409/422/500)
- Sifre hash (Argon2id), JWT (HS256) — sabit
- Migration **yazma** (dosya olusturma)
- sqlc query yazma + `make sqlc-<module>` calistirma
- DTO/Repository/Service/Handler iskeleti (skills.md sablonlarini izle)
- Test isimlendirme: `TestXxx_Scenario_ExpectedResult`
- Commit atma (atomic, feature bittiginde)
- Hata mesaji standardi (`platform/errors.AppError`)

### SOR, dogrudan UYGULAMA:
- **Yeni kutuphane** ekleme (ornek: validator icin go-playground vs ozzo)
- **Yeni modul veya servis** olusturma (scope, event semasi, main.go wiring)
- **Yeni event** semasi veya mevcut event payload degisikligi (geriye uyumsuzluk)
- **Migration CALISTIRMA** (`make migrate-up-<module>`) — yazma degil, calistirma sor
- **Sema breaking change** (kolon silme, NOT NULL ekleme, type degisikligi)
- **Frontend route silme** veya yeniden adlandirma
- Onemli refactor (3+ dosya etkileyen, davranis degisikligi)
- `go.mod` / `package.json` dependency guncelleme (patch haric)

---

## 7. Git Commit Formati

```
<type>(<scope>): <description>
```

| Type | Ne zaman |
|---|---|
| `feat` | Yeni ozellik |
| `fix` | Bug fix |
| `chore` | Build, infra, tooling |
| `refactor` | Davranis degismeden yeniden yazim |
| `docs` | Dokumantasyon |
| `test` | Sadece test ekleme |

**Scope:** `auth`, `staff`, `student`, `catalog`, `enrollment`, `attendance`, `grades`, `meal`, `payment`, `notification`, `shared`, `frontend`, `mobile`, `infra`

**Ornekler:**
```
feat(auth): add login and register endpoints
fix(shared): resolve logger initialization bug
chore(infra): update caddy configuration
feat(frontend): add student dashboard page
feat(mobile): implement attendance screen
```

**Kural:** Her ozellik tamamlanınca **HEMEN** commit. Atomic — bir commit bir mantiksal degisiklik.

**YAPMA:**
- `feat: stuff` (scope yok, aciklama yok)
- `update files` (type yok)
- 10 dosya tek commit'te birden cok ozellik
- `--amend` ile push edilmis commit'i degistirme
- `--no-verify` (hook bypass)

---

## 8. Is Bittiginde Checklist (Gorev Kapatmadan Once)

```
Backend feature:
- [ ] Migration yazildi + `make migrate-up-<module>` test edildi (kullanici calistirir)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Baaaki/my-dream-campus](https://github.com/Baaaki/my-dream-campus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
