---
trigger: always_on
description: <!-- test: verify Codex GitHub Actions workflows -->
---

# coJournalist

<!-- test: verify Codex GitHub Actions workflows -->

## Deployment Workflow - MANDATORY

**NEVER push directly to `main`.** All changes go through a branch + PR. This is not optional.

1. **Check for worktrees first:**
   ```bash
   git worktree list
   ```
   If a worktree exists for the current task, work there.

2. **Create a branch** from `main` (or use `develop`):
   ```bash
   git checkout -b my-feature main
   ```

3. **Push the branch and open a PR to `main`:**
   ```bash
   git push -u origin my-feature
   gh pr create --title "..." --body "..."
   ```

4. **Wait for CI to pass** — 4 required checks must be green:
   - `build-frontend` — SvelteKit build
   - `test-frontend` — Vitest suite
   - `test-backend` — pytest unit tests
   - `lint` — svelte-check

5. **Merge the PR** — Render auto-deploys backend from `main`.

**Why:** Pushing to `main` triggers a Render deploy immediately with no safety net. The PR flow ensures CI passes and Codex reviews the code before anything reaches production.

---

## Node Version - IMPORTANT

**This project requires Node 22 LTS.** The Dockerfile and `frontend/.nvmrc` are pinned to Node 22. Using a different major version (especially Node 25+ with npm 11) will generate an incompatible `package-lock.json` that breaks the Render build (`npm ci` fails). Always run `nvm use` in `frontend/` before `npm install`.

---

## Frontend ENV VAR TRAP — read this before changing any `PUBLIC_*` or `VITE_*` config

The Vite/SvelteKit/Render env-var pipeline bit us **5+ times** during the v2 cutover (2026-04-21). The pattern keeps recurring because three layers each have different rules and they don't compose intuitively. **Stop and read this section before touching `Dockerfile`, `frontend/.env.production`, or Render env vars.**

### The three layers (in build-time order)

1. **`frontend/.env.production`** (committed): Vite reads this AT BUILD TIME for `import.meta.env.PUBLIC_*` and `import.meta.env.VITE_*` substitution. Source of truth for client-side public values.
2. **Dockerfile `ENV` directives**: process.env during `RUN npm run build`. **Overrides .env.production** because Vite's load order puts process.env first. Hardcoded `ENV X=value` is opaque to Render — Render env vars can't change it.
3. **Render service envVars**: become process.env at runtime. **Only reach the docker BUILD context if (a) declared as `ARG` in Dockerfile AND (b) Render decides to forward — behavior varies by how the var was created (blueprint sync vs dashboard add vs API PUT).** Do not rely on this for build-time vars.

### Rules going forward

- **`PUBLIC_SUPABASE_URL`, `PUBLIC_SUPABASE_ANON_KEY`, `PUBLIC_DEPLOYMENT_TARGET`, `PUBLIC_MUCKROCK_ENABLED`, `VITE_API_URL`**: hardcoded `ENV` in `Dockerfile`. Single source of truth. Don't add to Render envVars (it's silently ignored). To change, edit Dockerfile + commit + redeploy.
- **`PUBLIC_MAPTILER_API_KEY`**: ARG in Dockerfile + Render envVar (it's a secret, has to be Render-side). Empty default means missing-key → degraded map UX, not a crash.
- **Backend secrets (`SUPABASE_SERVICE_KEY`, `DATABASE_URL`, etc.)**: Render envVars only. Read at runtime via `os.getenv`. Never in Dockerfile.
- **`.env.production`**: keep it as a redundant source of truth for local dev parity. Vite reads it; Dockerfile ENV overrides it in production builds.
- **Don't add `frontend/.env.*` globs to `.dockerignore`** — that masks `.env.production` from the build context. Use explicit `frontend/.env.local` etc.

### Symptoms that mean you fell into the trap

- `_app/env.js` shows a `PUBLIC_*` value as `""` while Render env vars list shows the correct value → Dockerfile didn't bake it.
- Browser bundle hits the wrong API host → check Dockerfile's `ENV VITE_API_URL=` literal first.
- "supabaseUrl is required" thrown by `supabase-js` → empty `PUBLIC_SUPABASE_URL` baked in.
- New env var added in Render dashboard, no effect on next deploy → it's a build-time var that needs Dockerfile too.

### Verifying a deploy picked up new build-time vars

```bash
# What got baked into the SPA
curl -s https://cojournalist.ai/_app/env.js
# What URL the api-client will call (look at any chunk + grep)
curl -s "https://cojournalist.ai/_app/immutable/chunks/$(curl -s https://cojournalist.ai/ | grep -oE 'chunks/[A-Za-z0-9_-]+\.js' | head -1)" | grep -oE 'https://[^"]+supabase\.co[^"]*' | head -2
```

### When to break the rule

If a build-time var legitimately needs to differ between SaaS and OSS deploys (e.g. project URL when forking), then add an `ARG` *with the SaaS default*, override via Render envVar (which DOES work for ARGs declared in Dockerfile), and document in the Dockerfile comment near the ARG. Maintenance escape hatch — exercise sparingly.

---

## Local MuckRock Auth — DO NOT REGRESS

The private repo has two distinct local auth goals and they are easy to mix up:

- **Daily pre-push SaaS testing:** sign in with a real MuckRock account on `http://localhost:5173`, return to localhost, and view **real hosted account data** before deploy.
- **Disposable demo testing:** local Supabase Auth with dummy/demo data only.

### The correct daily SaaS path

`cd frontend && npm run dev`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buriedsignals/cojournalist-os](https://github.com/buriedsignals/cojournalist-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
