---
trigger: always_on
description: This project is a modern SaaS boilerplate built with Next.js 15, offering a complete architecture to quickly launch a SaaS application. It includes authentication, subscription management, quotas, file storage, organizations, and internationalization.
---

# SaaS Boilerplate - Complete Documentation

## Overview

This project is a modern SaaS boilerplate built with Next.js 15, offering a complete architecture to quickly launch a SaaS application. It includes authentication, subscription management, quotas, file storage, organizations, and internationalization.

## Stack Technique

### Frontend
- **Next.js 15** avec App Router et React 19
- **TypeScript** pour le typage statique
- **Tailwind CSS** pour le styling
- **Shadcn/ui** pour les composants UI
- **React Hook Form** + **Zod** pour la validation des formulaires
- **next-intl** pour l'internationalisation (FR/EN)

### Backend
- **tRPC** pour les API type-safe
- **Prisma** comme ORM
- **PostgreSQL** comme base de données
- **Lucia Auth** pour l'authentification
- **Stripe** pour les paiements et abonnements

### Services Externes
- **SendGrid** pour l'envoi d'emails
- **Firebase Storage** / **Cloudflare R2** pour le stockage de fichiers
- **Google reCAPTCHA** pour la protection anti-spam

## Architecture du Projet

```
apply-backend/
├── app/                    # App Router Next.js
│   ├── [locale]/          # Routes internationalisées
│   │   ├── (protected)/   # Routes protégées (auth requise)
│   │   └── (withLayout)/  # Routes publiques avec layout
│   ├── api/               # Routes API REST
│   └── _trpc/             # Configuration tRPC
├── server/                # Serveur tRPC
│   ├── routes/            # Routes tRPC
│   └── trpc.ts           # Configuration de base tRPC
├── actions/               # Server Actions Next.js
├── components/            # Composants React
├── lib/                   # Utilitaires et configurations
│   ├── lucia/            # Configuration auth
│   ├── database/         # Configuration DB
│   └── storage/          # Services de stockage
├── prisma/                # Schéma et migrations DB
├── hooks/                 # Custom React hooks
├── providers/             # Context providers React
└── types/                 # Types TypeScript

```

## Système d'Authentification

### Vue d'ensemble
Le système utilise **Lucia Auth v3** avec plusieurs méthodes d'authentification :

1. **Magic Link** (Email)
2. **OAuth** (Google, GitHub, Facebook)

### Architecture de l'Auth

#### 1. Sessions avec Lucia
- Gestion des sessions côté serveur avec cookies HTTP-only
- Validation automatique des sessions via middleware
- Renouvellement automatique des sessions expirées

#### 2. Magic Link Flow
```typescript
// actions/magic-link.actions.ts
1. Utilisateur entre son email
2. Génération d'un JWT signé (5 min de validité)
3. Envoi du lien par email via SendGrid
4. Validation du token et création de session
5. Attribution automatique du plan "free_trial" aux nouveaux utilisateurs
```

#### 3. OAuth Flow
```typescript
// lib/lucia/oauth.ts
1. Redirection vers le provider OAuth
2. Callback avec code d'autorisation
3. Échange du code contre access token
4. Récupération des infos utilisateur
5. Création/mise à jour de l'utilisateur en DB
6. Création de session Lucia
```

### Modèles de Données Auth

```prisma
model User {
  id                   String    @id @default(uuid())
  email                String    @unique
  isEmailVerified      Boolean   @default(false)
  profilePictureUrl    String?
  stripeCustomerId     String?   @unique
  
  // Relations
  subscription         Subscription?
  oauthAccounts        OauthAccount[]
  magicLinks           MagicLink[]
  sessions             Session[]
  ProductUsage         ProductUsage[]
}

model Session {
  id        String   @id @default(uuid())
  userId    String
  expiresAt DateTime
  expiresIn Int
  user      User     @relation(...)
}

model OauthAccount {
  id             String   @id
  userId         String
  provider       String   // "google", "github", "facebook"
  providerUserId String
  accessToken    String
  refreshToken   String?
  expiresAt      DateTime
  user           User     @relation(...)
}
```

### Protection des Routes

#### 1. Middleware (middleware.ts)
- Vérifie les sessions sur toutes les routes
- Met à jour automatiquement les cookies de session
- Renouvelle les quotas si nécessaire

#### 2. Layout Protégé
```typescript
// app/[locale]/(protected)/layout.tsx
- Vérifie l'authentification côté serveur
- Redirige vers /sign-in si non authentifié
- Charge les données utilisateur et abonnement
```

#### 3. Composants d'Autorisation
- `<HasAuthorization>` : Vérifie les permissions côté client
- `<HasAutorizationServer>` : Vérifie les permissions côté serveur
- `<CheckQuota>` : Vérifie les quotas avant d'afficher le contenu

## Système d'Abonnements et Quotas

### Architecture des Abonnements

```prisma
model Subscription {
  id            String    @id
  planTitle     String    // ex: "free_trial", "starter", "pro"
  price         Decimal
  timeline      Timeline  // MONTHLY, YEARLY, ONETIME
  stripePriceId String?
  
  // Relations
  features      Feature[]  // Liste des fonctionnalités
  products      Product[]  // Produits avec quotas
  users         User[]
}

model Product {
  id             String  @id

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meirankri/saas-boilerplate](https://github.com/meirankri/saas-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
