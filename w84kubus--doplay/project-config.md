---
trigger: always_on
description: Multiplayerowe gry imprezowe w przeglądarce. Każdy gracz na swoim telefonie,
---

# Doplay — pamięć projektu

## Co to jest

Multiplayerowe gry imprezowe w przeglądarce. Każdy gracz na swoim telefonie,
wspólny pokój z 4-znakowym kodem. Domena: **doplay.pl**

Do sierpnia 2026 aplikacja nazywała się Domówka. Stąd klucze `domowka-locale`,
`domowka-session` i projekt Firebase `domowka-39gd0` — **tych nazw nie zmieniamy**:
identyfikują dane już zapisane w przeglądarkach graczy i w backendzie.

Gry (9, wszystkie w `registry.ts`): Stoper, Państwa-miasta, Wisielec, Impostor,
Mafia, Odcień, Kasyno, Kółko i krzyżyk, Chińczyk.

Stack: Next.js 15 (App Router) + TypeScript strict + Tailwind v4 + Firebase (Firestore + Anonymous Auth) + Vercel.

## Specyfikacja

Pełny spec: **`SPEC.md`** w rootcie repo. Nie jest ładowany automatycznie — czytaj na żądanie.

- Przed pracą nad grą: przeczytaj sekcję 3 (architektura) + sekcję tej gry (5.x).
- Po `/compact`: przeczytaj sekcję 3 ponownie.

## Zasady nienegocjowalne

1. **IMPORTANT: Klient NIGDY nie zapisuje stanu gry do Firestore.** Wszystkie zapisy idą przez Route Handlery + `firebase-admin`. Klient tylko czyta (`onSnapshot`). Złamanie tej zasady rozwala Mafię i Impostora — role byłyby widoczne w DevToolsach.

2. Role, tajne hasła i odpowiedzi innych graczy nigdy nie trafiają do `publicState`. Tajne dane żyją w `rooms/{kod}/secret/state` (reguła: `allow read: if false`) i `rooms/{kod}/private/{uid}`.

3. `engine.ts` każdej gry jest **czystą funkcją**. Zero `Date.now()`, zero `Math.random()`. Czas i losowość wchodzą przez `ctx.now` i `ctx.rng`.

4. Dodanie nowej gry = nowy folder w `src/games/` + wpis w **rejestrach** (pełna lista w konwencjach niżej). **Zero zmian w logice rdzenia** — żadnych warunków w stylu „jeśli gra == X" w `GameShell`, `game-runner` czy na stronie pokoju. Jeśli uważasz, że rdzeń wymaga zmiany — zatrzymaj się i zapytaj. Nie zmieniaj po cichu.

5. **Kod i komentarze po polsku. Interfejs dwujęzyczny** — każdy tekst widoczny dla gracza przez `t("klucz")` z `dict.ts`, zero napisów na sztywno. Fonty muszą mieć `latin-ext` (Ą Ć Ę Ł Ń Ó Ś Ź Ż). **Press Start 2P, Orbitron i VT323 nie mają polskich znaków — nie używaj.**

6. Mobile-first. Przyciski min. 56 px. `100dvh`, nie `100vh`. Pomiar czasu w Stoperze: `performance.now()`, nigdy `setInterval`.

7. `export const runtime = 'nodejs'` w każdym Route Handlerze — `firebase-admin` nie działa na Edge.

## Styl pracy

- Rób tylko bieżącą fazę. Nie refaktoruj rzeczy spoza zakresu.
- Gdy wahasz się między dwoma podejściami — przedstaw oba i zapytaj, nie decyduj sam.
- Przed zgłoszeniem „gotowe" odpal `npm test` i `npm run build`. Oba muszą przejść.
- Commit po każdej fazie, nie jeden wielki commit na końcu.

## Komendy

```
npm run dev
npm run build
npm test
npm run lint
firebase deploy --only firestore:rules
```

## Stan projektu

<!-- odhaczaj po każdej fazie -->

- [x] Faza 0 — setup, Firebase, auth, deploy hello-world na Vercel
- [x] Faza 1 — pokoje, lobby, presence, reconnect, QR, ekran hosta
- [x] Faza 2 — silnik gier + registry (walidacja na Stoperze; kółko i krzyżyk pominięte wtedy na życzenie Jakuba, dorobione później)
- [x] Faza 3 — Stoper: oba tryby gotowe — A „CEL" i B „ZGADNIJ CZAS" (rotacja Biegacza, beep/klik do wszystkich, typowanie stepperem; czas i typy tajne do odsłonięcia)
- [x] Faza 4 — Państwa-miasta (zweryfikowane na produkcji: tajność pisania, kwestionowanie, dedup, punktacja)
- [x] Faza 5 — Wisielec: 3 tryby (wyścig/kooperacja/zadający) w silniku + UI (klawiatura PL, SVG szubienicy); kooperacja zweryfikowana na produkcji (tajność hasła)
- [x] Faza 6 — Impostor: role/hasło tajne, 5 wariantów podpowiedzi (+ „nie wie, że jest impostorem"), głosowanie, zgadywanie po wylocie; zweryfikowane na produkcji (brak wycieku w publicState)
- [~] Faza 7 — Mafia: RDZEŃ gotowy i zweryfikowany na produkcji (mafia/mieszkańcy/detektyw/lekarz + auto-narrator, rozliczenie nocy, warunki wygranej, role tajne). Do zrobienia: role dodatkowe (§5.6) + tryb z prowadzącym
- [~] Faza 8 — polish: PWA (instalowalna, manifest+SW+ikona), Wake Lock (ekran nie gaśnie), konfetti+fanfara na wygranych — zweryfikowane na produkcji. Zostało: role dodatkowe Mafii (§5.6)

## Upgrade v2 — aktualny stan

- [x] Faza A — audyt (`AUDIT.md`): bezpieczeństwo OK, 0 [KRYTYCZNE], lista braków vs SPEC
- [x] Faza B — PWA: Serwist SW (NetworkOnly /api/ + Firebase), manifest kompletny (id/scope/shortcuts), ikony PNG (any+maskable+apple), prompt instalacji (beforeinstallprompt + iOS hint), ekran offline, useVisualViewport, useVibrate, user-scalable=no usunięte globalnie
- [x] Faza C — realtime: resync zegara po tła, powrót do pokoju (localStorage), migracja hosta na rozłączeniu (>30s via ping), actionId idempotencja, reset w transakcji, pasek połączenia, wykładniczy backoff
- [x] Faza D — wygląd: skeleton lobby, neon click-to-copy + ambient glow, slideIn/fadeIn animacje, timer-urgent pulsacja, nowe SFX (join/phaseChange/neonBuzz/defeat), ekran hosta TV (8rem kod, duże awatary), prefers-reduced-motion
- [x] Faza E — wydajność: dynamic imports gier (next/dynamic), manifests.ts (klient bez engines), selektywne private writes (JSON diff), debounce pingów 10s, /pokoj 339→314kB, / 110kB OK

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [w84kubus/doplay](https://github.com/w84kubus/doplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
