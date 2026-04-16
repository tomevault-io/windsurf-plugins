---
trigger: always_on
description: Tu travailles sur POURACCORD, une plateforme B2B2C française permettant :
---

# ============================================================
# POURACCORD — Cursor AI Rules
# Plateforme B2B2C de gestion et validation de dossiers locataires
# Version : 1.0 | Février 2026
# ============================================================
# Ce fichier est lu par Cursor à chaque conversation.
# Il donne à l'IA le contexte permanent du projet.
# Ne pas supprimer ou modifier les sections marquées [CRITIQUE].
# ============================================================


# ────────────────────────────────────────────────────────────
# 1. CONTEXTE PRODUIT
# ────────────────────────────────────────────────────────────

Tu travailles sur POURACCORD, une plateforme B2B2C française permettant :
- Aux LOCATAIRES (gratuit) : constituer un dossier unique sécurisé et le partager avec des agences
- Aux AGENCES (800€ HT/mois, essai 30j) : accéder à des dossiers pré-vérifiés par IA anti-fraude
- Aux ADMINS : modérer, gérer les utilisateurs et améliorer l'IA

Valeur clé : validation anti-fraude multicouche par IA (8 niveaux) + conformité RGPD automatisée.

Environnements :
- Dev local     : localhost:3001 (backend) | localhost:3000 (frontend)
- Staging       : staging.pouraccord.com
- Production    : api.pouraccord.com / pouraccord.com


# ────────────────────────────────────────────────────────────
# 2. STACK TECHNIQUE [CRITIQUE]
# ────────────────────────────────────────────────────────────

## Backend
- Runtime       : Node.js 20 LTS
- Framework     : Express 4.x
- Langage       : TypeScript 5.x strict
- ORM           : Sequelize 6.x (MySQL)
- Base de données: MySQL 8.0+ (charset utf8mb4, moteur InnoDB)
- Validation    : Joi (TOUS les inputs sans exception)
- Auth          : jsonwebtoken + speakeasy (2FA TOTP)
- Upload        : Multer + AWS SDK v3 (S3)
- Email         : Nodemailer + SendGrid
- Paiement      : Stripe Node.js SDK
- Logs          : Winston (src/utils/logger.ts)
- Tâches cron   : node-cron
- Sécurité      : helmet, express-rate-limit, cors

## Frontend
- Framework     : React 18.x
- Build         : Vite 4.x
- Langage       : TypeScript 5.x strict
- État global   : Redux Toolkit + react-redux
- Routage       : React Router v6
- Formulaires   : React Hook Form + Yup
- HTTP          : Axios (avec interceptors JWT)
- UI            : Tailwind CSS 3.x + Headless UI
- i18n          : react-i18next (FR prioritaire)

## Microservice IA (Python — dépôt séparé)
- Framework     : FastAPI
- OCR           : Tesseract 5.x
- ML            : scikit-learn / XGBoost
- Communication : HTTP REST (appelé par backend Node.js)
- Endpoint      : POST /analyze

## Tests
- Backend       : Jest + Supertest (cible couverture : 80% minimum)
- Frontend      : React Testing Library + Cypress (E2E)
- Helpers tests : src/__tests__/helpers.ts (createActiveUser, loginTestUser, etc.)

## CI/CD
- GitHub Actions (.github/workflows/ci.yml)
- Lint → Tests → Coverage → Build → Deploy staging


# ────────────────────────────────────────────────────────────
# 3. STRUCTURE DU PROJET [CRITIQUE]
# ────────────────────────────────────────────────────────────

pouraccord/
├── backend/
│   ├── src/
│   │   ├── config/          # env.ts, database.ts, s3.ts, stripe.ts
│   │   ├── controllers/     # AuthController, UserController, FolderController...
│   │   ├── middlewares/     # auth.middleware.ts, rateLimit.ts, audit.ts, upload.ts
│   │   ├── models/          # User.ts, Agency.ts, Folder.ts, Document.ts...
│   │   ├── routes/          # auth.routes.ts, users.routes.ts, folders.routes.ts...
│   │   ├── services/        # EmailService.ts, S3Service.ts, StripeService.ts, AIService.ts
│   │   ├── utils/           # logger.ts, response.ts, validators.ts, crypto.ts
│   │   ├── jobs/            # cleanupExpiredDocuments.ts, cleanupRefreshTokens.ts
│   │   ├── types/           # express.d.ts, index.ts
│   │   └── app.ts           # Point d'entrée Express
│   ├── migrations/          # Séquentiels : 001_create_users.ts, 002_create_agencies.ts...
│   ├── seeders/             # document_types.ts, admin_user.ts
│   └── __tests__/
│       ├── helpers.ts       # createActiveUser(), loginTestUser(), createTestAgency()...
│       ├── setup.ts         # Connexion DB test, migrations
│       └── teardown.ts      # Nettoyage DB test
│
├── frontend/
│   ├── src/
│   │   ├── components/      # Composants réutilisables (Button, Input, Modal...)
│   │   ├── pages/           # Register, Login, Dashboard, Documents, Settings...
│   │   ├── store/           # authSlice.ts, folderSlice.ts, store.ts
│   │   ├── services/        # api.ts (Axios instance), auth.service.ts...
│   │   ├── hooks/           # useAuth.ts, useFolder.ts, useDebounce.ts
│   │   ├── types/           # User.ts, Folder.ts, Document.ts, Agency.ts
│   │   └── utils/           # formatters.ts, validators.ts, constants.ts
│   └── cypress/             # Tests E2E
│
├── .cursorrules             # CE FICHIER
├── .cursor/mcp.json         # Configuration MCP (Trello, etc.)
└── .github/workflows/       # CI/CD


# ────────────────────────────────────────────────────────────
# 4. FORMAT DE RÉPONSE API [CRITIQUE — JAMAIS DÉROGER]
# ────────────────────────────────────────────────────────────

TOUTES les réponses API utilisent OBLIGATOIREMENT ce format :

```typescript
// Succès
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FredSBAF) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
