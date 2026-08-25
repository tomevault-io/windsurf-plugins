---
trigger: always_on
description: > **Nom provisoire :** StudyForge
---

# StudyForge — Architecture & Developer Guide

> **Nom provisoire :** StudyForge
> **But :** Application mobile de révision intelligente pour étudiants
> **Stack :** React Native + Expo (SDK 52) + TypeScript · Supabase · Anthropic API · RevenueCat

---

## Table des matières

1. [Stack technique](#stack-technique)
2. [Structure du projet](#structure-du-projet)
3. [Navigation](#navigation)
4. [Services](#services)
5. [Base de données Supabase](#base-de-données-supabase)
6. [Intégration Anthropic (IA)](#intégration-anthropic-ia)
7. [Monétisation RevenueCat](#monétisation-revenuecat)
8. [Variables d'environnement](#variables-denvironnement)
9. [Conventions de code](#conventions-de-code)
10. [Commandes utiles](#commandes-utiles)
11. [Roadmap](#roadmap)

---

## Stack technique

| Couche | Technologie | Rôle |
|---|---|---|
| Framework mobile | React Native + Expo SDK 52 | UI cross-platform (iOS / Android) |
| Langage | TypeScript (strict mode) | Typage statique |
| Navigation | Expo Router v4 (file-based) | Routing déclaratif |
| Backend / Auth | Supabase | Auth, PostgreSQL, Storage, Edge Functions |
| IA générative | Anthropic Claude (claude-opus-4-6) | Génération de fiches & quizz |
| Paiements | RevenueCat | Abonnements in-app (free / pro) |
| Style | StyleSheet natif | Pas de lib CSS-in-JS externe |

---

## Structure du projet

```
studyforge/
├── app/                        # Expo Router — routes de l'app
│   ├── _layout.tsx             # Root layout (SplashScreen, providers)
│   ├── (tabs)/                 # Groupe tabs (nav bar principale)
│   │   ├── _layout.tsx         # Configuration de la tab bar
│   │   ├── index.tsx           # Accueil
│   │   ├── library.tsx         # Bibliothèque de fiches
│   │   ├── quiz.tsx            # Quizz
│   │   └── profile.tsx         # Profil & abonnement
│   └── (auth)/                 # Groupe auth (non connecté)
│       ├── _layout.tsx
│       ├── login.tsx
│       └── register.tsx
│
├── components/                 # Composants réutilisables
│   ├── navigation/
│   │   └── TabBarIcon.tsx      # Icône Ionicons pour la tab bar
│   ├── ui/
│   │   ├── Button.tsx          # Bouton générique (primary/secondary/danger)
│   │   └── Card.tsx            # Conteneur carte avec ombre
│   ├── flashcards/
│   │   └── FlashcardItem.tsx   # Élément de liste de fiche
│   └── quiz/
│       └── QuizCard.tsx        # Carte de question QCM
│
├── screens/                    # Écrans complets (utilisés dans les routes ou modales)
│   ├── FlashcardDetailScreen.tsx
│   └── GenerateFlashcardScreen.tsx
│
├── services/                   # Couche d'accès aux données / APIs externes
│   ├── supabase.ts             # Client Supabase + helpers auth & DB
│   ├── anthropic.ts            # Appels Edge Functions pour l'IA
│   └── revenuecat.ts           # Initialisation et helpers RevenueCat
│
├── hooks/                      # Custom hooks React
│   ├── useAuth.ts              # Session Supabase (subscribe onAuthStateChange)
│   ├── useFlashcards.ts        # CRUD fiches + état loading/error
│   └── useSubscription.ts      # Tier (free/pro) via RevenueCat
│
├── types/                      # Types TypeScript globaux
│   ├── index.ts                # UserProfile, Flashcard, QuizQuestion, etc.
│   └── database.ts             # Types générés Supabase (schéma DB)
│
├── constants/
│   └── Colors.ts               # Palette de couleurs (light + dark)
│
├── assets/                     # Images, icônes, splash
│
├── supabase/                   # (À créer) Edge Functions Deno
│   └── functions/
│       ├── generate-flashcard/ # Wrapper Anthropic → fiche structurée
│       └── generate-quiz/      # Wrapper Anthropic → tableau de questions
│
├── app.json                    # Config Expo (bundle ID, scheme, plugins)
├── package.json
├── tsconfig.json               # strict + path aliases (@/*)
├── babel.config.js             # babel-preset-expo + module-resolver
└── CLAUDE.md                   # Ce fichier
```

---

## Navigation

L'app utilise **Expo Router v4** (file-based routing, similaire à Next.js).

### Groupes de routes

| Groupe | Chemin | Accès |
|---|---|---|
| `(tabs)` | `/`, `/library`, `/quiz`, `/profile` | Utilisateur connecté |
| `(auth)` | `/login`, `/register` | Utilisateur non connecté |

### Logique de redirection auth

À implémenter dans `app/_layout.tsx` avec `useAuth` :

```tsx
const { session, loading } = useAuth();

useEffect(() => {
  if (!loading) {
    if (session) router.replace('/(tabs)');
    else router.replace('/(auth)/login');
  }
}, [session, loading]);
```

### Ajout d'une route modale

Créer `app/flashcard/[id].tsx` → accessible via `router.push('/flashcard/abc123')`.

---

## Services

### `services/supabase.ts`

Client Supabase initialisé avec `expo-secure-store` pour la persistence de session native.

**Variables requises :**
- `EXPO_PUBLIC_SUPABASE_URL`
- `EXPO_PUBLIC_SUPABASE_ANON_KEY`

**Helpers exposés :**
- `signUp(email, password, firstName)` → inscription
- `signIn(email, password)` → connexion
- `signOut()` → déconnexion
- `getSession()` → session courante
- `getFlashcards(userId)` → liste des fiches
- `createFlashcard(data)` → création fiche
- `deleteFlashcard(id)` → suppression
- `getQuizSessions(userId)` → historique quizz
- `createQuizSession(data)` → nouvelle session quizz


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Momolastico08/app-revision-study](https://github.com/Momolastico08/app-revision-study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
