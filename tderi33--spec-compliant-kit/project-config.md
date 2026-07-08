---
trigger: always_on
description: How Supabase works under Lovable Cloud — migrations, grants, edge-function deploy, secrets, and the bundler. Read before writing ANY migration or edge function so it actually works once Lovable applies/deploys it.
---


# Supabase on Lovable Cloud — migrations, edge functions & secrets

This project's backend is a Supabase instance managed by **Lovable Cloud**. Migrations are written here (Cursor) but executed by Lovable's migration runner against the hosted DB. There is **no Supabase dashboard access**, no `SUPABASE_ACCESS_TOKEN`, no `SUPABASE_DB_PASSWORD`, and no local `supabase db push`. If a migration depends on dashboard-only state or missing privileges, it silently breaks the app at runtime (PostgREST returns permission errors and the frontend looks "empty").

Follow these rules so every migration you author here works the first time Lovable applies it.

## 1. The non-obvious rule: `public` schema has NO default Data API grants

Lovable Cloud's PostgREST does **not** inherit default privileges on `public` for `anon`, `authenticated`, or `service_role`. Enabling RLS is **not enough** — without an explicit `GRANT`, every request returns:

```
permission denied for table <name>
```

Every `CREATE TABLE public.<x>` migration MUST include `GRANT` statements **in the same migration**, in this exact order:

```sql
-- 1. CREATE
CREATE TABLE public.<table> (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  -- ... domain columns ...
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- 2. GRANT (REQUIRED — match the roles your policies allow)
GRANT SELECT, INSERT, UPDATE, DELETE ON public.<table> TO authenticated;
GRANT ALL ON public.<table> TO service_role;
-- GRANT SELECT ON public.<table> TO anon;   -- ONLY if an anon policy exists

-- 3. RLS
ALTER TABLE public.<table> ENABLE ROW LEVEL SECURITY;

-- 4. POLICIES
CREATE POLICY "..." ON public.<table> FOR SELECT TO authenticated USING (...);
```

Rules of thumb for the grant block:
- **Always** grant `service_role` for tables touched by edge functions, CRON jobs, or admin code (service_role bypasses RLS but still needs the table-level GRANT).
- **Drop `anon`** when every policy scopes to `auth.uid()`. This repo is authenticated-only except for the `feedback_*` guest-iframe path.
- Widen `anon` privileges only for fully public tables (none currently exist outside the `feedback-qr` storage bucket).

## 2. Functions — `EXECUTE` is also not automatic

For `SECURITY DEFINER` RPCs callable from the client (`supabase.rpc(...)` or `supabase.functions.invoke` patterns that hit PostgREST):

```sql
CREATE OR REPLACE FUNCTION public.my_rpc(...) RETURNS ...
LANGUAGE sql STABLE SECURITY DEFINER
SET search_path = public
AS $$ ... $$;

REVOKE ALL ON FUNCTION public.my_rpc(...) FROM PUBLIC, anon;
GRANT EXECUTE ON FUNCTION public.my_rpc(...) TO authenticated, service_role;
-- Only add `anon` if the function is part of a documented public surface
-- (in this repo: feedback_resolve_location, feedback_submit_response, feedback_iframe_completion).
```

Linter findings about "function callable by anon" are real — `REVOKE EXECUTE ... FROM anon` unless the function is intentionally public.

## 3. RLS-enabled tables with zero policies

A table with `ENABLE ROW LEVEL SECURITY` and no policies blocks all client access — that's often the intent for tables only touched by `service_role` edge functions (e.g. `livereach_nvr_token_cache`). The linter flags this as INFO. Declare intent explicitly:

```sql
CREATE POLICY "no client access" ON public.<table>
  FOR ALL TO anon, authenticated
  USING (false) WITH CHECK (false);
```

Service role still bypasses RLS, so edge functions keep working.

## 4. Storage buckets

Bucket policies live in `storage.objects` and follow the same GRANT model. A `public = true` bucket still serves files by direct URL anonymously — but if you also create a permissive `SELECT` policy granting `anon`, you enable **listing/enumeration** of every object. For QR-code-style buckets, scope the listing policy to `authenticated` only:

```sql
DROP POLICY IF EXISTS "Public can read X" ON storage.objects;
CREATE POLICY "Authenticated can list X" ON storage.objects
  FOR SELECT TO authenticated
  USING (bucket_id = 'X');
```

## 5. Roles & RBAC (project-specific)

- `app_role` enum is **2 values: `'admin', 'manager'`**. Do NOT add `attendant`, `employee`, or `user`. Floor attendants are not Supabase-auth users.
- Roles live in `public.user_roles` (never on `user_profiles`).
- RLS uses `SECURITY DEFINER` helpers `public.has_role(uid, role)` and `public.get_user_location(uid)` / `public.get_user_allowed_locations(uid)`. Reuse them — don't inline `EXISTS (SELECT 1 FROM user_roles ...)` in policies (causes recursion).
- Pattern: "admin sees all, manager sees own location."

## 6. Forbidden in migrations

- `ALTER DATABASE postgres ...` — rejected by Lovable's runner.
- Editing `auth`, `storage`, `realtime`, `supabase_functions`, `vault` schemas (except writing policies on `storage.objects`). No triggers on `auth.users` — use the existing `handle_new_user()` trigger pattern.
- Editing shipped migrations. Migrations are **forward-only**; write a new file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
