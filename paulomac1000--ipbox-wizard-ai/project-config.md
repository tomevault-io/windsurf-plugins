---
trigger: always_on
description: Instrukcja dla agentów pracujących z repozytorium `ipbox-wizard-ai`.
---

# AGENTS.md

Instrukcja dla agentów pracujących z repozytorium `ipbox-wizard-ai`.

Najpierw ustal, **w jakim trybie pracujesz**. Ten sam projekt może służyć do analizy dokumentów podatnika albo do rozwijania kodu. Nie mieszaj tych zadań bez wyraźnego polecenia użytkownika.

## Tryb 1: analiza rozliczenia użytkownika

Wybierz ten tryb, gdy użytkownik przekazał dokumenty, archiwum ZIP lub link do repozytorium i chce przygotować albo sprawdzić rozliczenie IP Box.

### Kolejność pracy

1. Przeczytaj `README.md` i `ipbox_algorytm.md`.
2. Zinwentaryzuj wszystkie dokumenty użytkownika.
3. Samodzielnie odczytaj PDF, XLSX, CSV i inne załączniki.
4. Przy każdej istotnej wartości wskaż dokument, stronę, arkusz albo wiersz źródłowy.
5. Pokaż użytkownikowi wyekstrahowane dane przed finalnym obliczeniem.
6. Zapytaj o braki. Brak danych nie jest zerem ani korzystnym założeniem.
7. Przygotuj znormalizowane dane robocze wewnętrznie — użytkownik nie musi pisać YAML-a.
8. Wykonaj krytyczne obliczenia kodem z `python_helper/`, nie w pamięci modelu.
9. Porównaj wynik z KPiR, ewidencją IP Box i formularzami PIT.
10. Oddziel:
    - błąd dokumentu lub rozliczenia;
    - brak danych;
    - obszar wymagający decyzji podatkowej;
    - możliwy błąd algorytmu.
11. Nie zmieniaj kodu, nie twórz commitów i nie modyfikuj testów, chyba że użytkownik wyraźnie zleci rozwój projektu.
12. Nie zapisuj prywatnych dokumentów ani danych podatnika w repozytorium.

### Raport pokrycia konkretnego przypadku

Na końcu każdej analizy podaj jawny status pokrycia:

```text
COVERED_DIRECTLY | COVERED_PARTIALLY | NOT_COVERED
```

Następnie wskaż:

- testy jednostkowe, które chronią istotne reguły;
- scenariusze z `tests/llm/scenarios/`, które odtwarzają tę samą ścieżkę biznesową;
- elementy przypadku, które nie mają bezpośredniego odpowiednika;
- czy odpowiednie scenariusze mają kompletną, aktualną macierz VCR.

Nie mów, że „dokładnie ten przypadek został potwierdzony przez siedem rodzin AI”, jeżeli znalazłeś jedynie podobny test albo częściowe pokrycie. Takie stwierdzenie jest dozwolone wyłącznie wtedy, gdy:

1. przypadek ma bezpośredni scenariusz biznesowy;
2. wynik scenariusza odpowiada analizowanemu invariantowi;
3. kasety wszystkich wymaganych rodzin są kompletne i aktualne;
4. playback przechodzi bez sekretu.

W pozostałych sytuacjach napisz precyzyjnie, które fragmenty są potwierdzone, a które wymagają nowego testu.

### Nowy lub nieobsłużony przypadek

Jeżeli przypadek nie jest pokryty albo ujawnia możliwy błąd:

1. nie kopiuj danych podatnika do repozytorium;
2. zredukuj problem do minimalnego przykładu syntetycznego;
3. zachowaj relacje matematyczne, ale zmień kwoty i identyfikatory;
4. opisz wynik rzeczywisty, oczekiwany i źródło oczekiwania;
5. ustal, czy problem dotyczy ekstrakcji dokumentu, danych wejściowych, reguły podatkowej czy implementacji;
6. poinformuj użytkownika, że taki przypadek może stać się trwałym testem regresyjnym.

Jeżeli agent ma dostęp do GitHuba, może utworzyć Issue **dopiero po zgodzie użytkownika**. Użyj formularza:

`https://github.com/paulomac1000/ipbox-wizard-ai/issues/new?template=new-tax-case.yml`

Jeżeli agent nie ma dostępu do zapisu, przygotuj gotowy tytuł i treść zgłoszenia oraz podaj ten link. Treść musi być zanonimizowana i zawierać minimalny przypadek, a nie pełne dokumenty.

Nowy poprawny przypadek jest twardą granicą jakości: po dodaniu regresji algorytm nie może zostać zaakceptowany, jeżeli ten przypadek nie przechodzi.

## Tryb 2: rozwój kodu w Codex, Claude Code lub podobnym narzędziu

Wybierz ten tryb wyłącznie wtedy, gdy użytkownik wyraźnie prosi o zmianę kodu, testów, dokumentacji albo infrastruktury projektu.

### Pierwsze 5 minut

Przeczytaj w tej kolejności:

1. `README.md` — produkt, sposoby użycia i granice wejścia.
2. `AGENTS.md` — zasady pracy z kodem.
3. `ipbox_algorytm.md` — domenowy kontrakt i kolejność decyzji.
4. Pliki związane z zadaniem.
5. Odpowiadające im testy jednostkowe i scenariusze.

Następnie:

1. sprawdź bieżący branch i SHA;
2. uruchom test celowany;
3. odtwórz problem minimalnym testem regresyjnym;
4. dopiero potem zmień implementację;
5. przed zakończeniem uruchom pełną bramkę jakości.

## Misja projektu

Utrzymuj audytowalne, fail-closed narzędzie wspierające przygotowanie i kontrolę danych do rozliczenia IP Box programisty B2B.

Projekt nie jest poradą podatkową ani generatorem gotowego zeznania. Wynik musi zostać zweryfikowany przez księgową albo doradcę podatkowego.

## Granica wejścia

Kod przyjmuje znormalizowany YAML/dict. Repozytorium nie zawiera kompletnego, uniwersalnego importera surowych PDF, XLSX, KPiR ani PIT.

W trybie rozmowy ekstrakcję wykonuje agent. W trybie programistycznym nie zakładaj, że kalkulator naprawi błędnie odczytane dane. Warstwa ekstrakcji musi zachować jawne fakty źródłowe, między innymi:

- rok i formę opodatkowania;
- kwalifikację prawa i faktury;
- przychód IP/NIE;
- czas pracy i semantykę `W`;
- `KUP`, koszyk kosztu i informację, czy pozycja pozostała w KPiR;
- metodę i źródło alokacji MIX;
- dowody NEXUS;
- ZUS, zdrowotną, ulgi, straty i zaliczki.

## Topologia projektu

| Obszar | Odpowiedzialność |
|---|---|
| `README.md` | główny punkt wejścia dla użytkownika |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulomac1000/ipbox-wizard-ai](https://github.com/paulomac1000/ipbox-wizard-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
