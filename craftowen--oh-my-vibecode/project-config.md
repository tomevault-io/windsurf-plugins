---
trigger: always_on
description: Instructions for AI agents (and humans) working in this repo. Read this file first; you should not need to explore the codebase to add a feature.
---

# AGENTS.md — oh-my-vibecode

Instructions for AI agents (and humans) working in this repo. Read this file first; you should not need to explore the codebase to add a feature.

Stack: React Router 8 (framework mode, SSR) on Cloudflare Workers · D1 + Drizzle ORM · Better Auth · Tailwind v4 · Vitest (workers pool) · bun.

## Commands

```bash
bun run setup             # one-time local provisioning: .dev.vars + local D1 migrations (no Cloudflare account needed)
bun dev                   # dev server at localhost:5173
bun run check             # typecheck && build && test — MUST pass before you claim any task is done
bun run typecheck         # react-router typegen && tsc
bun run test              # vitest run (runs in real Workers runtime; requires prior build)
bun run db:generate       # drizzle-kit generate (after editing app/db/schema.ts)
bun run db:migrate:local  # apply migrations to local D1
bun run deploy            # wrangler deploy (requires wrangler login; prod migrations via db:migrate:remote first)
```

## Structure

```
server.ts                 # Worker entry: request handler + RouterContextProvider(env, ctx)
app/routes.ts             # explicit route table — every new route is registered here
app/routes/               # home, login, signup, forgot-password, reset-password, verify-email,
                          #   logout, api.auth, api.theme, layout (protected shell), dashboard, settings
                          #   flat names; routing lives ONLY in app/routes.ts
app/components/ui/        # design-system primitives: button, input, label, card, badge, alert, skeleton
app/components/field.tsx  # Label + Input + inline error, already wired for screen readers
app/components/auth-shell.tsx # shared frame for signed-out pages
app/components/toast.tsx  # ToastProvider / useToast / useToastOnChange (aria-live, no dependency)
app/components/empty-state.tsx # what a list looks like when it is empty
app/components/local-time.tsx # hydration-safe timestamp rendering
app/app.css               # Tailwind v4 entry + design tokens (shadcn variable names) + dark mode
app/lib/app-context.ts    # cloudflareContext (env/ctx) + nonceContext (CSP)
app/lib/auth.server.ts    # buildAuth(env) — lazy per-env Better Auth instance
app/lib/auth-actions.server.ts # redirectWithSession / error-message helpers for auth actions
app/lib/email.server.ts   # sendEmail(env, message) — Resend over fetch, console fallback
app/lib/rate-limit.server.ts # D1-backed limiter for the kit's own auth actions
app/lib/validation.ts     # MIN_PASSWORD_LENGTH, field(form, name), FormErrors
app/lib/middleware.ts     # authMiddleware (RR8 middleware) + sessionContext
app/lib/theme.ts          # theme cookie read/write (dark mode)
app/lib/cn.ts             # class joiner (no clsx/tailwind-merge — see UI rules)
docs/recipes/             # guides for things left out of the core (email, OAuth, R2, AI, cron)
app/db/schema.ts          # app tables (Drizzle). auth-schema.ts is Better Auth's — never edit by hand
drizzle/                  # generated SQL migrations (committed)
tests/                    # vitest-pool-workers specs
```

## Core patterns (follow these exactly)

1. **Cloudflare bindings**: only available per-request. In loaders/actions: `const { env } = context.get(cloudflareContext)!;`. Never touch bindings at module scope.
2. **Auth**: `buildAuth(env)` (lazy, cached per env). Better Auth owns `/api/auth/*` — never add competing auth endpoints there.
   - Sign-in/sign-up/sign-out run **server-side in route actions** (`app/routes/login.tsx`, `signup.tsx`, `logout.tsx`) via `auth.api.signInEmail` / `signUpEmail` / `signOut` with `asResponse: true`, then `redirectWithSession(response, "/dashboard")` to carry the Set-Cookie headers onto the redirect. This is what makes auth work without JavaScript.
   - With `asResponse: true` a rejected credential comes back as a **non-OK Response, not a thrown error** — always check `response.ok` before redirecting. Keep the try/catch too: some failures (e.g. duplicate email) still throw.
   - The auth client (`app/lib/auth.client.ts`) is only for redirect-based social sign-in.
3. **Protected routes**: place them under the protected `layout` route. `authMiddleware` (RR8 middleware, runs once before all loaders in the subtree) redirects anonymous users to /login and stores the session; read it with `context.get(sessionContext)!` — never call `getSession` again in loaders under the protected layout.
4. **DB changes**: edit `app/db/schema.ts` → `bun run db:generate` → `bun run db:migrate:local` → commit generated files in `drizzle/`. Never write raw SQL migrations by hand. Tests discover migrations automatically (`tests/setup-db.ts` globs `drizzle/*.sql`) — nothing to register.
5. **Rate-limit anything that costs money or guesses secrets.** `limitAuthAttempt(env, request, "route-name", { window, max })` from `app/lib/rate-limit.server.ts`, returning `data({ errors }, { status: 429 })` when blocked. Better Auth's own limiter only covers `/api/auth/*`, never your actions.
6. **Email** goes through `sendEmail(env, message)`. It must never throw into a request path; it logs to the console when `RESEND_API_KEY` is unset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craftowen/oh-my-vibecode](https://github.com/craftowen/oh-my-vibecode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
