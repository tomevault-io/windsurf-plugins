---
trigger: always_on
description: Ein LLM-gepflegtes Branchen-Wiki nach dem [LLM Wiki Pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Das Wiki wird ausschließlich von LLM-Agenten geschrieben und gepflegt. Der Mensch kuratiert Quellen, stellt Fragen und gibt die Richtung vor.
---

# wiki-hermes

Ein LLM-gepflegtes Branchen-Wiki nach dem [LLM Wiki Pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Das Wiki wird ausschließlich von LLM-Agenten geschrieben und gepflegt. Der Mensch kuratiert Quellen, stellt Fragen und gibt die Richtung vor.

---

## KRITISCHE REGELN

> ⛔ Diese Regeln gelten IMMER. Kein Workflow ist abgeschlossen ohne sie zu befolgen.

### Pflicht-Updates nach JEDER Wiki-Operation

Jede Operation die Wiki-Seiten erstellt oder aktualisiert MUSS mit diesen drei Schritten ENDEN:

1. **`wiki/index.md` aktualisieren** — Alle neuen/geänderten Seiten eintragen. Lies den Index, prüfe ob alles drin steht.
2. **`wiki/overview.md` aktualisieren** — Branchenübersicht anpassen. Neue Themen, Trends, Player einarbeiten.
3. **`wiki/log.md` ergänzen** — Eintrag am Ende anfügen mit Datum, Operation und betroffenen Seiten.

Ein Ingest, Query oder Draft ohne diese drei Updates ist **UNVOLLSTÄNDIG**. Beende niemals einen Workflow ohne diese Dateien geprüft und aktualisiert zu haben.

### Verlinkung — kategorieübergreifend, bidirektional und pflichtmäßig

Die Verlinkungen zwischen Wiki-Seiten sind das Herzstück des Wikis. Sie machen die Wissensstruktur in Obsidian Graph View sichtbar. Ohne Verlinkungen ist das Wiki nur eine Sammlung loser Dateien.

**WICHTIG:** Nur echte Markdown-Links im Seiteninhalt erzeugen Kanten im Graphen. Das `sources`-Feld im YAML-Frontmatter ist für den Graphen UNSICHTBAR.

Jede Wiki-Seite MUSS am Ende einen Abschnitt haben:

```markdown
## Verwandte Seiten
- [Seitentitel](../kategorie/dateiname.md)
- [Seitentitel](../kategorie/dateiname.md)
```

Regeln:
- **Mindestens 2 Verlinkungen** pro Seite
- **Kategorieübergreifend** — Links MÜSSEN über Ordnergrenzen hinweg gehen (siehe Tabelle unten)
- **Bidirektional** — wenn Seite A auf Seite B verlinkt, MUSS Seite B auch auf Seite A verlinken
- Bei neuen Seiten: bestehende thematisch passende Seiten finden UND dort einen Rückverweis setzen
- Relative Markdown-Links verwenden: `[Titel](../kategorie/dateiname.md)`
- Bei Widersprüchen zwischen Seiten: `> ⚠️ Widerspruch: ...` markieren

### Pflicht-Verlinkungen nach Seitentyp

| Seitentyp | MUSS verlinken auf |
|---|---|
| `sources/` | Alle Topics, Trends, Players und Regulation-Seiten die aus dieser Quelle entstanden oder aktualisiert wurden |
| `topics/` | Relevante Sources, verwandte Trends, beteiligte Players, betroffene Regulation |
| `trends/` | Relevante Sources, verwandte Topics, beteiligte Players, betroffene Regulation |
| `players/` | Relevante Sources, zugehörige Topics, relevante Trends |
| `regulation/` | Relevante Sources, betroffene Topics, beteiligte Players |
| `market/` | Relevante Sources, verwandte Topics, relevante Trends, relevante Players |
| `synthesis/` | Alle Wiki-Seiten auf denen die Analyse basiert |

Beispiel einer gut verlinkten Topic-Seite:

```markdown
## Verwandte Seiten

### Quellen
- [McKinsey Report 2026](../sources/2026-04-06-mckinsey-report.md)
- [Heise-Artikel KI-Agenten](../sources/2026-04-05-heise-ki-agenten.md)

### Trends
- [KI-Agenten](../trends/ki-agenten.md)
- [Prozessautomatisierung](../trends/prozessautomatisierung.md)

### Akteure
- [Microsoft](../players/microsoft.md)
- [Fraunhofer IAO](../players/fraunhofer-iao.md)

### Regulierung
- [EU AI Act](../regulation/eu-ai-act.md)
```

---

## Architektur

```
raw/          → Unveränderliche Quelldokumente (Input)
wiki/         → LLM-generierte Wiki-Seiten (Verarbeitung)
content/      → Abgeleiteter Content für Veröffentlichung (Output)
```

### raw/ — Quellen

Immutable. Der Agent liest, aber verändert niemals Dateien in `raw/`.

| Unterordner | Inhalt |
|---|---|
| `raw/articles/` | Web-Artikel als Markdown |
| `raw/pdfs/` | PDF-Dokumente |
| `raw/notes/` | Eigene Notizen, Gesprächsnotizen, Ideen |

### wiki/ — Wissensbasis

Der Agent besitzt diesen Layer vollständig. Er erstellt Seiten, aktualisiert sie, pflegt Querverweise und hält alles konsistent.

| Unterordner | Inhalt |
|---|---|
| `wiki/topics/` | Fachthemen — Technologien, Methoden, Prozesse |
| `wiki/trends/` | Aufkommende Entwicklungen, schwache Signale, Prognosen |
| `wiki/regulation/` | Gesetze, Normen, Standards, Compliance |
| `wiki/market/` | Marktdaten, Segmente, Wachstumsbereiche, Preismodelle |
| `wiki/players/` | Firmen, Verbände, Personen, Forschungseinrichtungen, Partner |
| `wiki/sources/` | Eine Summary-Seite pro verarbeiteter Quelle |
| `wiki/synthesis/` | Eigene Analysen, Querverbindungen, Schlussfolgerungen |

Sonderdateien:

| Datei | Zweck |
|---|---|
| `wiki/index.md` | Katalog aller Wiki-Seiten mit Kategorie, Link und Kurzbeschreibung |
| `wiki/log.md` | Chronologisches Protokoll aller Operationen (append-only) |
| `wiki/overview.md` | High-Level-Branchenübersicht — das "Big Picture" |

### content/ — Output

Abgeleiteter Content für Veröffentlichung. Wird aus Wiki-Seiten generiert.

| Unterordner | Inhalt |
|---|---|
| `content/drafts/` | LLM-generierte Entwürfe (vor Review) |
| `content/published/` | Freigegebene, finale Inhalte |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PLATZDORSCH/wiki-hermes](https://github.com/PLATZDORSCH/wiki-hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
