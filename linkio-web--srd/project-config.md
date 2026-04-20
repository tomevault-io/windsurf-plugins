---
trigger: always_on
description: Site vitrine professionnel pour **SRD Partners Sàrl** (fiduciaire, Genève).
---

# claude.md

## Projet
Site vitrine professionnel pour **SRD Partners Sàrl** (fiduciaire, Genève).

**Stack :** Next.js 15 App Router · TypeScript · Tailwind CSS 3
**Architecture :** propre, scalable, maintenable — zéro dépendance inutile.

---

## PRINCIPES OBLIGATOIRES

### 1. DRY (Don't Repeat Yourself)
- Aucune duplication de logique, de style ou de texte.
- Pattern répété → composant. Valeur répétée → token ou constante. Style répété → classe utilitaire.

### 2. Séparation des responsabilités
- UI séparée de la logique.
- Textes visibles → fichiers de traduction uniquement (`src/messages/*.json`).
- Données structurelles non-traduites (icônes, IDs, liens, coordonnées) → `src/lib/siteData.ts`.
- Tokens de design → `tailwind.config.ts` (couleurs, ombres, polices) + `src/app/globals.css` (classes utilitaires).

### 3. Clean Code
- Composants petits et spécialisés. Nommage explicite.
- Pas de code mort. Pas de `console.log`. Pas de style inline sauf si absolument nécessaire.

### 4. Scalabilité
- Ajout d'une nouvelle langue : créer `src/messages/<code>.json` + ajouter le code dans `src/lib/i18n.ts`.
- Ajout d'un service : ajouter l'entrée dans `siteData.ts` + les clés dans chaque fichier de traduction.
- Ajout d'une section : créer la page ou le bloc, réutiliser `<Section>`.

---

## SYSTÈME DE DESIGN

> Documentation complète : `design.md` (racine du projet)

### Tokens dans `tailwind.config.ts`
Les couleurs, ombres et polices sont définies dans le thème Tailwind — ne pas les hardcoder.

**Couleurs extraites du logo SRD Partners (v2) :**

| Token Tailwind      | Hex        | Usage                                    |
|---------------------|------------|------------------------------------------|
| `bg-cream`          | `#F8F9FC`  | Fond section principale (cool near-white)|
| `bg-stone`          | `#D2C9D6`  | Fond section alternée (cool light gray)  |
| `bg-navy`           | `#301B4A`  | Fond sombre — hero, footer               |
| `text-gold` / `bg-gold` | `#97144F` | Accent crimson (arrow logo) — CTA, overlines |
| `text-primary-500`  | `#301B4A`  | Violet logo SRD — titres, btn-outline    |
| `text-ink`          | `#1A0F2A`  | Texte principal dark                     |
| `text-muted`        | `#5A6080`  | Texte secondaire cool                    |
| `shadow-premium`    | multicouche| Cartes premium                           |
| `font-display`      | **Playfair Display** | Titres H1/H2, accent italique  |
| `font-body`         | **Manrope**          | Corps de texte, UI, forms      |

### Classes utilitaires dans `globals.css`
Définies dans `@layer components` / `@layer utilities` — toujours préférer ces classes :

| Classe             | Usage                              |
|--------------------|------------------------------------|
| `.container-main`  | Conteneur centré responsive        |
| `.section-label`   | Overline de section (uppercase)    |
| `.section-title`   | Titre H2 (Cormorant font-light, tracking -0.01em) |
| `.section-subtitle`| Sous-titre de section              |
| `.btn-primary`     | Bouton or (CTA principal)          |
| `.btn-outline`     | Bouton outline or                  |
| `.btn-outline-dark`| Bouton outline blanc (sur fond sombre) |
| `.card-base`       | Carte blanche avec hover           |
| `.animate-fade-up` | Animation d'entrée fadeUp          |
| `.hero-gradient`   | Dégradé navy/or du hero            |
| `.hero-grain`      | Texture grain sur le hero          |
| `.gold-line`       | Trait décoratif doré               |

**Interdit :** hardcoder couleurs, tailles, radius, ombres hors de ces systèmes.

---

## MULTILINGUE

### Architecture i18n custom (sans bibliothèque externe)

**Fichier central :** `src/lib/i18n.ts`
```ts
export type Locale = 'fr' | 'en' | 'pt'
export const locales: Locale[] = ['fr', 'en', 'pt']
export const defaultLocale: Locale = 'fr'

export function getMessages(locale: Locale): Messages { … }
export function isValidLocale(value: string): value is Locale { … }
```

**Usage dans les pages/layouts :**
```ts
const t = getMessages(locale)
// t.nav.home, t.hero.title, etc.
```

**Middleware :** `src/middleware.ts` — redirige `/` → `/<locale>` via cookie ou défaut `fr`.

### Fichiers de traduction
```
src/messages/
  fr.json   ← référence (toutes les clés)
  en.json   ← même structure
  pt.json   ← même structure
```
- Toutes les clés doivent être identiques dans les 3 fichiers.
- **Aucun texte visible en dur dans les composants.**

### Routing
```
/fr           → Page d'accueil (française)
/fr/services
/fr/a-propos
/fr/contact
/en/…
/pt/…
```

---

## COMPOSANTS TYPOGRAPHY

### Footer `slantFill`
Le `Footer` accepte une prop `slantFill?: string` (défaut `BG.stone`).
Doit être passée explicitement quand le `ContactBlock` précédent a `bg="cream"` :
```tsx
<Footer ... slantFill={BG.cream} />  {/* si ContactBlock bg="cream" */}
```

### `PremiumHeading` + `Accent` (`src/components/PremiumHeading.tsx`)
Système de titres premium : serif léger + fragment italique doré.

```tsx
import { PremiumHeading, Accent } from '@/components/PremiumHeading'

// Props : as (h1|h2|h3), size (hero|page|section), color (light|dark)
<PremiumHeading as="h1" size="page" color="light">
  {t.section.titleMain} <Accent>{t.section.titleAccent}</Accent>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linkio-web) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
