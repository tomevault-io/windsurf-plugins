---
trigger: always_on
description: Pokyny pro práci s tímto repozitářem (Claude Code, Codex, Cursor, Copilot a další).
---

# AGENTS.md — pravidla pro AI agenty a přispěvatele

Pokyny pro práci s tímto repozitářem (Claude Code, Codex, Cursor, Copilot a další).
Platí pro celý repozitář. Obecný popis projektu je v [README.md](README.md).

## O projektu

MyUcto.cz — český self-hosted fakturační a účetní systém (vystavené + přijaté
faktury, multi-supplier, DPH/KH/SH výkazy, EPO XML, CRM, REST API).
Backend PHP 8.5 + Slim 4, frontend Vue 3 + TypeScript + Vite + Tailwind,
databáze MariaDB 11.8+.

## Layout repozitáře

- `api/` — PHP backend (Slim, autowired actions, services, repositories); `api/bin/` = CLI skripty, `api/tests/` = PHPUnit
- `web/` — Vue 3 + TS frontend; zdrojáky ve `web/src/`, lokalizace ve `web/src/i18n/`
- `dist/` — produkční build frontendu (commitovaný — uživatelé testují přes něj)
- `db/migrations/` — SQL migrace (číslované, idempotentní)
- `manual/` — uživatelský manuál (Markdown, česky); `manual/generated/` = vyrenderované HTML
- `tools/` — pomocné skripty (generování manuálu, převody obrázků)
- `cmd/` — cron/deploy wrappery (`.sh` + `.cmd`/`.ps1`)

## Fork MyÚčto.cz — pravidla portování

- Upstream: `git@github.com:radekhulan/myinvoice.git` (remote `upstream`, VEŘEJNÉ) —
  merguj `upstream/master` pravidelně.
- Repozitáře:
  - **`radekhulan/myucto`** — veřejné, **tady běží vývoj i PR přispěvatelů**.
    Jeho historie začíná veřejnou historií MyInvoice po `7dd51638` (4.51.0) a nad
    ní je jeden commit s celým stromem MyÚčta. Díky tomu **sdílí merge base
    s upstreamem**, takže `git merge upstream/master` jde spustit přímo tady.
  - **`radekhulan/myucto-dev`** — privátní ARCHIV původní historie (1600+ commitů,
    reálná data zákazníků). Vývoj proti němu neběží, nepushuje se do něj.
- Proč to takhle: MyInvoice je velmi živý (~1000 commitů/90 dnů), takže ztráta
  merge base by znamenala ruční portování stovek commitů. Squash vlastní historie
  do jednoho commitu skryje interní data, ale předka s upstreamem zachová.
- Kód piš **aditivně**: nové moduly do nových souborů; sdílené soubory jen doplňuj
  malými lokalizovanými změnami, ať zůstane merge z upstreamu levný.
- MyÚčto-specifické migrace čísluj od `1000_` — range `0125`–`0999` je rezervovaný
  pro upstream.
- Namespace `MyInvoice\` a interní identifikátory (env vary, cookie/localStorage/Redis
  klíče) se **NEpřejmenovávají** — mění se jen user-visible branding (UI texty,
  e-maily, dokumentace, loga).
- DB baseline a minimální podporovaná verze je **MariaDB 11.8 LTS**.

## Příkazy

```bash
# Frontend — build (NUTNÉ po každé změně web/src, dist/ se commituje)
cd web && pnpm build            # = vue-tsc --noEmit && vite build (npm run build funguje též)
cd web && pnpm type-check       # jen typová kontrola

# PHP testy (PHPUnit 13)
cd api && php vendor/bin/phpunit                  # vše
cd api && php vendor/bin/phpunit --filter Xyz     # podmnožina

# Migrace — VŽDY přes migrate.php, NIKDY mysql klientem přímo
php api/bin/migrate.php
php api/bin/migrate.php --status

# Manuál — po změně manual/*.md regenerovat pouze HTML
php tools/generateManualHtml.php
```

## Tvrdá pravidla

### Migrace
- Nová migrace = nový číslovaný soubor v `db/migrations/`, spouští se **výhradně** přes `php api/bin/migrate.php`.
- Každá migrace musí být **idempotentní** (opakovatelně spustitelná): používej nativní MariaDB `IF [NOT] EXISTS` (`ADD COLUMN IF NOT EXISTS`, `CREATE TABLE IF NOT EXISTS`, …), ne PREPARE/EXECUTE triky.
- Cílová DB je MariaDB 11.8+: v SQL preferuj **window functions a CTE** před vnořenými subselecty; nepoužívej `SQL_CALC_FOUND_ROWS`.

### i18n
- Veškeré nové UI texty přes `t()` z vue-i18n — **nikdy** natvrdo česky/anglicky v šablonách. Vždy doplň **obě** locale (`web/src/i18n/cs.json` i `en.json`).
- Pole/seznamy překladů přes `tm()` + `rt()` — `t()` pole stringifikuje.
- Literální `{` `}` v textu zprávy escapuj jako `{'{token}'}` — jinak to vue-i18n bere jako interpolaci a render tiše spadne.

### OpenAPI sync
- Při **jakékoli** změně veřejného API (nová route, změna serializace, nový/změněný sloupec promítnutý do JSON, nové query/body pole) ihned aktualizuj `api/openapi.yaml` — jak `paths` (`/api/v1/*`), tak `components/schemas`.
- Po editaci ověř: YAML se parsuje, žádné duplicitní klíče (PyYAML je tiše přepíše — použij striktní loader), žádné dangling `$ref`.
- Veřejné API je kurátorovaný read-only subset; mutace číselníků a interní plumbing se nedokumentují.

### DPH a daňová správnost
- Veškerá evidence DPH jde přes `VatLedgerService` — nikdy neobcházet vlastním SQL.
- Výkazy a rekapitulace sumují **řádky** (`invoice_items` / per-řádkové totály), ne hlavičku dokladu.
- Při zásahu do daní/DPH proaktivně ověř daňovou správnost (zařazení do správného období), ne jen „napojení na existující kód". Kontroluj **symetrii** filtrů: obě strany evidence proti všem typům dokladů (`invoice_type` vs `document_kind`); proforma = záloha na vstupu.
- Každá nová cesta, která tvoří doklad z jiného dokladu (proforma → faktura, dobropis, kopie, recurring), musí přenést `prices_include_vat` — jinak se brutto cena chybně přepočítá jako netto.
- Agregace nákladů z `purchase_invoices` musí vyřadit spárované/zaplacené zálohové doklady (jinak se náklad počítá 2×).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radekhulan/myucto](https://github.com/radekhulan/myucto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
