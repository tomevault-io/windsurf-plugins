---
trigger: always_on
description: L'objectif est de générer une base de site web moderne, modulaire, et hautement performante en react/vite. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).
---

# Blueprint: Architecture React Ultra-Optimisée (Full-Stack Ready) - 2026

## 1. Vision Générale

L'objectif est de générer une base de site web moderne, modulaire, et hautement performante en react/vite. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).

## 2. Stack Technique (Core)

Secteur,Technologie,Points clés
Framework,React Router v7,"Mode Framework (ex-Remix), SSR/SSG natif."
Runtime,React 19,"Support des Actions, use API et Suspense."
Build Tool,Vite.js,"Plugin RR7, HMR ultra-rapide, build optimisé."
Styling,Tailwind CSS v4,"Container Queries, cascade simplifiée, CSS natif."
State,Zustand,État global léger pour l'UI et la session client.
Data Fetching,TanStack Query,Cache serveur/client et sync temps réel.
I18N,i18next,Détection serveur (SEO) + switch client sans reload.
Animations,Framer Motion,Transitions de routes et micro-interactions.

## 3. Architecture de Projet (Feature-Based)

/
├── app/
│ ├── root.tsx # Entrée principale (Providers, I18N, SEO global)
│ ├── routes.ts # Définition centralisée des routes
│ ├── entry.server.tsx # Config du rendu côté serveur
│ ├── entry.client.tsx # Config de l'hydratation côté client
│ ├── routes/ # Pages et Layouts
│ │ ├── \_public.tsx # Layout SEO (Navbar, Footer)
│ │ ├── \_public.articles.tsx # Blog/Articles (Prerendered)
│ │ ├── \_auth.tsx # Layout Login/Register
│ │ └── dashboard.\_index.tsx # App privée (Client-only)
│ ├── components/
│ │ ├── ui/ # Atomes (Boutons, Inputs via Tailwind v4)
│ │ └── features/ # Molécules métier (Comparateur, Graphiques)
│ ├── lib/
│ │ ├── i18n/ # Config i18next (server & client)
│ │ └── query-client.ts # Singleton TanStack Query
│ └── store/ # Stores Zustand (authStore, uiStore)
├── public/
│ ├── locales/ # Fichiers JSON de traduction (fr/en)
│ └── assets/ # Images et polices statiques
├── vite.config.ts # Configuration Vite + RR7 + Tailwind
└── tsconfig.json # Config TypeScript stricte (Path Aliases)

1. Vision Générale
   L'objectif est de générer une base de site web moderne, modulaire, et hautement performante. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).
   . Remplacer useEffect par les Loaders
   Configuration Internationalisation (I18N)
   L'i18n est gérée en deux temps pour garantir un SEO parfait :

Serveur : Détection de la langue via URL, Cookie ou Header Accept-Language.

Client : Hydratation du dictionnaire et changement de langue instantané.

Utilisation dans un composant :

TypeScript

import { useTranslation } from "react-i18next";

export function PricingCard() {
const { t } = useTranslation();
return <button>{t("common.compare_prices")}</button>;
}
⚡ Flux de Données & Rendu

1. Rendu Hybride (SSR & Prerendering)
   Vitrine & Blog (SEO Priority) : Utilisation des loaders natifs de RR7. Les données sont récupérées côté serveur.

Zod Validation : Chaque retour d'API est validé via un schéma Zod avant d'atteindre le composant pour garantir l'intégrité du typage.

Résultat : Premier octet (TTFB) ultra-rapide et Indexation Google 100% fiable.

Dashboard (App Mode) : Utilisation du mode clientLoader pour les parties de l'application ne nécessitant pas de SEO, permettant une navigation instantanée type SPA sans solliciter le serveur inutilement.

2. Mutations & Formulaires (Progressive Enhancement)
   React 19 Actions : Remplacement des gestionnaires onSubmit classiques par les Actions de RR7.

Conform & Zod : Utilisation de la bibliothèque Conform pour la gestion des formulaires.

Validation côté serveur avec retour d'erreurs d'accessibilité immédiat.

Gestion native des états pending (chargement) sans besoin de useState manuel.

Note : Fonctionne même si le JavaScript n'est pas encore chargé.

3. Synchronisation & Cache (TanStack Query)
   Usage Sélectif : TanStack Query n'est plus utilisé pour le fetch initial (géré par les loaders), mais uniquement pour :

Polling/Temps Réel : Rafraîchissement des données du dashboard (ex: prix, notifications) en arrière-plan.

Optimistic UI : Mise à jour instantanée de l'interface avant confirmation serveur sur les actions complexes.

Persistance : Cache global partagé entre les routes pour éviter les fetchs redondants sur les ressources statiques.

4. Internationalisation (I18N)
   Détection : La langue est résolue dans le loader de la route root.tsx (via URL ou Header).

Zero-Flash : Le dictionnaire de traduction est injecté dans le contexte SSR, évitant tout "Layout Shift" ou texte non traduit au premier affichage.

♿ Accessibilité (A11y) & Qualité

- Respect strict des normes WCAG.
- Utilisation systématique des attributs ARIA.
- Gestion parfaite du focus clavier sur les éléments interactifs.
- Utilisation de Error Boundaries pour isoler les défaillances de composants.

🔍 SEO & Performance

- Intégration de JSON-LD pour chaque page (Articles, Organisation, Breadcrumbs).
- Utilise l'export meta de RR7.
- Mobile-First : Design responsive pensé d'abord pour le mobile.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fredconv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
