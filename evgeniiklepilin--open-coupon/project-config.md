---
trigger: always_on
description: **Description:** An open-source, full-stack browser extension framework (Honey clone) that automatically applies coupon codes at checkout.
---

# CLAUDE.md - Project Context & Rules for AI Agents

## 1. Project Overview

**Name:** OpenCoupon
**Description:** An open-source, full-stack browser extension framework (Honey clone) that automatically applies coupon codes at checkout.
**Architecture:** Monorepo with separate client and server packages.

- `client/`: React + Vite (Chrome Extension Manifest V3)
- `server/`: Node.js + Express (REST API)

## 2. Tech Stack & Versions

### Frontend (Client)

- **React:** 19.1.0
- **TypeScript:** 5.9.3 (hoisted from root)
- **Vite:** 7.0.5
- **Tailwind CSS:** 3.4.19
- **Chrome Extension:** Manifest V3 with @crxjs/vite-plugin 2.0.3
- **Testing:** Vitest 4.0.16, @testing-library/react 16.3.1
- **Linting:** ESLint 9.39.2, Prettier 3.7.4
- **Hooks:** Husky 9.1.7 (pre-commit)

### Backend (Server)

- **Node.js:** 20+
- **Express:** 5.2.1
- **TypeScript:** 5.9.3
- **Prisma ORM:** 7.1.0
- **PostgreSQL:** 15 (Dockerized)
- **Validation:** Zod 4.2.1
- **Testing:** Jest 30.2.0, ts-jest, Supertest 7.1.4
- **Rate Limiting:** express-rate-limit 8.2.1
- **Linting:** ESLint 9.39.2, Prettier 3.7.4
- **Hooks:** Husky 9.1.7

### Infrastructure

- **Monorepo:** npm workspaces (root + client + server)
- **Database:** PostgreSQL 15-alpine (Docker)
- **Admin UI:** pgAdmin 4 (Docker, port 5050)
- **Package Manager:** npm 10+ with workspaces

## 3. Common Commands

### Root Level

| Action                  | Command                 |
| :---------------------- | :---------------------- |
| **Install All**         | `npm install`           |
| **Dev Mode (Both)**     | `npm run dev`           |
| **Dev Client Only**     | `npm run dev:client`    |
| **Dev Server Only**     | `npm run dev:server`    |
| **Build All**           | `npm run build`         |
| **Build Client**        | `npm run build:client`  |
| **Build Server**        | `npm run build:server`  |
| **Test All**            | `npm run test`          |
| **Test Client**         | `npm run test:client`   |
| **Test Server**         | `npm run test:server`   |
| **Test Coverage (All)** | `npm run test:coverage` |
| **Lint All**            | `npm run lint`          |
| **Lint Fix**            | `npm run lint:fix`      |
| **Format All**          | `npm run format`        |
| **Format Check**        | `npm run format:check`  |
| **Clean All**           | `npm run clean`         |
| **Start DB**            | `npm run db:up`         |
| **Stop DB**             | `npm run db:down`       |
| **DB Logs**             | `npm run db:logs`       |
| **DB Migrate**          | `npm run db:migrate`    |
| **DB Seed**             | `npm run db:seed`       |
| **DB Studio**           | `npm run db:studio`     |

### Client (`cd client` or `npm -w client`)

| Action              | Command                 |
| :------------------ | :---------------------- |
| **Dev Mode**        | `npm run dev`           |
| **Build Extension** | `npm run build`         |
| **Run Tests**       | `npm run test`          |
| **Test UI**         | `npm run test:ui`       |
| **Test Coverage**   | `npm run test:coverage` |
| **Lint**            | `npm run lint`          |
| **Preview Build**   | `npm run preview`       |
| **Clean**           | `npm run clean`         |

### Server (`cd server` or `npm -w server`)

| Action                | Command                    |
| :-------------------- | :------------------------- |
| **Dev Mode**          | `npm run dev`              |
| **Build**             | `npm run build`            |
| **Start**             | `npm run start`            |
| **Run Tests**         | `npm run test`             |
| **Test Watch**        | `npm run test:watch`       |
| **Unit Tests Only**   | `npm run test:unit`        |
| **Integration Tests** | `npm run test:integration` |
| **Test Coverage**     | `npm run test:coverage`    |
| **Lint**              | `npm run lint`             |
| **Lint Fix**          | `npm run lint:fix`         |
| **Migrate**           | `npm run migrate`          |
| **Seed DB**           | `npm run seed`             |
| **Clean**             | `npm run clean`            |

## 4. Coding Standards

### TypeScript Rules

- **Strict Mode:** Enabled in both client and server
- **noImplicitAny:** true
- **Interfaces:** Prefer `interface` over `type` for object definitions
- **Shared Types:** Define in `client/src/types/index.ts` or ensure Frontend imports from API response definitions
- **Explicit Returns:** All functions must have explicit return types
- **Path Aliases:** Client uses `@/*` → `src/*`
- **Additional Strictness (Server):** `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`

### Extension Specifics (Frontend)

- **Manifest V3:** Strictly adhere to MV3 limitations (no remote code execution)
- **Minimal Permissions:** Only `sidePanel`, `activeTab`, `storage`, `alarms`
- **DOM Safety:** When writing content scripts, verify elements exist before manipulating. Use optional chaining `?.` and `isValidDOMElement()` utility
- **Message Validation:** Use `isValidMessageSender()` for Chrome runtime message validation
- **State:** Use `chrome.storage.local` for persisting user preferences, NOT `localStorage`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EvgeniiKlepilin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
