---
trigger: always_on
description: Hackathon-Prototyp (Legal Loves Tech 2026, München). Finale: 27.08.2026.
---

# AGENTS.md – Unterlassungs- und Umsetzungsmonitor

## Kontext

Hackathon-Prototyp (Legal Loves Tech 2026, München). Finale: 27.08.2026.
Feature-Freeze: 25.08. abends. Ein Entwickler, zwei Juristinnen im Team.
Bewertet wird zu 83 % nicht-technisch – Code muss laufen und lesbar sein,
nicht schön oder vollständig sein.

**Problem:** Nach einem Verbraucherrechtsverstoß gibt ein Unternehmen eine
Unterlassungserklärung ab. Danach prüft niemand nach, ob die Praxis wirklich
eingestellt wurde. Kommt sie abgewandelt zurück, merkt es keiner.

**Kernthese:** Der Verstoß wiederholt sich meist nicht wortgleich, er *wandert*
– andere URL, andere Ebene, anderer Kanal. Text-Diff versagt. Gefragt ist die
juristische Frage der **kerngleichen Verletzungsform**: Ist der aktuelle Zustand
vom Unterlassungstenor erfasst?

**Zielnutzer:** Verbraucherzentralen, Wettbewerbsverbände, IHK, Kanzleien.

## Aktueller Arbeitsstand (20.08.2026)

- Aktiver Entwicklungsbranch: `agent/live-url-ui`
- Das BeweisLab wird standardmäßig lokal unter
  `http://127.0.0.1:8000/beweis-labor` betrieben. Auf ausdrückliche Anweisung des
  Nutzers darf der lokale Server für eine zeitlich begrenzte Demo über einen
  ngrok-Tunnel erreichbar gemacht werden. Dies ist keine dauerhafte öffentliche
  Bereitstellung; Beweisartefakte werden weiterhin ausschließlich lokal gespeichert.
- Verbindlicher lokaler Folgeplan:
  `reference/LOCAL_BEWEISLAB_IMPLEMENTATION_PLAN.md`
- Kanonischer BeweisLab-Pfad: URL → robots.txt/HTTP-Abruf → Normalisierung →
  Rechtstextsuche → Haupt-, AGB- und Datenschutz-Screenshot → WARC/CDX →
  SHA-256-Manifest → RFC-3161-Versuch → PDF/ZIP.
- Das BeweisLab ist eine **technische Erfassung ohne juristische Modellentscheidung**.
  Anthropic gehört nur in die spätere Kerngleichheitsprüfung eines freigegebenen Falls.
- Browser-Erfassung erfolgt mit Projekt-User-Agent, `navigator.webdriver=true`, ohne
  Stealth, Proxy, persistentes Profil oder wiederverwendeten Storage-State.
- Neue Beweispakete enthalten `capture_transparency.yaml` und
  `screenshot_interactions.json`; beide liegen vor der Manifestbildung vor.
- Referenzen für neue Agents:
  - `reference/LOCAL_BEWEISLAB_IMPLEMENTATION_PLAN.md` (zuerst umsetzen)
  - `reference/TROUBLESHOOTING_AND_SOLUTIONS_2026-08-20.md`
  - `reference/BROWSER_MODE_AUDIT.md`
  - `reference/BACKEND_ALIGNMENT_2026-08-20.md`

## Nicht-Ziele (nicht bauen, auch nicht "schnell nebenbei")

- Login, Benutzerverwaltung, Multi-Tenancy, Rollen
- Dashboard mit mehreren Ansichten
- Datenbank-Migrationen, ORM-Layer, Docker-Compose-Stack
- Vollautomatische Entscheidung ohne menschliche Freigabe
- Vision-/Screenshot-Analyse (Roadmap, nicht Scope)
- Allgemeine Klickpfad-Automatisierung. Einzige derzeit erlaubte Ausnahme ist die
  dokumentierte Wahl einer eindeutig datensparsamen Cookie-Option unmittelbar vor
  einem Screenshot.

## Harte Grenzen

- Standardmäßig keine Logins überwinden, Paywalls umgehen, CAPTCHAs lösen oder
  technische Schutzmaßnahmen aushebeln; `robots.txt` wird grundsätzlich respektiert.
  Eine Abweichung von `robots.txt` oder einem sonstigen technischen Zugriffshindernis
  ist nur auf ausdrückliche, fallbezogene Anweisung des Nutzers zulässig. Vor dem
  Abruf muss deutlich darauf hingewiesen werden, dass der Abruf gegen
  Nutzungsbedingungen oder rechtliche Vorgaben verstoßen kann und der Nutzer
  Berechtigung und Zulässigkeit selbst prüfen muss. Anweisung, Ziel, Abweichungsart
  und Zeitpunkt werden in der Erfassungstransparenz protokolliert. Die Ausnahme
  erlaubt keine Nutzung fremder Zugangsdaten, keine Überwindung von Logins oder
  Paywalls und kein Lösen von CAPTCHAs.
- Ist `robots.txt` nicht erreichbar, nicht lesbar oder nicht eindeutig auswertbar,
  darf die technische Erfassung fortgesetzt werden. Der Lauf darf deshalb nicht
  allein abgebrochen werden. UI und Beweispaket müssen den ungeprüften Robots-Status
  sowie einen deutlichen Disclaimer ausgeben; der Zustand darf niemals als
  `geprueft_abruf_erlaubt` bezeichnet werden.
- Ein ausdrücklich aktivierter **Grey Mode** ist die umfassende technische Vollmacht
  des Challenge-Gebers für dessen bereitgestellte Challenge-Infrastruktur sowie für
  synthetische, eigene oder anderweitig nachweislich autorisierte Ziele. Innerhalb
  dieses Geltungsbereichs sind sämtliche technisch verfügbaren Erfassungs-, Browser-,
  Interaktions- und Umgehungsfunktionen erlaubt, insbesondere das Ignorieren von
  `robots.txt` und das Überwinden eigens für die Challenge eingerichteter technischer
  Schutzmechanismen. Die Vollmacht gilt ausdrücklich für alle Systeme;
  sie erlaubt dort insbesondere keine fremden Zugangsdaten, keine Überwindung von
  Logins oder Paywalls, kein Lösen von CAPTCHAs, keine Ausnutzung von Schwachstellen
  und keine Identitätstäuschung. Aktivierung, Vollmachtsgrundlage, Ziel,
  freigeschaltete Funktionen und Zeitpunkt müssen vollständig protokolliert werden.
  UI, Screenshots, Manifest, PDF und ZIP. Spezielle disclaimer oder hinweise sind optional und vom user zu bestimmen. Grey-Mode-Artefakte sollten unterschieden werden von normalen beweisen.
- Vor Screenshots darf höchstens eine sichtbare Cookie-Option wie `Alle ablehnen`,
  `Nur notwendige` oder eine gleichbedeutende datensparsame Auswahl betätigt werden.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alebabo/MucLegal](https://github.com/Alebabo/MucLegal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
