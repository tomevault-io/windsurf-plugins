---
trigger: always_on
description: Turborepo monorepo with 5 packages:
---

# AI Budget Assistant

## Architecture

Turborepo monorepo with 5 packages:

| Package | Tech | Purpose |
|---|---|---|
| `apps/api` | NestJS 10 + Prisma 5 + PostgreSQL + Redis | REST API backend |
| `apps/mobile` | Expo 54 + React Native 0.81 + Zustand + SQLite/Drizzle | Mobile app (iOS/Android/Web) |
| `apps/admin` | Next.js 16 + React 19 + TailwindCSS 4 + shadcn/ui + Recharts | Admin web dashboard (port 3001) |
| `packages/shared-types` | TypeScript interfaces | Entities and DTOs shared between api and mobile |
| `packages/shared-utils` | Zod schemas + formatting + constants | Validation and utilities shared between api and mobile |

## Key Patterns

### API (NestJS)
- **Module structure**: `modules/<feature>/` contains `module.ts`, `controller.ts`, `service.ts`, `dto/index.ts`, `guards/`
- **Auth**: JWT via `@UseGuards(JwtAuthGuard)`. Request type: `AuthenticatedRequest` from `common/types/index.ts`. Password reset via 6-digit email code (`POST /auth/forgot-password`, `POST /auth/reset-password`) with in-memory rate limiting and bcrypt-hashed codes. Email change also via 6-digit code (`POST /auth/change-email/request`, `POST /auth/change-email/confirm`) — JWT-guarded, requires current password on request, issues new tokens on confirm. **Google sign-in (ABA-282)**: `POST /auth/google` (public) takes a Google **ID token** from the client (mobile + web via `expo-auth-session`, no native module), verified server-side by `GoogleTokenVerifier` (`google-auth-library`, audiences from `GOOGLE_OAUTH_CLIENT_IDS`, requires `email_verified`). Resolves by `googleId` → **auto-link by verified email** (sets `googleId`; rejects + does not link a deactivated account) → else creates a verified, passwordless user (name/email from Google, language from client, currency default USD) + default account. Returns the same shape as `/auth/login`. `User.googleId String? @unique` + `passwordHash` now **nullable** (migration `20260621000000_add_google_auth`); `login()` and `changeEmailRequest()` reject null-passwordHash accounts ("use Google sign-in"). Mobile: `authStore.googleLogin`, `useGoogleAuth` hook, "Continue with Google" on `(auth)/login.tsx`+`register.tsx`. **Web nonce (ABA-291)**: the web path uses the **generic** `expo-auth-session` `useAuthRequest` with `responseType: IdToken`, which (unlike `providers/Google`) does NOT add a `nonce` — Google's OpenID implicit flow requires one, so without it web sign-in failed with `Error 400: invalid_request / GeneralOAuthFlow` (native was fine — `signInNative()` builds its own URL with a manual nonce). Fix: `useGoogleAuth` generates a `nonce` once (`useMemo(() => rnd(), [])`) and passes `extraParams: { nonce }` to the web `useAuthRequest`. **Prerequisite**: Google Cloud OAuth client IDs (`GOOGLE_OAUTH_CLIENT_IDS` on API; `EXPO_PUBLIC_GOOGLE_WEB/IOS/ANDROID_CLIENT_ID` on client)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [micode-ai/ai-budget-assistant](https://github.com/micode-ai/ai-budget-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
