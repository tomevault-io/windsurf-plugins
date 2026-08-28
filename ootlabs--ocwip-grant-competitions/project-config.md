---
trigger: always_on
description: Webowa platforma obsługująca pełny cykl życia konkursu dotacyjnego dla Opolskiego Centrum Wspierania Inicjatyw Pozarządowych. OCWIP występuje po stronie ORGANIZATORA konkursu, nie wnioskodawcy.
---

# OCWIP Generator konkursów - instrukcje dla agentów

Webowa platforma obsługująca pełny cykl życia konkursu dotacyjnego dla Opolskiego Centrum Wspierania Inicjatyw Pozarządowych. OCWIP występuje po stronie ORGANIZATORA konkursu, nie wnioskodawcy.

**Stack:** Next.js (TypeScript, Tailwind CSS) - .NET - PostgreSQL - Docker Compose

> **Jedno źródło prawdy.** `CLAUDE.md` importuje ten plik, `.cursor/rules/` na niego wskazuje. Edytuj **ten** plik, nigdy nie kopiuj reguł do pozostałych.

---

## Stan repozytorium

To jest **szkielet**. Świadomie nie ma tu jeszcze:

brak encji domenowych - brak logowania i ról - brak kreatora formularzy - brak modułu oceny - brak generowania umów - brak sprawozdawczości - brak wysyłki maili

Co jest: trzy kontenery, które się budują i widzą nawzajem, health endpointy, infrastruktura migracji EF Core (bez tabel domenowych), testy z CI oraz dokumentacja i kontekst projektu. Każdy z brakujących elementów ma swoją kartę na Trello. Nie buduj ich "przy okazji".

Zakres MVP i świadome cięcia: [`docs/zakres.md`](docs/zakres.md). Jeśli zaczynasz robić coś, czego tam nie ma, przerwij i zapytaj.

---

## Kontekst domenowy - przeczytaj przed pierwszą zmianą

| Chcę zrozumieć | Czytaj |
|---|---|
| Kto jest klientem i jaki problem rozwiązujemy | [`docs/kontekst-projektu.md`](docs/kontekst-projektu.md) |
| Słownik pojęć (oferta, nabór, konkurs, generator) | [`docs/slownik.md`](docs/slownik.md) |
| Role, typy podmiotów, twarde reguły biznesowe | [`docs/reguly-biznesowe.md`](docs/reguly-biznesowe.md) |
| Co świadomie NIE wchodzi do MVP | [`docs/zakres.md`](docs/zakres.md) |

Używaj słownika z `docs/slownik.md` w UI, w nazwach endpointów i w rozmowie z zamawiającym. Zamawiający opisuje system językiem Witkaca. Jeśli nasze nazwy będą inne, każde spotkanie zacznie się od tłumaczenia pojęć.

---

## Gdzie szukać - zrób to zamiast przeszukiwania repo

Ślepy grep zamienia pięciominutową zmianę w godzinę. Zacznij stąd za każdym razem:

| Co robisz | Czytaj najpierw |
|---|---|
| Szukasz pliku, który robi X | `docs/map/README.md`, potem mapa obszaru |
| Zmiana w backendzie | `docs/map/backend.md` |
| Zmiana we froncie | `docs/map/frontend.md` |
| Docker, baza, skrypty, zmienne środowiskowe | `docs/map/infra.md` |
| Jak system trzyma się do kupy | `docs/architektura.md` |
| Nazewnictwo, struktura folderów, styl | `docs/konwencje.md` |
| Model danych i jawne założenia | `docs/model-danych.md` |
| Pisanie i uruchamianie testów, CI | `docs/testy.md` |
| Co się ostatnio zmieniło i dlaczego | `docs/log.md` - **tylko kilka górnych wpisów** |

**Reguła:** najpierw mapa, grep drugi. Jeśli mapa czegoś nie miała, mapa była zła, więc popraw ją w ramach swojej zmiany.

Ładuj tylko obszar, którego dotykasz. Czytanie wszystkich map po to, żeby zmienić jedną zmienną CSS, to dokładnie to marnotrawstwo, przed którym ta struktura ma chronić.

---

## Język (zabija "Ponglish")

- **Kod: angielski.** Identyfikatory, komentarze, nazwy plików źródłowych, komunikaty commitów, ścieżki API, kolumny w bazie.
- **Dokumentacja i UI: polski.** Cała treść w `docs/`, teksty w interfejsie, etykiety, dane testowe widoczne w produkcie.
- **Czat: dowolny język.** Nie zmienia niczego powyżej.

Powód tego podziału jest konkretny: kod czytają narzędzia i agenci, a dokumentację czyta zespół i strona zamawiająca, która nie pracuje po angielsku. Bez mieszania w obrębie jednego artefaktu. Polski identyfikator w kodzie albo angielska etykieta w polskim UI wracają z review.

## Interpunkcja (reguła twarda)

**Nigdy nie używaj myślnika typograficznego (em dash) ani półpauzy (en dash).** Ani w kodzie, ani w komentarzach, dokumentacji, commitach, tekstach UI czy odpowiedziach do użytkownika. Użyj przecinka, dwukropka, nawiasu albo zwykłego dywizu `-`.

`scripts/check_text.py` tego pilnuje, a hook pre-commit blokuje commit. Nie ma wyjątku wartego dyskusji.

---

## Bezpieczeństwo i dane osobowe - to nie jest dodatek

System przetwarza dane organizacji i osób fizycznych, a przy umowach pojawiają się PESEL-e. To podnosi poprzeczkę dla każdej karty, nie tylko dla tych oznaczonych jako bezpieczeństwo.

1. **Brak reguły oznacza brak dostępu.** Domyślną odpowiedzią jest odmowa. Endpoint, o którym ktoś zapomniał, ma być niedostępny, a nie otwarty dla wszystkich.
2. **Wnioskodawca nigdy nie widzi cudzego wniosku.** Reguła bez testu automatycznego to życzenie, nie reguła.
3. **Nie ujawniamy, czy konto istnieje.** Rejestracja, logowanie i reset hasła odpowiadają tak samo dla adresu zajętego i wolnego.
4. **Nie logujemy haseł ani danych wrażliwych.** Log z hasłami to gorszy wyciek niż ten, przed którym się bronimy.
5. **Nie kasujemy twardo.** Retencja minimum 5 lat. "Usunięcie" to oznaczenie jako nieaktywne.
6. **Każde pole trzymające dane wrażliwe oznacz komentarzem w kodzie**, żeby przy szyfrowaniu nikt niczego nie przeoczył.

---

## Git

**Dwie gałęzie żyją na stałe.** Nigdy nie commituj bezpośrednio do żadnej z nich.

- **`main` to produkcja.** Zawsze wdrażalna. Ląduje tu tylko merge release z `dev` albo `hotfix/`.
- **`dev` to gałąź integracyjna.** Domyślna. Odbijasz od niej i wracasz do niej.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ootLabs/OCWIP-grant-competitions](https://github.com/ootLabs/OCWIP-grant-competitions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
