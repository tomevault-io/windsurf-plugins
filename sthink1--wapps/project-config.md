---
trigger: always_on
description: - **Node.js**: v18+ (recommended)
---

# CLAUDE.md – WonderfulApps Developer Onboarding Guide

## Tech Stack & Versions

### Runtime & Framework
- **Node.js**: v18+ (recommended)
- **Express.js**: ^4.18.2 (actual: 4.18.2+)
- **Runtime**: JavaScript (ES6+)

### Database
- **Primary**: MySQL 5.5.x (Remote Host) / MySQL 8.0 (Local Dev)
- **Driver**: mysql2/promise ^3.6.5
- **Connection Pool**: 50 connections max, queue unlimited
- **Charset**: utf8mb4
- **Date Handling**: DATE fields returned as YYYY-MM-DD strings

### Authentication & Security
- **JWT Library**: jsonwebtoken ^9.0.2
- **Password Hashing**: bcrypt ^5.1.1
- **Authorization**: Bearer token in `Authorization` header
- **Token Expiry**: 8 hours
- **Secret Storage**: `process.env.JWT_SECRET` (environment variable only)

### Validation & Middleware
- **Input Validation**: express-validator ^7.3.1 (actual installed)
- **CORS**: cors ^2.8.5
- **File Upload**: multer ^1.4.5-lts.1
- **HTTP Logging**: morgan ^1.10.0
- **Logging Framework**: winston ^3.11.0

### Frontend & PWA
- **PWA Service Worker**: Registered via `/sw.js`
- **Caching Strategy**: Cache-first for static assets, no-cache for service worker
- **Confetti Animation**: canvas-confetti @1.9.3 (CDN)
- **Input Sanitization**: DOMPurify 2.3.10 (CDN)

### HTTP Client & External APIs
- **HTTP Client**: axios ^1.x (for external API calls)
- **ETF Data APIs**: Tiingo, Finnhub, Polygon (price & reference data via axios)

### Development & Email
- **Environment Variables**: dotenv ^16.3.1
- **Email Service**: Custom `send_email.js` (Resend HTTP-based)
- **Reverse Geocoding**: Nominatim (OSM API proxy via `/routes/geocode.js`)

---

## Core Commands

### Development
```bash
# Start development server (nodemon recommended)
npm start

# Start with explicit port
PORT=8080 npm start

# Run with verbose logging
NODE_ENV=development npm start
```

### Testing
```bash
# Test database connection
node testConnection.js

# Hash a password (one-off utility)
node hash.js
```

### Production
```bash
# Set environment to production
NODE_ENV=production npm start

# Run on non-standard port
PORT=3000 NODE_ENV=production npm start
```

### Database
- **Restore Schema**: Import `wappsDump.sql` into your MySQL instance
- **Connection String**: Built from `.env` variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_PORT`)

---

## Database Rules & Architecture

### Critical Constraints
1. **Foreign Key Constraints with Cascading Deletes**: All tables with parent references use `ON DELETE CASCADE ON UPDATE CASCADE`.
   - Tables with FKs: ActivitiesT, etfCategoryT, etfSymbolT, InterestEarnedT, TrackUsageT, UserSequenceT, WeightActivitiesT, WeightsT
   - All cascade to UsersT (or intermediate tables)
   - Implication: Deleting a user automatically deletes child records at the database level; no app-level cascade code needed.

2. **MySQL 5.5 Compatibility**:
   - No `JSON` data type (avoid storing JSON columns)
   - No `GENERATED` columns
   - Limited window functions (use application-level grouping)
   - Date strings must be in `YYYY-MM-DD` format

3. **Dual-Key Architecture**: `AUTO_INCREMENT` primary keys + user-scoped secondary keys
   - Global PKs (WeightID, ActivityID, IntErndID, etc.) ensure app-wide uniqueness
   - User-scoped IDs (UserWeightID, UserActivityID, UserIntErndID, etc.) via `UserSequenceT` table provide per-user sequences (1,2,3...)
   - Both are actively used: PKs for joins, user-scoped IDs for user-friendly URLs and API parameters
   - Managed via `dbConnection.js::getNextUserSpecificID(userId, tableName)` — generates and increments UserSequenceT entries
   - FK to UsersT: `ON DELETE CASCADE` removes sequence data when user is deleted

### Key Tables & Relationships

| Table | Purpose | Parent | Child Seq |
|-------|---------|--------|-----------|
| `UsersT` | User accounts | — | UserSequenceT |
| `WeightsT` | Weight entries | UsersT | UserWeightID |
| `ActivitiesT` | Activity definitions | UsersT | UserActivityID |
| `WeightActivitiesT` | Weight-Activity mapping | WeightsT, ActivitiesT | (none) |
| `InterestEarnedT` | Interest contracts | UsersT | UserIntErndID |
| `TrackUsageT` | Page views & time spent | UsersT | (none) |
| `etfCategoryT` | ETF portfolio categories | UsersT | (none) |
| `etfSymbolT` | ETF symbols & metadata | UsersT, etfCategoryT | (none) |
| `UserSequenceT` | ID generation (manual) | UsersT (implicit) | (none) |

### Data Flow Pattern
```
User Input (Frontend)
  → Validation (express-validator)
  → Auth Middleware (JWT check)
  → Route Handler (routes/*.js)
  → Transaction Wrapper (withTransaction)
  → Pool Query (mysql2/promise)
  → Response JSON
```

---

## Security & Authentication

### Mechanism
- **JWT (JSON Web Tokens)** issued upon successful login
- Token contains: `{ userId, username, expiresIn: '8h' }`
- Verified in `middleware/auth.js` before protected routes

### Token Lifecycle
1. **Registration** (`/users/register`): Hash password with bcrypt (10 rounds), generate token
2. **Login** (`/users/login`): Verify credentials, issue token
3. **Protected Routes**: Extract token from `Authorization: Bearer <token>` header
4. **Token Expiry**: 8 hours; client must re-login

### Security Notes
- **Remote Database**: Development only; no production PII storage
- **SSL/TLS**: 
  - Enabled locally (MySQL 8.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sthink1/wapps](https://github.com/sthink1/wapps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
