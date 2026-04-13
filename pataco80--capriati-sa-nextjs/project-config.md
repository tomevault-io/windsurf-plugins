---
trigger: always_on
description: ![MIT License](https://img.shields.io/badge/license-MIT-blue)
---

# 📚 Capriati S.A. – Peinture & Décoration

![MIT License](https://img.shields.io/badge/license-MIT-blue)
[![TypeScript](https://badgen.net/badge/icon/typescript?icon=typescript&label)](https://typescriptlang.org)
![ESLint](https://img.shields.io/badge/code%20style-eslint-brightgreen)

Site web de l’entreprise de peinture Capriati S.A., développé avec Next.js 15 et TypeScript.

## 🚀 Technologies

- **Framework**: [Next.js 15](https://nextjs.org/)
- **Langage**: [TypeScript](https://www.typescriptlang.org/)
- **Styling**: [Tailwind CSS V4](https://tailwindcss.com/)
- **UI Components**: [Shadcn UI](https://ui.shadcn.com/)
- **Animations**: Framer Motion
- **CMS**: [Sanity](https://www.sanity.io/)
- **Email**: [Resend](https://resend.com/)
- **Déploiement**: [Vercel](https://vercel.com/)

## 🛠️ Technologies Incluses

- **Next.js 15**
- **React 19**
- **TypeScript 5**
- **ESLint 9**
- **Prettier 3**
- **Tailwind CSS 4**
- **Shadcn UI**
- **App Directory**
- **System, Light & Dark Mode**
- **Next.js Bundle Analyzer**

### 🔧 Plugins ESLint

- [**@eslint/js**](https://www.npmjs.com/package/@eslint/js) - Configuration ESLint moderne
- [**typescript-eslint**](https://github.com/typescript-eslint/typescript-eslint) - Support TypeScript
- [**eslint-plugin-react**](https://github.com/jsx-eslint/eslint-plugin-react) - Règles React
- [**@next/eslint-plugin-next**](https://github.com/vercel/next.js) - Règles Next.js
- [**eslint-config-prettier**](https://github.com/prettier/eslint-config-prettier) - Compatibilité Prettier
- [**eslint-plugin-tailwindcss**](https://github.com/francoismassart/eslint-plugin-tailwindcss) - Règles Tailwind
- [**eslint-plugin-import**](https://github.com/import-js/eslint-plugin-import) - Gestion des imports
- [**eslint-plugin-promise**](https://github.com/eslint-community/eslint-plugin-promise) - Gestion des promesses

### ✨ Plugins Prettier

- [**@trivago/prettier-plugin-sort-imports**](https://github.com/trivago/prettier-plugin-sort-imports) - Tri automatique des imports
- [**prettier-plugin-tailwindcss**](https://github.com/tailwindlabs/prettier-plugin-tailwindcss) - Formatage Tailwind

## 🎯 Principes Fondamentaux

1. **Performance**

   - SSG (Static Site Generation) pour toutes les pages
   - Optimisation des images avec`next/image`
   - Revalidation ISR pour le contenu dynamique

2. **Accessibilité**

   - Respect des normes WCAG 2.1
   - Navigation au clavier optimisée
   - Zoom de la typographie fluid avec fallbacks

3. **UX/UI**

   - Design responsive
   - Animations fluides avec Framer Motion

## 📁 Structure du Projet

```md
📦 capriati-sa/
├── 📚 .cursor/                # Règles, conventions, doc interne (Markdown)
│   └── 📂 rules/
├── 📂 public/                 # Fichiers statiques (images, favicon, etc.)
├── 📂 src/
│   ├── 📂 app/                # Routing Next.js 15 (pages, layouts, templates)
│   │   ├── 📝 page.tsx        # Accueil
│   │   ├── 📝 layout.tsx      # Layout principal
│   │   ├── 📂 about/          # Page À propos
│   │   ├── 📂 contact/        # Page Contact
│   │   ├── 📂 services/       # Catalogue des services
│   │   │   ├── 📂 [slug]/     # Pages dynamiques pour chaque service
│   │   ├── 📂 works/          # Réalisations (galerie, projets)
│   │   │   ├── 📂 [slug]/     # Pages dynamiques pour chaque réalisation
│   │   └── 📝 globals.css     # Styles globaux
│   ├── 🧩 components/
│   │   ├── 🧩 ui/             # Composants Shadcn UI (ne pas modifier)
│   │   ├── 🧩 shared/         # Composants réutilisables (Header, Footer, etc.)
│   │   └── 🧩 pages/          # Composants spécifiques à une page
│   ├── 🔧 lib/                # Fonctions utilitaires, config Sanity, hooks, etc.
│   │   ├── 🔧 sanity/         # Config et clients Sanity
│   │   ├── 🔧 resend/         # Configuration de resend pour les formulaires
│   │   └── 🔧 utils/          # Fonctions utilitaires diverses
│   ├── 🎨 styles/             # Fichiers CSS/variables complémentaires
│   ├── 🔷 types/              # Types TypeScript partagés
│   ├── 🎣 hooks/              # Hooks de l'application
│   └── ✅ tests/              # Tests unitaires et d’intégration
├── 📝 .env.local              # Variables d’environnement (non versionné)
├── 📝 .eslintrc.json          # Config ESLint
├── 📝 .prettierrc             # Config Prettier
├── tailwind.config.ts      # Config Tailwind
├── 📝 next.config.mjs         # Config Next.js
├── 📝 package.json
├── 📝 pnpm-lock.yaml
└── 📝 README.md
```

## 📝 Conventions de Code

1. **Nommage**

   - Composants : PascalCase
   - Fonctions : camelCase
   - Types : PascalCase
   - Variables : camelCase

2. **Organisation**

   - Un composant par fichier
   - Types dans des fichiers séparés
   - Styles avec Tailwind CSS

3. **Documentation**

   - JSDoc pour les fonctions complexes
   - README dans chaque dossier important
   - Commentaires en français

## 🔄 Workflow Git

1. **Branches**

   - `main` : Production

     - 🔒 Protection contre les push directs
     - 🔍 Pull Request requis
     - 👥 Reviews approuvées requises (1 minimum)
     - ✅ Status checks requis
     - 🏷️ Tags sémantiques obligatoires

   - `develop` : Développement

     - 🔍 Pull Request requis
     - 👥 Reviews approuvées requises (1 minimum)
     - ✅ Status checks requis

   - `feature/*` : Nouvelles fonctionnalités

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pataco80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
