---
trigger: always_on
description: > **Résumé en une ligne**: Plateforme d'apprentissage de code avec éditeur Monaco, système de gems et niveau urbain ALGOBOG (2500 problèmes algorithmiques)
---

# 🎯 Projet: CodeBog Web

> **Résumé en une ligne**: Plateforme d'apprentissage de code avec éditeur Monaco, système de gems et niveau urbain ALGOBOG (2500 problèmes algorithmiques)

---

## 📋 Contexte Projet

**Type**: Plateforme d'apprentissage
**Statut**: En développement

---

## 🛠️ Stack Technique

### Frontend
- **Framework**: Next.js 16.1.1 + React 19
- **Styling**: Tailwind CSS 4
- **Éditeur**: Monaco Editor
- **Build**: Next.js CLI

### Backend
- **BaaS**: Appwrite (node-appwrite)
- **Paiements**: Stripe
- **Sandbox**: isolated-vm pour exécution de code

### Infrastructure
- **Tests**: Vitest + Testing Library + Playwright

---

## 🔧 Commandes Essentielles

```bash
npm install           # Installation
npm run dev           # Dev server
npm run build         # Build production
npm run test          # Tests Vitest
npm run test:run      # Tests en mode CI
npm run lint          # ESLint

# Scripts ALGOBOG
npx tsx scripts/setup-algobog-collections.ts  # Créer les collections Appwrite
npx tsx scripts/seed-algobog-data.ts          # Seeder districts et buildings
npx tsx scripts/import-algobog-problems.ts    # Importer les problèmes

# Scripts existants
npm run sync:piscine      # Sync données piscine
npm run setup:submissions # Setup soumissions
npm run setup:gems        # Setup collections gems
```

---

## 📁 Architecture

```
/
├── src/
│   ├── app/
│   │   └── api/
│   │       ├── algobog/
│   │       │   ├── unlock/route.ts       → API achat unlock avec gems
│   │       │   └── submissions/route.ts  → API soumission de solutions
│   │       ├── gems/                     → APIs système de gems
│   │       └── submissions/              → APIs soumissions JS/C
│   └── lib/
│       ├── algobog/
│       │   ├── access-control.ts         → Contrôle d'accès (progression + gems)
│       │   └── gem-config.ts             → Prix des unlocks en gems
│       ├── gems/                         → Gestion du solde gems
│       └── sandbox.ts                    → Exécution code isolée
├── scripts/
│   ├── setup-algobog-collections.ts      → Création collections Appwrite
│   ├── seed-algobog-data.ts              → Seed districts + buildings
│   └── import-algobog-problems.ts        → Import problèmes depuis curriculum
├── tests/                                → Tests Playwright E2E
├── public/                               → Assets statiques
└── ALGOBOG_PLAN.md                       → Documentation architecture ALGOBOG
```

---

## 🏙️ ALGOBOG - Niveau Urbain Algorithmique

### Structure
- **6 Districts** (phases) : Downtown → Industrial → Transit → Tech Park → Research → Skyline
- **33 Buildings** (modules) : Un par pattern algorithmique
- **2500 Problèmes** : LeetCode reformulés (1129 importés actuellement)

### Collections Appwrite

| Collection | Description | Accès |
|------------|-------------|-------|
| `algo-districts` | 6 phases/districts | Lecture: users |
| `algo-buildings` | 33 modules/buildings | Lecture: users |
| `algo-problems` | Problèmes algorithmiques | Lecture: users |
| `algo-submissions` | Soumissions utilisateur | API only |
| `algo-progress` | Progression par building | API only |
| `algo-unlocks` | Déblocages avec gems | API only |

### Système de Progression

1. **Progression naturelle** : Résoudre un problème débloque le suivant (SANS gems)
2. **Skip avec gems** : Acheter un unlock pour sauter un niveau
3. **Hiérarchie** : District → Building → Problem (chaque niveau vérifie le parent)

### APIs ALGOBOG

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/api/algobog/unlock` | GET | Vérifier statut unlock et coût |
| `/api/algobog/unlock` | POST | Acheter un unlock avec gems |
| `/api/algobog/submissions` | GET | Historique soumissions d'un problème |
| `/api/algobog/submissions` | POST | Soumettre une solution |

### Logique de déverrouillage (`src/lib/algobog/access-control.ts`)

```typescript
// Ordre de vérification pour isProblemUnlocked:
1. Admin/Moderator → accès total (unlockAll)
2. Premier problème du building → toujours accessible
3. Gem unlock → accès si acheté
4. Progression → accès si problème précédent réussi (passed: true)
```

### Prix des gems (`src/lib/algobog/gem-config.ts`)

| Type | Coût |
|------|------|
| District (sauf Downtown) | 100-500 gems |
| Building (sauf 1er) | 25-75 gems |
| Problem Easy | 5 gems |
| Problem Medium | 15 gems |
| Problem Hard | 30 gems |

---

## ⚠️ Points d'Attention

- **isolated-vm**: Sandbox pour exécution sécurisée du code utilisateur
- **Appwrite**: Vérifier les permissions des collections (API-only pour données sensibles)
- **Monaco Editor**: Attention à la taille du bundle (lazy loading)
- **Progression ALGOBOG**: Toujours vérifier `hasCompletedProblem` côté serveur
- **Gems**: Calcul du coût côté serveur uniquement (jamais faire confiance au client)

---

## 🤖 Instructions Claude

- Réponses en français
- Utiliser async/await (pas de .then())
- Tests obligatoires pour les nouvelles fonctionnalités
- Ne pas modifier les scripts de migration sans validation
- Pour ALGOBOG: toujours vérifier l'accès côté serveur avant d'exécuter le code
- Consulter `ALGOBOG_PLAN.md` pour l'architecture détaillée

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p4cm4n972/codebog](https://github.com/p4cm4n972/codebog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
