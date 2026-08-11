---
trigger: always_on
description: **Stand:** KI-Verhalten `r8.1E6E7AE3`, Commit `1d330a05`
---

# Nova.AiLab — Handreichung für Agenten

**Stand:** KI-Verhalten `r8.1E6E7AE3`, Commit `1d330a05`
(`feat/ai-goal-system-r8`), Definitionstabelle `0xD5F219A3F68088FF` ·
Referenzlauf [`20260810-1858-1d330a05`](reports/latest.md): Entscheidung
**6.490**, Endzustand **`0x0F892EFC042D6514`** · **die Zahlen in §5 stammen
aus genau diesem Lauf**
**Gilt zusätzlich:** `../CLAUDE.md` (Arbeitsvertrag), `../Project_Nova/AGENTS.md`,
[`README.md`](README.md) nebenan
**Erst lesen, wenn man neu hier ist:** [`UEBERGABE.md`](UEBERGABE.md) (der
vollständige Einstieg über beide Repositories) und [`START-HIER.md`](START-HIER.md)

Dieses Dokument beantwortet **eine** Frage: wie ein Agent das Labor für
automatische Evaluierung benutzt. **Was als nächstes gebaut wird, steht in
[`ROADMAP.md`](ROADMAP.md)** — eine Liste, eine Nummerierung.

---

## 0. Vier Sätze, bevor irgendetwas läuft

1. **Werkzeug, kein Beitrag.** Das Labor liegt seit dem Ausbau in einem
   **eigenen Repository** neben dem Spiel-Checkout und gerät in keinen
   `feat/`-Branch. `feat/`-Branches werden frisch von `upstream/main`
   abgezweigt. Im Spiel-Repo bleiben nur die In-Game-Debughilfen, auf
   `lab/ai-simulation`.
2. **Ein grüner Laborlauf ist Diagnose, kein Nachweis.** Was nicht im laufenden Spiel
   gesehen wurde, steht als ungesehen im PR-Text. Kein generierter Satz darf ein
   Laborergebnis so formulieren, als sei es gespielt worden.
3. **Verhalten und Baseline nie im selben PR.** Seit `e1a6a57` erzwingt das eine CI
   (`.github/workflows/baseline-guard.yml`), nicht mehr nur Disziplin — siehe §4.
4. **Ins Spiel wird nur über den Fork gepusht, nie auf dessen `main`.** Commit,
   Push und PR sind drei getrennte Freigaben; keine gilt für den nächsten Schritt
   mit. Das Labor ist ein eigenes Repo — dort ist `main` frei.

---

## 1. Der Regelkreis

Ein Agent, der KI-Verhalten ändert, braucht auf vier Fragen eine maschinenlesbare
Antwort. Alle vier kosten zusammen unter zehn Sekunden.

| Frage | Kommando | Antwort steht in |
|---|---|---|
| Hat sich das Verhalten überhaupt geändert? | `match --hash-every 100 --out <dir>` | `result.json` → `finalStateHash`, `decidedTick`; `hashchain.json` → **ab welchem Tick** es auseinanderläuft |
| Ist die Änderung deterministisch? | `match --repeat 2 --hash-every 100` | **Exit-Code** (siehe unten), nicht der Text |
| Ist sie besser oder nur anders? | `compare --out <dir>` | `resultset.json`, `report.html`, je Kandidat ein PR-Entwurf |
| Woran liegt es? | `match --view-every 25 --fog --out <dir>` | `player.html` + `view.ndjson`, dazu `dashboard.html` |
| Woran liegt es **bei dieser einen Einheit**? | derselbe Lauf | `player.html`: Einheit anklicken → Laufroute, Ereignisband, Detailfeld mit ihrem aktuellen Verhalten (Ziel, Angreifer, Ernte, Rückweg). Roh in `tracks.ndjson`, `events.ndjson`, `units.json` |
| **Was hat die KI vorgehabt** — und wie weit ist sie vom nächsten Vorhaben? | derselbe Lauf | `goals.ndjson`, im Player unter „what the AI wants": Goal, seit wann, welches davor, die Zahlen der Bedingung und der Abstand zur nächsten Schwelle |
| Und **was hätte sie getan, wenn** …? | `live --port 8787 --out <dir>` | die laufende Partie im Browser: anhalten, einzeln takten, einer Auswahl ein Goal aufzwingen. Schreibt `overrides.ndjson` und ein `result.json` mit `intervened: true` |
| Und **auf einem anderen Branch**? | `./lab-gui.sh` | die Steuerseite: Branch auswählen, messen, gegen einen früheren Lauf legen. Der Arbeitscheckout wird nie umgeschaltet — gemessen wird in einem `git worktree` |

Vorspann für alle Kommandos, falls `dotnet` nicht im PATH ist:

```bash
export DOTNET_ROOT="$PWD/.dotnet"; export PATH="$DOTNET_ROOT:$PATH"
```

### Exit-Codes — die einzige Stelle, an der der Rückgabewert selbst ein Befund ist

| Code | Bedeutung | Was ein Agent tun muss |
|---:|---|---|
| `0` | Lauf durch | weiterlesen in den Artefakten |
| `1` | Bedienfehler (unbekannter Modus, kaputte Spec) | Kommando korrigieren |
| `2` | **`NON-DETERMINISTIC`** bzw. **`SWEEP INVALID`** | **Sofort stoppen.** Zwei Läufe desselben Specs sind auseinandergelaufen. Das ist kein Flake, sondern geteilter Zustand zwischen parallelen Matches oder ein Determinismusbruch im Verhalten. Jede Zahl aus diesem Lauf ist wertlos, auch die grünen. |

Ein Agent, der `compare` oder `sweep` fährt, **prüft `$?` und bricht bei `2` ab**,
statt die Tabelle zu lesen. Jeder zwanzigste Sweep-Lauf wird zur Selbstkontrolle
doppelt gefahren — genau dafür.

### Zwei stille Fehlerarten, die kein Exit-Code meldet

- **`COMPARISON REFUSED`** auf stdout: Die Ergebnismenge wurde an einem anderen
  Commit oder gegen eine andere Definitionstabelle gemessen. Der Bericht zeigt dann
  **den Grund statt einer Tabelle**. Das ist das gewünschte Verhalten, kein Defekt —
  nach einem Merge-Fenster wird neu vermessen, nicht über die Grenze hinweg verglichen.
- **`orders refused — this row is not a measurement`**: Eine Zeile im Duell- oder
  Bewegungsbericht, deren Befehle abgelehnt wurden. Nicht als Ergebnis lesen.
- **`intervened: true` in `result.json`**: In diesem Lauf hat jemand über das
  Admin-Panel eingegriffen. Er sagt, was die KI *hätte* tun können, nie was sie
  tut — er wird nicht archiviert und nicht mit einem Messlauf verglichen. Der

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arn-c0de/Nova.AiLab](https://github.com/arn-c0de/Nova.AiLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
