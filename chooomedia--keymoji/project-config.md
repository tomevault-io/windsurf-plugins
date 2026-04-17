---
trigger: always_on
description: Keymoji ist eine moderne Svelte-Anwendung für sichere Emoji-Passwort-Generierung mit Magic-Link-Authentifizierung, Account-Management über n8n/Google Sheets, und AI Story Mode.
---

# Keymoji Cursor Rules - Senior Web Dev Pro Level
# Version: 0.8.0
# Last Updated: 2025-01-XX

## 🎯 Projekt-Übersicht
Keymoji ist eine moderne Svelte-Anwendung für sichere Emoji-Passwort-Generierung mit Magic-Link-Authentifizierung, Account-Management über n8n/Google Sheets, und AI Story Mode.

## 📋 Kern-Prinzipien

### 1. Code-Qualität & Best Practices
- **IMMER TypeScript verwenden** - Alle neuen Dateien müssen TypeScript sein
- **Svelte Runes verwenden** - `$state`, `$derived`, `$effect` statt klassische Stores wo möglich
- **DRY (Don't Repeat Yourself)** - Keine Duplikate, wiederverwendbare Komponenten/Utils
- **Single Source of Truth** - Jede Datenquelle hat genau einen Ort
- **Performance First** - Funktionen nur so oft ausführen wie nötig
- **Memory Leak Prevention** - Alle Timeouts/Intervals müssen gecleared werden

### 2. Architektur & Struktur

#### Stores (Svelte Runes bevorzugt)
```typescript
// ✅ GUT: Svelte 5 Runes
let count = $state(0);
let doubled = $derived(count * 2);
$effect(() => console.log(count));

// ⚠️ ALT: Nur wenn nötig für Kompatibilität
import { writable } from 'svelte/store';
```

#### API Calls
- **IMMER** `cachedFetch` aus `utils/apiCache.js` verwenden
- **KEINE** direkten `fetch()` Calls außer in `apiCache.js`
- **KEINE** doppelten API-Calls - Daten zwischen Komponenten weitergeben
- **IMMER** Error Handling mit try/catch
- **IMMER** Loading States

#### TypeScript Schemas
```typescript
// ✅ GUT: Schema-Definitionen
export interface Account {
  userId: string;
  email: string;
  tier: 'free' | 'pro';
  profile: UserProfile;
  metadata: AccountMetadata;
}

export interface UserProfile {
  name: string;
  avatar?: string;
}
```

### 3. Komponenten-Struktur

#### Svelte Komponenten
```svelte
<script lang="ts">
  // 1. Imports
  import { onMount } from 'svelte';
  
  // 2. TypeScript Interfaces
  interface Props {
    title: string;
    count?: number;
  }
  
  // 3. Props mit $props()
  let { title, count = 0 }: Props = $props();
  
  // 4. State mit $state()
  let isLoading = $state(false);
  
  // 5. Derived mit $derived()
  let displayCount = $derived(count * 2);
  
  // 6. Effects mit $effect()
  $effect(() => {
    console.log('Count changed:', displayCount);
  });
  
  // 7. Functions
  function handleClick() {
    // ...
  }
</script>
```

### 4. Tailwind CSS

#### Regeln
- **NUR** Tailwind Utility-Klassen verwenden
- **KEINE** `<style>` Tags in Komponenten (außer für Animationen)
- **Gemeinsame Patterns** in `tailwind.config.js` als Utilities definieren
- **PurgeCSS** aktiviert für Production Builds
- **Dark Mode** via `dark:` Klassen

#### Beispiel
```svelte
<!-- ✅ GUT -->
<div class="flex items-center justify-between p-4 bg-white dark:bg-gray-800 rounded-lg shadow-md">
  <h2 class="text-xl font-bold text-gray-900 dark:text-white">Title</h2>
</div>

<!-- ❌ SCHLECHT -->
<div class="custom-container">
  <h2 class="custom-title">Title</h2>
</div>
<style>
  .custom-container { /* ... */ }
</style>
```

### 5. A11y (Accessibility)

#### Regeln
- **IMMER** `aria-label` für Icon-Buttons
- **IMMER** Keyboard Navigation unterstützen
- **IMMER** Focus Management für Modals/Dialogs
- **IMMER** Skip Links für Screen Reader
- **IMMER** Alt-Text für Bilder
- **IMMER** Semantic HTML verwenden

#### Komponenten
- `FocusManager.svelte` für Focus Trapping
- `SkipLink.svelte` für Skip Navigation
- Reaktive ARIA-Labels für dynamische Inhalte

### 6. API & Backend

#### Magic-Link Authentication
- **IMMER** `secureLoginWithMagicLink()` verwenden
- **IMMER** `secureVerifyMagicLink()` verwenden
- **IMMER** Session Validation vor Account-Operationen
- **IMMER** Cross-Tab Communication für Magic Links

#### Account Management
- **IMMER** `accountStore.js` für Account-Operationen verwenden
- **IMMER** `syncAccountData()` nach Login/Update aufrufen
- **IMMER** `currentAccount` Store als Single Source of Truth
- **IMMER** Metadata Cleaning vor API-Calls

#### n8n Integration
- **IMMER** `WEBHOOKS` aus `config/api.js` verwenden
- **IMMER** `body: { body: { ... } }` Wrapper für n8n
- **IMMER** JSON-Strings für `profile` und `metadata` in Google Sheets

### 7. Storage & State

#### localStorage
- **IMMER** `storageHelpers` aus `config/storage.js` verwenden
- **IMMER** `STORAGE_KEYS` Konstanten verwenden
- **IMMER** Migration bei Strukturänderungen
- **IMMER** Error Handling für QuotaExceededError

#### Session Management
- **IMMER** `USER_PREFERENCES` für Account-Daten
- **IMMER** Session Expiry prüfen
- **IMMER** Cross-Tab Synchronization

### 8. Performance

#### Optimierungen
- **IMMER** API-Caching verwenden (`cachedFetch`)
- **IMMER** Debouncing für User Input
- **IMMER** Lazy Loading für große Komponenten
- **IMMER** Code Splitting für Routes
- **IMMER** Memoization für teure Berechnungen

#### Reaktivität
- **IMMER** `$derived` für abgeleitete Werte
- **IMMER** `$effect` für Side Effects
- **KEINE** reaktiven Statements mit Side Effects
- **KEINE** unnötigen Re-Renders

### 9. Error Handling

#### Regeln
- **IMMER** try/catch für async Operations
- **IMMER** User-freundliche Error Messages
- **IMMER** Error Logging für Debugging
- **IMMER** Fallback Values


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chooomedia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
