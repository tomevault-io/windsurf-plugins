---
trigger: always_on
description: > **This file governs V88 solutions built with Daia (Deployed AI Architecture)** — V88's solution template, distributed via the `v88-ai-solution-template` GitHub repo. Daia ships a runnable starter app — auth, user management, avatar menu, notifications, glass design system, PWA manifest. Your behaviour depends on whether you're **maintaining Daia itself** or **building an app on top of it** — Step 0 below tells you which.
---

# CLAUDE.md — V88 Project

> **This file governs V88 solutions built with Daia (Deployed AI Architecture)** — V88's solution template, distributed via the `v88-ai-solution-template` GitHub repo. Daia ships a runnable starter app — auth, user management, avatar menu, notifications, glass design system, PWA manifest. Your behaviour depends on whether you're **maintaining Daia itself** or **building an app on top of it** — Step 0 below tells you which.
>
> In APP-BUILD mode, your job is to:
>
> 1. Detect the current state of the project (state detection below)
> 2. Either run starter setup, wait for scope, or build the app — exactly one of those
> 3. Never duplicate what the starter already gives you

---

## Pre-Flight Checks

> **STOP. Before generating any code, run Step 0 to determine which mode you're in. Everything that follows depends on it.**

### Step 0 — detect mode (every session, before anything else)

Run **both** of these. Use the strongest signal — either one matching is enough.

```bash
basename "$(pwd)"
basename -s .git "$(git remote get-url origin 2>/dev/null)"
```

| Result | Mode |
|---|---|
| Either output is exactly `v88-ai-solution-template` | **TEMPLATE-MAINTENANCE MODE** |
| Anything else | **APP-BUILD MODE** |

#### If TEMPLATE-MAINTENANCE MODE

- **Do NOT continue with the state-detection checks below.** They are app-build only.
- **Read `TEMPLATE.md` IMMEDIATELY — before any other action.** It contains the **Template Change Protocol** that governs every modification to this repo. Do not write any code, edit any file, or propose any change until you have classified the request per the protocol's four-category framework and confirmed scope with the user.
- Every change to `src/`, `supabase/`, `skills/`, `public/`, `scripts/`, `CLAUDE.md`, `TEMPLATE.md`, or `README.md` follows the protocol — there are no exceptions for "small" changes (small changes are exactly where drift creeps in).
- The skill-editing prohibition in the Boundaries section below does **not** apply to you — you ARE the maintainer.

#### If APP-BUILD MODE

- Continue to "State detection" below.
- The Boundaries section applies fully — never edit `SKILL.md` files, never make ad-hoc schema changes, etc.

---

### State detection (APP-BUILD mode only)

Use three signals to place the project on the matrix:

**1. Supabase schema state** — via Supabase MCP, call `list_tables` on the `public` schema.
- No `profiles` table → **schema is empty** (starter not yet applied)
- `profiles` + `notifications` present → **starter applied**

**2. Scope state** — read `V88-SCOPE.md`.
- Any section still contains `[TODO]` → **scope not populated**
- All sections filled (app name, brand colour, entities, features) → **scope populated**

**3. App icon state** — check whether `v88_context/app-icon.png` exists.
- File present → **icon ready**
- File missing → **icon missing** (still needed before rebrand can run)

### Four states, four paths

| Schema | Scope | Icon | What to do |
|--------|-------|------|------------|
| Empty | Any | Any | **Run starter setup.** Do NOT read `V88-SCOPE.md` yet. Invoke the `starter-setup` skill. |
| Applied | `[TODO]` present | Any | **Wait for scope.** Starter works — tell the user to log in, try it, then fill in `V88-SCOPE.md`. |
| Applied | Populated | Missing | **Wait for icon.** Tell the user: *"Drop a square PNG (≥ 128 px, recommended ≥ 512 px) at `v88_context/app-icon.png` so I can generate every platform variant during rebrand."* |
| Applied | Populated | Present | **Run app pre-flight**, confirm with user, rebrand the starter, then build. See "App pre-flight" below. |

### Starter setup (schema empty)

Invoke the `starter-setup` skill and follow it exactly. Summary of what it does:

1. Confirm the Supabase project URL with the user
2. `npm install`
3. Write `.env.local` with URL + anon key (retrieved via MCP)
4. Apply all three migrations in order
5. Deploy all four edge functions
6. Invoke `bootstrap-admin` to seed `admin@v88.co.uk / Admin!123` with welcome notification
7. Start `npm run dev` in background, tell the user to log in at `http://localhost:5173`

(See `skills/starter-setup/SKILL.md` for the canonical, numbered step list — including the VAPID + `app_secrets` seed needed before push works.)

After setup, tell the user:

> "Setup complete. Log in, try it, then fill in `V88-SCOPE.md` and come back to build your app."

Do not proceed to app code until scope is populated.

### Starter applied, scope has `[TODO]`

Don't build anything. Say something like:

> "The starter is already set up. Log in at http://localhost:5173 with `admin@v88.co.uk / Admin!123`, poke around (Users, Notifications, Profile), then fill in `V88-SCOPE.md` with your app name, brand colour, entities, and features. Come back when that's done."

### App pre-flight (starter applied, scope populated)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jens-v88/kosmos0](https://github.com/jens-v88/kosmos0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
