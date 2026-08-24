---
trigger: always_on
description: Application web pour **L'Oiseau Blanc Traiteur** permettant de gérer des frigos connectés installés en entreprise.
---

# CLAUDE.md — L'Oiseau Blanc Traiteur - Application Frigos Connectés

## Description du projet

Application web pour **L'Oiseau Blanc Traiteur** permettant de gérer des frigos connectés installés en entreprise.
Deux interfaces distinctes :
- **Panel Admin** : gestion utilisateurs, dashboard analytics, notifications, connexion Bicom
- **Panel Public** : consultation des plats, dépôt d'avis, préférences alimentaires, inscription notifications

Le client est Frédéric Bartoli, gérant de L'Oiseau Blanc Traiteur.

## Stack technique

- **Frontend** : React (Vite), React Router, Tailwind CSS
- **Backend** : Node.js, Express
- **Base de données** : PostgreSQL
- **ORM** : Prisma
- **Auth admin** : JWT + bcrypt
- **Emails** : Mailchimp API (déjà utilisé par le client)
- **Push notifications** : Web Push (service workers) — à confirmer
- **API externe** : Bicom (lecture seule, clé API en attente)

## Structure du projet

```
oiseau-blanc-app/
├── CLAUDE.md
├── client/                  # Frontend React
│   ├── src/
│   │   ├── components/      # Composants réutilisables
│   │   ├── pages/
│   │   │   ├── admin/       # Pages panel admin
│   │   │   └── public/      # Pages panel public
│   │   ├── hooks/           # Custom hooks
│   │   ├── services/        # Appels API (axios)
│   │   ├── context/         # Auth context, etc.
│   │   └── utils/
│   └── package.json
├── server/                  # Backend Node.js/Express
│   ├── src/
│   │   ├── routes/          # Routes API
│   │   ├── controllers/     # Logique métier
│   │   ├── middleware/      # Auth, validation, error handling
│   │   ├── services/        # Bicom, Mailchimp, push
│   │   └── utils/
│   ├── prisma/
│   │   └── schema.prisma    # Schéma BDD
│   └── package.json
├── database/                # Scripts SQL complémentaires, seeds
└── README.md
```

## Modèle de données

### admins
- id (UUID, PK)
- email (unique, not null)
- password_hash (not null)
- role (enum: SUPER_ADMIN, ADMIN)
- created_at, updated_at

### subscribers
Utilisateurs du panel public. Pas de mot de passe, identification par email/téléphone.
- id (UUID, PK)
- email (unique, nullable)
- phone (nullable)
- consent_email (boolean, default false)
- consent_push (boolean, default false)
- push_token (nullable)
- created_at, updated_at

Contrainte : au moins email OU phone requis.

### reviews
- id (UUID, PK)
- subscriber_id (FK → subscribers, not null)
- dish_id (string, référence Bicom)
- rating (integer, 1-5)
- comment (text, nullable)
- created_at

### preference_surveys
Sondages créés par l'admin (questions récurrentes type qualité/prix, quantité).
- id (UUID, PK)
- title (not null)
- questions (JSONB) — structure : [{id, label, type, options}]
- active (boolean)
- created_at, updated_at

### preference_responses
- id (UUID, PK)
- survey_id (FK → preference_surveys)
- subscriber_id (FK → subscribers)
- answers (JSONB)
- created_at

### menu_votes
Votes sur les menus à venir (ex: "dans 10 jours, couscous ou penne ?").
- id (UUID, PK)
- title (not null)
- options (JSONB) — liste des plats proposés
- vote_deadline (timestamp)
- created_at

### menu_vote_responses
- id (UUID, PK)
- menu_vote_id (FK → menu_votes)
- subscriber_id (FK → subscribers)
- selected_options (JSONB)
- created_at

## Fonctionnalités par lot

### Panel Admin
1. **Auth admin** — login JWT, gestion comptes admin, reset password
2. **Gestion utilisateurs** — CRUD admins, visualisation subscribers
3. **Connexion Bicom** — ⚠️ EN ATTENTE (clé API pas encore reçue). Préparer un service Bicom avec des données mockées en attendant. Lecture seule : frigos, plats, stocks, prix.
4. **Notifications** — intégration Mailchimp API (ajout contacts, déclenchement campagnes), fréquence configurable par l'admin
5. **Dashboard** — moyennes par plat, tendances avis, résultats sondages/votes. Export CSV/PDF (nice to have).
6. **Gestion sondages** — création/modification des formulaires de préférences
7. **Gestion votes menus** — proposer des menus futurs, consulter les résultats

### Panel Public (sans compte)
1. **Interface principale** — sélection frigo, liste plats disponibles, fiche détaillée
2. **Dépôt d'avis** — formulaire notation + commentaire, email ou téléphone obligatoire
3. **Préférences** — réponse aux sondages actifs
4. **Vote menus** — choix parmi les menus proposés
5. **Inscription notifications** — collecte email, consentement push/email

## Conventions de code

- **Langue du code** : anglais (variables, fonctions, composants)
- **Langue des contenus/UI** : français
- **Nommage** : camelCase (JS), PascalCase (composants React), snake_case (colonnes BDD)
- **API REST** : préfixe `/api/v1/`, JSON
- **Routes admin** : `/api/v1/admin/*` (protégées par middleware JWT)
- **Routes public** : `/api/v1/public/*` (ouvertes ou auth légère par email)
- **Gestion erreurs** : middleware centralisé, codes HTTP standards
- **Validation** : Zod côté serveur
- **Pas de console.log en production** — utiliser un logger (winston ou pino)

## Variables d'environnement attendues

```env
# Server
PORT=3001
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/oiseau_blanc
JWT_SECRET=

# Bicom (en attente)
BICOM_API_URL=
BICOM_API_KEY=

# Mailchimp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NolanMartinez/oiseau-blanc-app](https://github.com/NolanMartinez/oiseau-blanc-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
