---
trigger: always_on
description: Zbuduj single-page kalkulator unit economics dla Amazon US FBA seller.
---

# Amazon FBA Unit Economics Calculator — US Market

## Cel projektu

Zbuduj single-page kalkulator unit economics dla Amazon US FBA seller.
Użytkownik wpisuje wymiary i wagę produktu → kalkulator automatycznie klasyfikuje size tier
i oblicza FBA fee z wbudowanej tabeli.
Pozostałe koszty wpisywane ręcznie → kalkulator pokazuje profit, margin, ROI w czasie rzeczywistym.

Rynek: **Amazon US** (bez VAT, waluta USD).

---

## STATUS: Multi-market (US + EU) — 2026-07-03

Kalkulator obsługuje teraz **6 rynków** przez dropdown na górze: **US · DE · FR · IT · ES · UK**.
Jeden opublikowany `index.html`, przełącznik zmienia tabele fee, walutę, logikę size-tier i etykiety.

- **US** — bez zmian: logika Small/Large Standard (imperial wewnętrznie), price-tier, proximity gauges, weight-band optimiser.
- **EU (DE/FR/IT/ES/UK)** — osobny silnik metryczny:
  - Źródło danych: **`260410-FBA-Rate-Card-DE.pdf`** (Amazon "Tarifübersicht — Gebühren für Europa", ważne od 17 kwietnia 2026). PDF zawiera WSZYSTKIE kraje w jednym dokumencie; liczby identyczne w wersji DE/EN/PL. Jest też kopia PL w folderze.
  - **DE = kolumna CEP (DE/PL/CZ)** (wybór Toma), NIE "Nur DE".
  - Programy: **Standard (str. 6)** + **Low-Price (str. 5)** — toggle widoczny tylko dla EU. Low-Price = przedmioty ≤ £20/€20 (z VAT), max Small parcel ≤ 400 g.
  - Tiery metryczne: koperty (Umschlag) → paczki (Paket) → oversize. Waga wolumetryczna = **L×W×H ÷ 5000**. Dla paczek billable = max(actual, volumetric); dla kopert tylko actual.
  - Oversize → ręczne wpisanie fee (jak US).
  - **Referral fee liczona od GROSS (z VAT)** — Amazon EU nalicza Empfehlungsgebühr od Angebotspreis (cena brutto), NIE od netto (str. 24 PDF). Dlatego dla EU jest pole **Amazon Price (gross)** + **VAT %** (default per kraj: DE 19, FR 20, IT 22, ES 21, UK 20 — edytowalne, bo reduced rates dla food/books/health). User wpisuje gross → **Selling Price (net)** auto = gross/(1+VAT); pola zlinkowane dwukierunkowo (`onGrossInput/onNetPriceInput/onVatInput`, `EU_VAT`). Referral = gross × ref%. Reszta profit-math na netto. Break-even EU liczony osobno (referral skaluje z gross, PPC z net). PPC/ACoS zostaje na netto (Tom prosił tylko o referral). US bez zmian (referral od net, brak VAT).
  - Referral % — nadal ręczne pole (default 15%), wspólne dla obu rynków.
  - Uwaga: Amazon dolicza **+1.5% fuel & logistics surcharge** na shipping fees od 17 Apr 2026 — traktujemy liczby z tabeli jako bazę, weryfikować z fakturą.
- **Silnik EU zweryfikowany** standalone (9/9 testów) + integracja przez DOM-harness (US regression + DE/UK/Low-Price). Dane EU zaszyte w `index.html` jako `EU_STANDARD_ENVELOPE/PARCELS`, `EU_LOWPRICE_*`. Funkcje: `euFee()`, `computeEuFba()`, `onMarketChange()`, `setProgram()`.
- **Bugfix przy okazji:** oversize NIE ustawia już `fbaFeeManualOverride=true` (US i EU) — auto-fill wraca gdy produkt przestaje być oversize. Override ustawia tylko ręczna edycja pola FBA Fee.

- **EU Size Tier Proximity** (2026-07-03) — zbudowany osobny widok dla EU (`euProximitySection`, funkcje `updateEuProximity()`, `euBandsForTier()`, `euRenderBands()`, `EU_ZONES`). Pokazuje: 4 gauge (L/W/H/waga) z headroom do limitu tieru (container zones: Envelope 33×23×6/0.96kg → Small parcel 35×25×12/3.9kg → Standard parcel 45×34×26/11.9kg), chip driver (📐 volumetric / ⚖️ actual), linię "next tier up", oraz strip fee-by-weight-band (drop-to-cheaper / current / next-band cost + headroom). US gauges (`proximitySection`) chowane dla EU i odwrotnie. `euFee()` zwraca teraz też `dims` + `isParcel`.

- **EU Monthly Storage (2026-07-08)** — auto-liczony magazyn/szt./mies. = objętość pudełka × stawka. Dane `EU_STORAGE` (rate card str. 17): EUR (DE/FR/IT/ES) €/m³, UK £/ft³, każda `[Sty–Wrz, Paź–Gru]`, klasy: standard / apparel / oversize. Objętość = L×W×H/1e6 m³ (UK ×35.3147 na ft³). Klasa oversize brana z `res.tier==='oversize'`. UI: `euStorageControls` (select sezon + kategoria) w sekcji Koszty, `storageHint`, auto-fill pola `storage` (jak FBA fee; `storageManualOverride`, `onStorageManual/onStorageParamChange`, `euStorageFee()`). Dangerous goods pominięte. Weryfikacja: DE 25.4×17.8×7.6 → €0.09 (Sty–Wrz) / €0.18 (Paź–Gru), UK ft³ OK, override trzyma.

### Do zrobienia w przyszłości (jeśli Tom poprosi)
- EU oversize tiers (base + per-kg) — obecnie manual.
- Storage: dangerous goods + Lagernutzungszuschlag (utilization surcharge) — obecnie pominięte.
- Kolejne rynki (NL/SE/PL/BE) — dane są w tym samym PDF, wystarczy dodać kolumny.
- Referral % auto z kategorii (tabele str. 24-26).

---

## Stack

- **Jeden plik `index.html`** — HTML + CSS + JS inline, zero zależności zewnętrznych
- Zero npm, zero frameworków, zero build step
- Działa po otwarciu w przeglądarce offline
- Gotowy do deploy na GitHub Pages bez żadnej konfiguracji

---

## Struktura pól kalkulatora

### Sekcja 1: REVENUE
| Pole | Typ | Placeholder | Opis |
|------|-----|-------------|------|
| Selling Price | number, $ | 29.99 | Cena na listingu Amazon US |
| Units Sold / Month | number | 100 | Do wyliczeń miesięcznych (opcjonalne) |

### Sekcja 2: FBA FEE CALCULATOR
Użytkownik wpisuje wymiary → kalkulator automatycznie wylicza fee.

| Pole | Typ | Placeholder | Opis |
|------|-----|-------------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vente-europe/unit-economics-calculator](https://github.com/vente-europe/unit-economics-calculator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
