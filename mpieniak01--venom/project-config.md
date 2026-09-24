---
trigger: always_on
description: Ten plik zawiera **instrukcje dla agentów kodowania** pracujących w tym repozytorium.
---

# Wytyczne dla Agentów Kodowania (PL)

Ten plik zawiera **instrukcje dla agentów kodowania** pracujących w tym repozytorium.

Jeśli szukasz listy agentów systemu Venom, użyj:
- [SYSTEM_AGENTS_CATALOG.md](SYSTEM_AGENTS_CATALOG.md)

## Zasady Bazowe

- Wprowadzaj zmiany małe, testowalne i łatwe do review.
- Utrzymuj jakość typowania (`mypy venom_core` powinno przechodzić).
- Utrzymuj bezpieczeństwo (znaleziska Sonar/Snyk naprawiamy, nie ignorujemy).
- Unikaj martwego kodu i placeholderów.
- Ścieżki błędów mają być jawne i, gdzie sensowne, pokryte testami.
- Przed uruchamianiem narzędzi Pythona aktywuj środowisko repo: `source .venv/bin/activate`.
- Domyślnym językiem odpowiedzi agentów workspace jest polski, chyba że user poprosi o angielski albo wymagany jest zewnętrzny format.

## MCP First dla pytań OpenAI/Codex

Gdy zadanie dotyczy użycia produktów OpenAI/Codex, modeli lub zachowania API:

1. Najpierw sprawdź oficjalną dokumentację przez Docs MCP.
2. Pliki repo traktuj jako źródło prawdy dla zachowania projektowego.
3. W raporcie podawaj krótki wniosek zamiast długiego bloku cytowanej dokumentacji.
4. Jeśli dokumentacja i kod są niespójne, zgłoś to jawnie przed implementacją.

To ogranicza wzrost kontekstu i zmniejsza token burn.

## Lekki Brief Kontekstowy (artefakt CI)

Jeśli jest dostępny, zaczynaj od `test-results/agent-context/preflight-brief.json` (lub `preflight-brief.md`), a potem `brief.json`, zanim wejdziesz w szerokie czytanie repo.
Traktuj go jako krótki punkt startowy dla:

1. kolejności bramek,
2. skrótu metadanych lane/testów,
3. reguły użycia Docs MCP dla tematów OpenAI/Codex.

Jeśli brief i kod są niespójne, źródłem prawdy pozostaje kod.

## Najczęstsze Pułapki

- Najpierw ustal źródło prawdy: gałąź, PR, dokument zadania i dokładny zakres, który faktycznie jest w grze.
- Nie mieszaj w głowie pracy funkcjonalnej z naprawami jakościowymi; jeśli jedna gałąź zastępuje drugą, nazwij to wprost.
- Zanim zmienisz UI albo dokumentację, potwierdź rzeczywisty przepływ danych. Tekst wyglądający jak placeholder nie jest jeszcze dowodem.
- Artefakty runtime traktuj jak element higieny repo. Jeśli zadanie generuje pliki lokalne, dodaj je do `.gitignore` od razu.
- Gdy Sonar lub hotspot bezpieczeństwa wskazuje problem, wybieraj dozwolone namespace, jawne ścieżki i liniowe parsowanie zamiast dynamicznego składania ścieżek lub ciężkich regexów.
- Przy failu bramki najpierw napraw root cause i odpal najmniejszy celowany test, a dopiero potem `make pr-fast`.
- Jeśli ten sam gate failuje dwa razy bez zmiany kodu albo środowiska, zatrzymaj się i zgłoś bloker zamiast kręcić pętlę.
- Aktualizacje dokumentacji mają być krótkie i użyteczne: zapisz regułę, która zapobiegnie kolejnemu błędowi, nie cały przebieg debugowania.

## Domyślna Komenda Testów (zacznij od tego)

Gdy zakres testów nie jest jeszcze doprecyzowany, zacznij od:

```bash
source .venv/bin/activate
pytest -q
```

## Anty-Loop: „nie ten Python” (obowiązkowe)

Jeśli w terminalu widzisz `(.venv)`, to lokalnie zwykle wszystko działa poprawnie (`python`, `python3`, `pytest`).
Najczęstszy problem agentów: komendy lecą w **nowej powłoce**, która **nie dziedziczy** Twojego `source .venv/bin/activate`.

Zasada bezpieczna (preferowana w automatyzacji):

```bash
/home/ubuntu/venom/.venv/bin/python -V
/home/ubuntu/venom/.venv/bin/pytest -q
```

Alternatywa (w jednej komendzie, gdy używasz aktywacji):

```bash
cd /home/ubuntu/venom && source .venv/bin/activate && pytest -q
```

Szybka kontrola kontekstu:

```bash
which python
which pytest
```

Obie ścieżki muszą wskazywać na `/home/ubuntu/venom/.venv/...`.

## Kontrakt Dostarczania Bez Limitów Czasu (obowiązkowy)

To jest domyślny tryb pracy dla GitHub Coding Agent i ma pierwszeństwo przed długą eksploracją.

Obietnice oparte o minuty są zabronione. Agent może być wznawiany/wstrzymywany z zewnątrz, więc limity minutowe nie są wiarygodnym mechanizmem kontroli.

Checkpointy postępu:

1. tylko preflight (`git status`, pliki docelowe, wymagane narzędzia/env),
2. implementacja minimalnego zakresu end-to-end,
3. pierwszy commit od razu po przygotowaniu pierwszego spójnego wycinka zmian (WIP dozwolony, nawet jeśli gate nie są jeszcze zielone),
4. domknięcie zakresu + testy celowane,
5. `make pr-fast`, poprawki blockerów, raport końcowy.

Twarde zasady stop:

1. Brak ponownej eksploracji repo po rozpoczęciu implementacji.
2. Maksymalnie jedno wywołanie subagenta na fazę (explore/implement/verify).
3. Jeśli dwie kolejne iteracje pracy nie wnoszą zmiany w kodzie ani zmiany w testach, przerwij i zgłoś bloker.
4. Jeśli ten sam gate failuje 2 razy bez zmiany kodu/środowiska, przerwij i zgłoś bloker.
5. Nie uruchamiaj ciężkich checków nieobowiązkowych przed zielonym `make pr-fast`.

Dyscyplina commitów:

1. Pierwszy commit ma powstać od razu po pierwszym spójnym wycinku zmian; nie odkładaj wszystkiego na koniec.
2. Preferuj 1-3 małe commity zamiast jednego dużego na końcu.
3. Nie odkładaj wszystkich commitów na koniec długiej pętli debugowania.

## Szybki Bootstrap (instalacja pakietów)

Gdy stan środowiska jest niepewny, użyj tej sekwencji:

```bash
test -f .venv/bin/activate || python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpieniak01/Venom](https://github.com/mpieniak01/Venom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
