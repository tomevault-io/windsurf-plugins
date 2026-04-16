---
trigger: always_on
description: Dies ist eine NFC Web Prototype Anwendung gebaut mit React, TypeScript und Tailwind CSS.
---

# GitHub Copilot Instructions - NFC Web Prototype

## Projekt-Übersicht

Dies ist eine NFC Web Prototype Anwendung gebaut mit React, TypeScript und Tailwind CSS.
Die App wird auf GitHub Pages gehostet und automatisch via GitHub Actions deployed.

Auf einem mobilen Gerät mit NFC-Unterstützung können Nutzer NFC-Tags scannen und die ausgelesenen Daten werden in der App angezeigt.

## Tech Stack

- **Framework**: React 18+ mit TypeScript
- **Routing**: React Router v6
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Deployment**: GitHub Pages via GitHub Actions
- **Package Manager**: npm

## Code-Stil und Konventionen

### TypeScript

- Verwende **strikte TypeScript-Konfiguration** mit aktivierten `strict`, `noUnusedLocals`, und `noUnusedParameters`
- Definiere alle Props mit TypeScript Interfaces oder Types
- Vermeide `any` - nutze stattdessen `unknown` oder spezifische Typen
- Nutze Type Guards für Runtime Type Checking
- Bevorzuge `interface` für React Props und öffentliche APIs
- Bevorzuge `type` für Unions, Intersections und berechnete Typen

### React Komponenten

- Nutze **funktionale Komponenten** mit Hooks
- Bevorzuge **Named Exports** für Komponenten
- Verwende **PascalCase** für Komponentennamen
- Verwende **camelCase** für Props und Handler-Funktionen
- Event Handler sollten mit `handle` prefixed werden (z.B. `handleClick`, `handleSubmit`)
- Props für Event Handler sollten mit `on` prefixed werden (z.B. `onClick`, `onSubmit`)

### Komponenten-Struktur

```typescript
// Gute Struktur
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

export function Button({
  label,
  onClick,
  variant = "primary",
  disabled = false,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`px-4 py-2 rounded ${
        variant === "primary"
          ? "bg-blue-600 text-white"
          : "bg-gray-200 text-gray-800"
      }`}
    >
      {label}
    </button>
  );
}
```

### Ordnerstruktur

```
src/
├── components/        # Wiederverwendbare UI-Komponenten
│   ├── ui/           # Basis UI-Komponenten (Button, Input, Card, etc.)
│   ├── layout/       # Layout-Komponenten (Header, Navigation, Footer)
│   └── features/     # Feature-spezifische Komponenten
├── hooks/            # Custom React Hooks
├── types/            # TypeScript Type Definitionen
├── utils/            # Utility-Funktionen
├── services/         # API-Calls und externe Services (z.B. NFC API)
├── pages/            # Seiten-Komponenten (Home, Impressum, etc.)
└── assets/           # Statische Assets (Bilder, Icons)
```

## Navigation und Routing

### Navigationsstruktur

- Implementiere ein **Hamburger-Menü** für mobile Navigation
- Das Menü soll auf Desktop als horizontale Navigation angezeigt werden
- Nutze React Router v6 für Client-Side Routing
- Verwende `BrowserRouter` mit `basename` für GitHub Pages

### Seitenstruktur

Die App muss mindestens folgende Seiten enthalten:

- **Home** (`/`) - Startseite mit NFC-Scanner
- **Impressum** (`/impressum`) - Impressums-Seite mit rechtlichen Informationen

### Routing Best Practices

```typescript
// src/App.tsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter basename="/nfc-web-prototype">
      <Layout>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/impressum" element={<ImpressumPage />} />
        </Routes>
      </Layout>
    </BrowserRouter>
  );
}
```

### Navigation Component

- Mobile: Hamburger-Icon, das ein Slide-in Menü öffnet
- Desktop: Horizontale Navigation in der Header-Leiste
- Active Route sollte visuell hervorgehoben werden
- Smooth Transitions für Menü-Öffnen/-Schließen
- Accessibility: Keyboard-Navigation und ARIA-Labels

## Tailwind CSS Best Practices

### Styling Richtlinien

- Nutze Tailwind Utility Classes direkt in JSX
- Vermeide inline Styles - nutze stattdessen Tailwind Classes
- Für komplexe, wiederverwendbare Styles nutze `@apply` in CSS-Dateien
- Nutze Tailwind's responsive Prefixes: `sm:`, `md:`, `lg:`, `xl:`, `2xl:`
- Nutze Tailwind's state variants: `hover:`, `focus:`, `active:`, `disabled:`

### Beispiel für responsive Design

```typescript
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <Card className="p-4 hover:shadow-lg transition-shadow" />
</div>
```

### Dark Mode Support

- Bereite Komponenten für Dark Mode vor mit `dark:` Prefix

```typescript
<div className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white">
```

## NFC Web API Integration

### Web NFC API Nutzung

- Prüfe Browser-Unterstützung: `if ('NDEFReader' in window)`
- Nutze TypeScript Types für NFC: `NDEFReader`, `NDEFMessage`, `NDEFRecord`
- Implementiere Error Handling für fehlende NFC-Unterstützung
- Nutze try-catch für alle NFC-Operationen

### Beispiel NFC Service

```typescript
// src/services/nfc.service.ts
export class NFCService {
  private reader: NDEFReader | null = null;

  async isSupported(): Promise<boolean> {
    return "NDEFReader" in window;
  }

  async scan(): Promise<NFCData> {
    if (!(await this.isSupported())) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oliverscheer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
