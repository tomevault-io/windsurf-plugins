---
trigger: always_on
description: **Vision**: Schaffung einer persönlichen, hochautomatisierten und intelligenten Buchhaltungs- und Dokumentenmanagement-Engine als private, überlegene Alternative zu Standardlösungen.
---


# LLKJJ ML Plugin - KI-Assistent Anweisungen (2025 Best Practices)

## Projektübersicht & Vision

**Vision**: Schaffung einer persönlichen, hochautomatisierten und intelligenten Buchhaltungs- und Dokumentenmanagement-Engine als private, überlegene Alternative zu Standardlösungen.

**Zweck**: Deutsches Elektrohandwerk-Buchhaltungs-Plugin für intelligente Rechnungsverarbeitung mit KI/ML-Pipeline (Teil des LLKJJ-Gesamtsystems)

**Kontext**: Dies ist **Workspace 2** (ML-Pipeline) des LLKJJ-Systems - dem deutschen buchhaltungsbutler.de-Ersatz für Elektrotechnik-Handwerksfirmen mit Rechtsform UG (haftungsbeschränkt), die zur doppelten Buchführung verpflichtet sind.

**Domäne**: Dokumenten-KI, OCR, NLP, SKR03-Kontierung, Machine Learning
**Sprache**: Deutsche Optimierung mit Elektrotechnik-Spezialisierung
**Primäre Mission**: PDF → strukturierte, SKR03-klassifizierte Rechnungsdaten

## Leitprinzipien & Strategische KI-Roadmap

### Leitprinzipien

1. **Autonomie**: Das System soll lernen, Kontexte verstehen und eigenständig korrekte Entscheidungen treffen
2. **Präzision**: >92% SKR03-Klassifizierungsgenauigkeit als oberste Priorität
3. **Sicherheit**: Höchste Priorität bei sensiblen Finanzdaten (Code-Analyse bis Datenspeicherung)
4. **Wartbarkeit**: Saubere, entkoppelte Architektur (KISS-Prinzip) für langfristige Pflege

### Strategische KI-Entwicklung (2-Phasen-Strategie)

**Phase 1 (Aktuell): "Gemini-First"-Pipeline**

- **Ansatz**: Google Gemini (`gemini-2.5d-flash` oder besser) als primäres "Gehirn"
- **Ziel**: Ein API-Aufruf ersetzt komplexe mehrstufige Prozesse
- **Nebeneffekt**: Sammlung hochwertiger Trainingsdaten für Phase 2

**Phase 2 (Zukunft): Lokale, autonome KI-Lösung**

- **Ansatz**: Nahtloser Übergang zu API-unabhängiger Lösung
- **Komponenten**: Selbst trainierte spaCy-Modelle (NER, TextCat) + lokales RAG-System (ChromaDB)
- **Datengrundlage**: Validierte Daten aus Phase 1

## Kern-Technologien & Stack

### Primäre Technologien

### **\*\***nutze IMMER bei Terminalbefehlen: poetry run ...**\*\***

- **Python 3.12+**: Moderne Type Hints, Pattern Matching, async/await
- **Poetry**: Dependency Management und CLI-Werkzeuge
- **Docling 2.44.0**: IBMs PDF-Verarbeitung mit TableFormer KI
- **spaCy 3.7+**: Deutsche NLP und Entitätserkennung
- **Gemini 2.5 flash**: KI-Verbesserung für Extraktionsqualität

### KI/ML Frameworks

- **Docling**: PDF → strukturierte Datenextraktion
- **Google Gemini**: LLM-basierte Content-Verbesserung
- **spaCy**: Named Entity Recognition für deutsche Texte
- **ChromaDB**: Vektordatenbank für intelligente Klassifizierung
- **Transformers**: Für zukünftige Modell-Integration

### Entwicklungstools

- **Pre-commit**: Code-Qualitäts-Automatisierung
- **Ruff**: Schnelles Python-Linting
- **Black**: Code-Formatierung
- **mypy**: Statische Typ-Überprüfung

## LLKJJ Gesamtarchitektur (4-Workspace-System)

Das LLKJJ-System basiert auf einer strikten Trennung zwischen einem zentralen Backend-Kern und spezialisierten, austauschbaren Plugins:

```
+---------------------------------------------------------------------------------+
|                                 LLKJJ-System                                    |
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  |                Workspace 1: Das Backend (Core-System)                     |  |
|  |---------------------------------------------------------------------------|  |
|  | [FastAPI] -> API Routers -> [Service Schicht] -> [Repository] -> PostgreSQL |  |
|  |      ^                (Business Logic)      (SQLAlchemy)      (Struktur)  |  |
|  |      |                                                                    |  |
|  |      +------------------(Definierte Schnittstellen)-----------------------+  |
|  +------------------------------------|----------------------------------------+  |
|                                       |                                         |
|      +--------------------------------+--------------------------------+        |
|      |                                |                                |        |
|      v                                v                                v        |
| +-------------------------+  +-------------------------+  +-------------------------+ |
| | Workspace 2:            |  | Workspace 3:            |  | Workspace 4:            | |
| | ML-Pipeline             |  | E-Invoice-System        |  | Export-System           | |
| | (`llkjj_ml_plugin`)     |  | (`llkjj_efaktura`)      |  | (`llkjj_export`)        | |
| |-------------------------|  |-------------------------|  |-------------------------| |
| | - OCR (Docling)         |  | - XRechnung / UBL XML   |  | - DATEV CSV Export      | |
| | - Gemini-First (aktiv)  |  | - PDF/A-3 Generierung   |  | - JSON / Standard CSV   | |
| | - RAG (ChromaDB)        |  | - KoSIT-Validierung     |  | - Universelle Schemas   | |
| +-------------------------+  +-------------------------+  +-------------------------+ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Czok12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
