---
trigger: always_on
description: Hopps ist eine cloud-basierte Open-Source Buchhaltungssoftware mit KI für gemeinnützige Organisationen (Vereine). Das Projekt wird von der Deutschen Stiftung für Engagement und Ehrenamt gefördert und vom Open Project e.V. Pfaffenhofen an der Ilm entwickelt.
---

# Hopps - Cloud-basierte Open-Source Buchhaltungssoftware mit KI

## Projektübersicht

Hopps ist eine cloud-basierte Open-Source Buchhaltungssoftware mit KI für gemeinnützige Organisationen (Vereine). Das Projekt wird von der Deutschen Stiftung für Engagement und Ehrenamt gefördert und vom Open Project e.V. Pfaffenhofen an der Ilm entwickelt.

**Architektur:** Monorepo mit Microservices-Architektur

## Projektstruktur

```
/backend          - Java Microservices (Quarkus)
/frontend         - Frontend-Anwendungen (SPA + Mobile)
  /spa            - React Web-Anwendung
  /mobile         - React Native Mobile App
  /api-client     - Shared TypeScript API Client
  /keycloak-theme - Custom Keycloak Theme
/infrastructure   - Docker Compose & Deployment
/charts           - Helm Charts für Kubernetes
/architecture     - Architekturdokumentation
/formatter        - Java Code-Formatter-Konfiguration
```

## Backend

### Tech Stack
- **Framework:** Quarkus 3.37.2
- **Sprache:** Java 21
- **Build:** Maven
- **Datenbank:** PostgreSQL 16 mit Flyway Migrations
- **ORM:** Hibernate mit Panache
- **Auth:** Keycloak (OAuth2/OIDC), Quarkus OIDC + Keycloak Admin Client
- **Storage:** AWS S3 (LocalStack lokal)
- **Realtime:** Quarkus WebSockets Next (Live-Benachrichtigungen bei Dokumentänderungen)
- **AI/ML:** LangChain4j mit OpenAI, Azure Document AI

### Microservices

#### app.hopps.org (Port 8101)
**Haupt-Service für Organisation und Member Management**

**Domain Models** (Feature-Slices unter `app/hopps/{feature}/domain`):
- `Organization` + `Address` - Hauptentität für Vereine (Name, Slug, Typ, Adresse, Website)
- `Member` - Benutzer mit Verknüpfung zu mehreren Organisationen
- `Bommel` - Hierarchische Baumstruktur für Organisationseinheiten (Abteilungen/Teams)
- `Category` - Kategorisierungssystem für Transaktionen
- `Transaction` - Finanztransaktionen mit Dokumentreferenzen
- `Document` + `TradeParty`, `DocumentTag` - hochgeladene Belege inkl. Analyse-/Extraktionsstatus
- `BankAccount`, `BankImport`, `BankTransaction`, `BankCsvSchema`, `BankCsvColumnMapping`, `BankTransactionMatch` - Bank-CSV-Import und Abgleich mit Transaktionen (`bankimport`-Slice)
- `Tag` (shared)

**API Endpoints** (class-level `@Path`):
- `/organization`, `/organizations` - Organisation CRUD, Registrierungsworkflow, Bommel-Zuordnung
- `/bommel` - Baumstruktur-Management mit rekursiven Operationen
- `/member` - Member-Verwaltung
- `/category` - Kategorie-Verwaltung
- `/transactions` - Transaktionsdatensätze
- `/documents` - Dokumenten-Upload und Abruf
- `/statistics` - Auswertungen/Kennzahlen
- `/bankaccounts`, `/imports`, `/bank-transactions`, `/bank-schemas` - Bank-Import-Feature
- `/health` - Health-Check
- `/ws/documents` - WebSocket für Live-Updates zu Dokumentanalysen

**Features:**
- Keycloak User Provisioning
- S3 Dokumentenspeicherung
- Bank-CSV-Import mit konfigurierbaren Schemata und Transaktions-Matching
- WebSocket-Live-Benachrichtigungen

#### app.hopps.az-document-ai (Port 8100, root-path `/api/az-document-ai`)
Dokumentenanalyse mit Azure Document Intelligence (OCR, Datenextraktion)

#### app.hopps.zugferd (Port 8103)
ZUGFeRD-Rechnungsverarbeitung (XML aus PDFs extrahieren)

### Datenbank Setup
- **Flyway Migrations:** `/src/main/resources/db/migration`
- **Testdaten:** `/src/main/resources/db/testdata` (dev mode)
- **Multi-Database:** Separate DBs für org, keycloak
- **Schema Version:** aktuell V1.0.19 (Nummerierung mit Lücke: V1.0.0–V1.0.2, dann V1.0.13–V1.0.19)

## Frontend

### SPA (Single Page Application)

**Tech Stack:**
- **Framework:** React 19.1
- **Build:** Vite 7
- **Sprache:** TypeScript 5.5
- **Routing:** React Router v7.5
- **State:** Zustand 5.0
- **API:** @tanstack/react-query 5.90
- **UI:** Radix UI + TailwindCSS 3.4
- **Forms:** React Hook Form 7.66 + Zod 4
- **Grid:** AG Grid 32.3
- **Charts:** Recharts 2.15
- **i18n:** i18next 23.16 + react-i18next 15
- **Auth:** Keycloak-js 24.0

**Features:**
- Drag-and-drop File Upload (react-dropzone)
- PDF Viewing (pdfjs-dist)
- Tree Visualisierung (react-d3-tree, @minoru/react-dnd-treeview)
- Emoji Support (emoji-mart)

**Struktur:**
```
src/
├── AppRoutes.tsx              - Zentrale Routen-Definition (von App.tsx eingebunden)
├── i18n.ts                    - i18next-Konfiguration
├── components/
│   ├── ui/                    - Wiederverwendbare UI-Komponenten (shadcn/ui-inspiriert)
│   ├── views/                 - Seiten-Level Komponenten
│   ├── BommelTreeView/        - Organisations-Baumvisualisierung
│   ├── Categories/            - Kategorie-Management
│   ├── Forms/                 - Formular-Komponenten
│   ├── BankAccounts/, Receipts/, Transactions/, InvoicesTable/, SettingsPage/, ...
│   └── AgGrid/, common/, sidebar-navigation/
├── services/                  - ApiService, OrganisationTreeService, Theme/Language/Connectivity/Emoji, auth/
├── store/                     - Zustand Stores
├── hooks/                     - Custom React Hooks
├── context/, providers/       - React Context & Provider
├── guards/                    - Route Guards (AuthGuard)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hopps-app/hopps](https://github.com/hopps-app/hopps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
