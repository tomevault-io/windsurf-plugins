---
trigger: always_on
description: Builder de portfolios en ligne. Formulaire guidé → choix template → site live sur `pseudo.vizly.fr`.
---

# Vizly — Portfolio Builder SaaS

Builder de portfolios en ligne. Formulaire guidé → choix template → site live sur `pseudo.vizly.fr`.

## Stack

- **Next.js 15** (App Router) — frontend + backend
- **Supabase** — PostgreSQL, Auth (email + Google OAuth), Storage (images)
- **Stripe** — subscriptions (Starter 4.99€/Pro 9.99€) + one-shot templates premium (2.99€)
- **Resend** — emails transactionnels (noreply@vizly.fr)
- **Vercel** — hosting, wildcard `*.vizly.fr`
- **Tailwind CSS + shadcn/ui** — styling (toujours personnaliser shadcn)
- **Framer Motion** — animations
- **Playwright** — tests E2E

## MCPs

- **Supabase** : tables, migrations, RLS, storage, auth, SQL
- **Vercel** : deploy, domaines, env vars

## Conventions

- TypeScript strict : pas de `any`, `@ts-ignore`, `as unknown as`, `!`
- Server Components par défaut, `"use client"` uniquement si hooks/events/browser APIs
- Server Actions pour les mutations, Route Handlers pour les API publiques
- Zod pour toute validation (formulaires, API, webhooks)
- RLS sur 100% des tables Supabase
- Commits : `type(scope): description` (feat, fix, refactor, style, test)

# DESIGN-SYSTEM.md — Vizly

> Ce fichier est la source de vérité pour tout le design de Vizly.
> Claude Code DOIT lire ce fichier avant de toucher à n'importe quel composant UI.
> En cas de doute, la règle est : MOINS > PLUS.

---

## PHILOSOPHIE

Vizly doit ressembler à un SaaS conçu par une vraie équipe produit, pas à un side project vibe-coded.
Références visuelles : Apollo, Maze, Neon, Remote, Sprig.
ADN commun de ces apps : fond blanc, très peu de couleurs, beaucoup de whitespace,
inputs simples sans décoration, hiérarchie typographique claire, zéro effet superflu.

Mantra : **"Si tu hésites à ajouter quelque chose, ne l'ajoute pas."**

---

## COULEURS

```
--color-bg:              #FFFFFF        /* Fond principal, toujours blanc pur */
--color-bg-subtle:       #F9FAFB        /* Fond secondaire (sections alternées, sidebar) */
--color-bg-muted:        #F3F4F6        /* Fond inputs désactivés, hover léger */
--color-border:          #E5E7EB        /* Bordures par défaut — gris très léger */
--color-border-focus:    #D1D5DB        /* Bordures en focus — un cran plus foncé */

--color-text-primary:    #111827        /* Titres et texte principal — presque noir */
--color-text-secondary:  #6B7280        /* Texte secondaire, labels, descriptions */
--color-text-tertiary:   #9CA3AF        /* Placeholders, texte désactivé */

--color-accent:          #E8553D        /* Accent Vizly (le corail/rouge actuel) — UNIQUE couleur vive */
--color-accent-hover:    #D4442E        /* Hover sur accent */
--color-accent-light:    #FEF2F0        /* Background très léger pour badges/tags accent */

--color-success:         #059669        /* Vert validation — utilisé avec parcimonie */
--color-error:           #DC2626        /* Rouge erreur uniquement */
--color-warning:         #D97706        /* Orange warning uniquement */
```

### Règles couleurs
- L'accent corail (#E8553D) est utilisé UNIQUEMENT pour : le bouton CTA principal, le stepper actif, les liens d'action importants
- Maximum 1 élément accent visible à l'écran en même temps (hors stepper)
- Pas de gradients. Jamais. Nulle part.
- Pas de couleurs de fond colorées sur les sections (pas de bg-blue-50, bg-purple-50, etc.)
- Les icônes sont en --color-text-secondary (#6B7280), jamais en couleur

---

## TYPOGRAPHIE

```
--font-family:           'DM Sans', system-ui, sans-serif
```

DM Sans pour tout. Pas de deuxième police. Pas de serif. Pas de monospace sauf code.

### Échelle typographique (stricte, pas d'autres tailles)

| Token             | Taille  | Poids | Line-height | Usage                           |
|--------------------|---------|-------|-------------|---------------------------------|
| --text-page-title  | 24px    | 600   | 32px        | Titre de page ("Mes projets")   |
| --text-section     | 18px    | 600   | 28px        | Titre de section dans un form   |
| --text-subsection  | 14px    | 600   | 20px        | Sous-titre, label de catégorie  |
| --text-body        | 14px    | 400   | 20px        | Corps de texte, labels de champ |
| --text-small       | 13px    | 400   | 18px        | Descriptions, helper text       |
| --text-caption     | 12px    | 500   | 16px        | Compteurs (4/30), badges, meta  |

### Règles typo
- Pas de texte en bold (700) sauf dans les boutons
- Pas de texte en uppercase sauf les badges de statut (PRO, BROUILLON)
- Pas d'italique
- Pas de text-decoration sauf :hover sur les liens
- Les labels de champ sont en --text-body (14px/400), couleur --color-text-secondary
- Les titres de section n'ont PAS d'icône devant eux (retire les icônes décoratives)

---

## SPACING

Grille de 4px. Tous les espacements sont des multiples de 4.

```
--space-xs:    4px
--space-sm:    8px
--space-md:    12px
--space-base:  16px
--space-lg:    24px
--space-xl:    32px
--space-2xl:   48px
--space-3xl:   64px
```

### Règles spacing
- Padding intérieur d'une page : 32px horizontal, 32px vertical
- Gap entre sections de formulaire : 32px (pas de cards — juste du whitespace + un border-bottom)
- Gap entre champs dans une section : 16px
- Padding intérieur d'une card : 20px

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomlefb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
