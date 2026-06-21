---
trigger: always_on
description: >
---


# Skill: Moodle-Lernsituationen-Creator (MCP-Version)

Erstellt einen vollständigen Moodle-Kursabschnitt auf Basis einer Lernsituation.
Nutzt ausschliesslich den **Moodle MCP Server** – kein Browser, keine Klicks.

---

## Grundprinzipien

### Phasenanzahl ist flexibel
Es gibt KEINE feste Anzahl von Phasen. Der Skill analysiert die vorliegende
Lernsituation und erstellt so viele Phasen wie darin beschrieben sind.
Typisch sind 3-6 Phasen – aber es koennen auch 2 oder 8 sein.

### Phasen-Design ist frei waehlbar
Phasen muessen NICHT dem FIAE-Schema folgen. Moegliche Phasenmodelle:
- Handlungsorientiert: Informieren / Planen / Durchfuehren / Kontrollieren / Reflektieren
- Projektbasiert: Analyse / Konzept / Implementierung / Test / Abnahme
- Problembasiert: Problem / Hypothese / Experiment / Auswertung
- Eigene Struktur aus der Lernsituation ableiten

### Inhalte aus der Lernsituation ableiten
Alle Texte, Aufgaben und Materialien werden AUS DER VORLIEGENDEN LERNSITUATION
abgeleitet. Nicht erfinden, nicht aus Beispielen kopieren.

---

## Verfuegbare MCP-Tools

| Tool | Verwendung |
|---|---|
| `moodle_get_sections` | Abschnitte eines Kurses lesen |
| `moodle_get_modules` | Aktivitaeten + cmids eines Abschnitts lesen |
| `moodle_update_section` | Abschnittsname + Handlungssituation-Card setzen |
| `moodle_create_label` | Phasen-Header anlegen (direkt auf Kursseite) |
| `moodle_create_page` | Textseite anlegen (nur lesen) |
| `moodle_create_url` | Externen Link anlegen |
| `moodle_create_assign` | Aufgabe anlegen (mit PDF-Button) |
| `moodle_update_label` | Label bearbeiten |
| `moodle_update_page` | Textseite bearbeiten |
| `moodle_update_assign` | Aufgabe bearbeiten |
| `moodle_update_url` | Link bearbeiten |

---

## Workflow

### Schritt 1: Lernsituation analysieren

Vor dem ersten API-Aufruf die Lernsituation lesen und notieren:
- Wie viele Phasen gibt es? Wie heissen sie?
- Welche Farbe bekommt jede Phase? (Frei waehlbar, aber konsistent)
- Welche Aktivitaeten gehoeren zu welcher Phase?
- Was muessen SuS NUR LESEN? Was muessen sie ABGEBEN?

### Schritt 2: Kursstruktur pruefen

```
moodle_get_sections(courseid=KURS_ID)
```

Freien Abschnitt waehlen.

### Schritt 3: Abschnitt benennen + Einstiegskarte setzen

```
moodle_update_section(courseid, sectionnum, name, summary)
```

### Schritt 4: Pro Phase

Fuer jede Phase der Lernsituation:
1. `moodle_create_label` – Phasen-Header
2. Je nach Inhalt: `moodle_create_page`, `moodle_create_url`, `moodle_create_assign`

---

## Aktivitaetstypen waehlen

| Situation | Tool |
|---|---|
| SuS liest nur (Infoblatt, Leitfaden, Anleitung, Codebeispiel) | `moodle_create_page` |
| SuS fuellt etwas aus / gibt etwas ab / reflektiert | `moodle_create_assign` |
| Externe Dokumentation, GitHub, MDN, Referenz | `moodle_create_url` |
| Phasen-Trenner (direkt auf Kursseite sichtbar) | `moodle_create_label` |

**GOLDENE REGEL:** Sobald SuS irgendetwas ausfullen, eintragen, ankreuzen
oder hochladen sollen -> IMMER `moodle_create_assign`, NIEMALS `moodle_create_page`!

---

## HTML-Vorlagen

### Einstiegskarte (fuer moodle_update_section summary)

Ersetze alle [PLATZHALTER] mit echten Inhalten aus der Lernsituation:

```html
<div style="background:linear-gradient(135deg,#1a237e,#283593);border-radius:12px;padding:0;margin-bottom:20px;overflow:hidden;box-shadow:0 4px 15px rgba(0,0,0,0.2);">
  <div style="background:rgba(255,255,255,0.1);padding:12px 20px;display:flex;align-items:center;gap:10px;">
    <span style="font-size:1.4em;">&#127919;</span>
    <div>
      <div style="color:rgba(255,255,255,0.7);font-size:0.75em;font-weight:600;letter-spacing:2px;text-transform:uppercase;">LERNSITUATION [NR] — [FIRMENNAME]</div>
      <div style="color:#fff;font-size:1.1em;font-weight:700;">Handlungssituation</div>
    </div>
  </div>
  <div style="background:#fff;margin:0 16px 16px;border-radius:8px;padding:20px;">
    <p style="color:#333;line-height:1.7;margin-bottom:16px;">[SITUATIONSBESCHREIBUNG AUS DER LERNSITUATION]</p>
    <div style="border-top:2px solid #e8eaf6;padding-top:14px;">
      <div style="color:#1a237e;font-size:0.75em;font-weight:700;letter-spacing:1.5px;text-transform:uppercase;margin-bottom:10px;">&#127919; HANDLUNGSERGEBNISSE</div>
      <ul style="margin:0;padding-left:20px;color:#444;line-height:2;">
        <li>[ERGEBNIS 1 AUS DER LERNSITUATION]</li>
        <li>[ERGEBNIS 2 AUS DER LERNSITUATION]</li>
      </ul>
    </div>
  </div>
</div>
```

### Phasen-Header (fuer moodle_create_label content)

```html
<div style="background:linear-gradient(135deg,[FARBE]dd,[FARBE]);border-radius:10px;padding:16px 20px;margin:10px 0;box-shadow:0 3px 10px rgba(0,0,0,0.15);">
  <div style="display:flex;align-items:center;gap:14px;">
    <span style="font-size:2em;">[ICON]</span>
    <div>
      <div style="color:rgba(255,255,255,0.8);font-size:0.7em;font-weight:700;letter-spacing:2px;text-transform:uppercase;">PHASE [NR]</div>
      <div style="color:#fff;font-size:1.25em;font-weight:700;">[PHASENNAME]</div>
      <div style="color:rgba(255,255,255,0.85);font-size:0.82em;margin-top:3px;">&#9203; ca. [ZEIT] Minuten &nbsp;•&nbsp; [SOZIALFORM]</div>
    </div>
  </div>
</div>
```

Farben und Icons frei waehlen, aber pro Kurs konsistent halten.
Empfehlungen (nicht verpflichtend):

| Typ | Farbe | Icon |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtuttas/MoodleMcp](https://github.com/jtuttas/MoodleMcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
