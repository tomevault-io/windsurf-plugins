---
trigger: always_on
description: A social network that's only open a few hours per day (currently 20:00-22:00 CET, configurable via env vars). Users post memes (image + caption), text-only posts, links with OG previews, or audio posts (in-app recorded, max 10s, caption required). They upvote posts and comments, and a daily leaderboard crowns the funniest person. Content is cleaned up daily. Every day starts fresh. The Hall of Fame preserves the best post of each day with its top comments.
---

# twohrs — Zeitbegrenztes Soziales Netzwerk

## Project Overview

A social network that's only open a few hours per day (currently 20:00-22:00 CET, configurable via env vars). Users post memes (image + caption), text-only posts, links with OG previews, or audio posts (in-app recorded, max 10s, caption required). They upvote posts and comments, and a daily leaderboard crowns the funniest person. Content is cleaned up daily. Every day starts fresh. The Hall of Fame preserves the best post of each day with its top comments.

## Tech Stack

- **Runtime:** Next.js 15 (App Router) · TypeScript (strict) · Node v25
- **Styling:** Tailwind CSS 4
- **Backend:** Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- **Hosting:** Vercel (2 environments: production + preview)
- **Source Control:** GitHub ([loris307/twohrs](https://github.com/loris307/twohrs)) — public, open-source
- **CI:** GitHub Actions — 6 workflows in `.github/workflows/` (CI, CodeQL, preview smoke, Supabase schema lint, dependency review, Lighthouse)
- **Package manager:** pnpm (never npm or yarn)

## Commands

```bash
pnpm install          # install dependencies
pnpm dev              # dev server on http://localhost:3000
pnpm build            # production build (NEVER while dev server is running!)
pnpm lint             # ESLint
pnpm tsc --noEmit     # type-check without emitting (safe during dev)
pnpm typecheck        # full type-check (next typegen + tsc --noEmit)
```

Optional local-only launcher (kept ignored, not committed):
```bash
./plans/localhost/start.sh    # start localhost with the local helper
./plans/localhost/restore.sh  # manual restore if the helper was interrupted
```
The local helper is intentionally kept outside Git and may temporarily force the shared app window open while it runs.

Deploy:
```bash
vercel --yes && vercel alias <url> socialnetwork-dev.vercel.app  # preview → dev alias
vercel promote <deployment-id-or-url> --yes                      # promote tested preview → production aliases
vercel rollback <deployment-id-or-url> --yes                     # rollback production quickly if needed
```

Release workflow:
```bash
# 1) deploy preview from current branch
vercel --yes
vercel alias <preview-url> socialnetwork-dev.vercel.app

# 2) test on preview

# 3) promote the tested preview release to production
vercel promote <preview-deployment-id-or-url> --yes
```

Database (Supabase CLI):
```bash
supabase login                           # one-time: authenticate CLI with Supabase
supabase link --project-ref <ref>        # one-time: link project (prompts for DB password)
supabase migration list                  # show local vs remote migration status
supabase db push                         # push pending migrations to remote DB
supabase migration repair --status applied <version>  # mark migration as already applied
```

## Boundaries

### Always (do without asking)
- Use **"twohrs"** as the brand name everywhere (lowercase, one word). Never use "2Hours", "2 Hours", or "2hours".
- Use `pnpm`, never `npm` or `yarn`
- Release via preview first, then `vercel promote` the tested deployment to production
- Check `isAppOpen()` in every new server action before writes
- Check auth (`supabase.auth.getUser()`) in every new server action
- Validate input with Zod schemas from `validations.ts`
- Check nullable `image_url`/`image_path` before `.toLowerCase()` or `<Image>`
- Return `ActionResult<T>` from all server actions
- Use `catch {}` not `catch (error) {}` when error var is unused
- Use `cn()` from `@/lib/utils/cn` for conditional Tailwind classes
- Document new features in `ENTWICKLUNG.md` (and big ones in `CLAUDE.md`)
- Dispatch `new Event("navigation-start")` before `router.push()` calls
- **All UI text must be lowercase** — global `text-transform: lowercase` is on `<body>`. CSS handles it. Only legal pages use `normal-case`. Never add `uppercase` or `capitalize` classes.

### Ask First (get approval)
- Adding new dependencies
- Database schema changes (new migrations)
- Changing RLS policies
- Modifying the time-gating logic (any of the 4 layers)
- Changes to cron jobs / cleanup order
- Changing public API signatures
- Modifying deployment configuration

### Never (hard stops)
- Run `pnpm build` while dev server is running (corrupts Turbopack cache)
- Commit `.env` files, secrets, or API keys
- Edit `pnpm-lock.yaml` manually
- Modify `supabase/functions/` with Node-style imports (Deno runtime)
- Include `supabase/functions/` in tsconfig
- Bypass time-gate enforcement without explicit approval
- Delete persistent tables (`profiles`, `follows`, `daily_leaderboard`, etc.)
- Force push to main
- Do not use `vercel --yes --prod` for normal releases. Always promote a tested preview deployment instead.
- Connect Vercel to GitHub (deploys are manual via CLI)
- Push secrets, Supabase URLs/keys, or credentials to the public repo


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loris307/twohrs](https://github.com/loris307/twohrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
