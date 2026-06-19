---
trigger: always_on
description: Guidance for AI coding assistants working in a codebase that uses Postgres
---

# pgrls for AI agents

Guidance for AI coding assistants working in a codebase that uses Postgres
Row-Level Security. Read this before suggesting RLS-related changes.

## What pgrls is

`pgrls` is a CLI linter for Postgres Row-Level Security. It connects to a live
database, introspects every table and policy, and reports problems by rule ID.
It is framework-agnostic — it does not care whether the project uses Supabase,
PostgREST, Hasura, Prisma, SQLAlchemy, Django, or raw SQL.

In the current release it ships **fifty-seven rules across four
categories**. Error: `SEC001` (missing RLS), `SEC002` (missing
`FORCE`), `SEC003` (permissive policies on `PUBLIC`), `SEC004`
(inverted auth checks — the Lovable CVE pattern), `SEC006`
(write-side policies missing `WITH CHECK`), `SEC032` (table has
policies but RLS is off — the policies are dormant and the table is
wide open), `SEC033` (policy scopes by a user-modifiable JWT claim
like `user_metadata` — the authenticated user can rewrite the value
the policy reads), `SEC036` (policy `EXISTS (SELECT FROM auth.users
WHERE …)` clause with no caller binding — evaluates to "is there
any admin at all" instead of "is THIS user an admin", so every
authenticated user passes once any matching row exists), `SEC038`
(semantic anonymous-read leak — the Z3-backed sibling of SEC004:
proves the USING predicate is unconditionally TRUE for an
unauthenticated session under Kleene 3VL, catching the NOT-wrapped
and cast-wrapped inverted-auth variants SEC004's syntactic match
misses; requires the optional `pgrls[diff-z3]` extra and NO-OPs
without it), `SEC039` (permissive write policy — INSERT/UPDATE/
DELETE/ALL — grants the unauthenticated `anon` role, so anonymous
PostgREST/Supabase clients can modify rows), `HYG001`
(policies referencing dropped columns), and `VIEW001`
(view bypasses RLS without `security_invoker`). Warning:
`SEC005` (policy expression has no own-column reference),
`SEC034` (policy gates on `auth.email()` — silent denial of
service to self when email changes, when SQL `=` is case-sensitive
but emails aren't, or when plus-addressing means `x+y@host` and
`x@host` compare unequal),
`SEC037` (policy compares `auth.role()` to an unknown role name —
silently denies every row because the equality never matches),
`SEC008` (permissive `USING (true)` — admits every row),
`SEC031` (restrictive `USING (true)` — a no-op floor that enforces
nothing), `SEC009` (RLS enabled but no policies —
silent deny-all), `SEC010` (`USING (false)` deny-all anti-pattern),
`SEC011` (`OR true` debug branch hidden inside a policy),
`SEC012` (table has only RESTRICTIVE policies — silent deny-all),
`SEC013` (trigger on RLS-protected table can bypass policies —
triggers fire as table owner),
`SEC014` (SECURITY DEFINER function bypasses caller's RLS —
audit every SECDEF function),
`SEC015` (SECURITY DEFINER function exposed to `pg_temp`
search-path shadowing),
`SEC016` (role with the `BYPASSRLS` attribute bypasses every
RLS policy),
`SEC017` (function with the `LEAKPROOF` attribute is evaluated
below the RLS barrier),
`SEC018` (policy compares a column against `current_user` /
`session_user` — no isolation under a shared pool role),
`SEC020` (policy `WITH CHECK` is constant `true` while `USING`
restricts — writes accept rows reads never would),
`SEC023` (policy applies to a role that bypasses RLS — the role's
`BYPASSRLS` attribute makes the policy's `TO` clause inert),
`SEC025` (policy predicate references a table that has RLS
disabled — the cross-table read is only as strong as the
referenced table's isolation),
`SEC026` (policy uses LIKE / ILIKE / SIMILAR TO / POSIX regex
against an auth-context value — a wildcard-shape GUC matches
every row),
`SEC028` (permissive write policy whose `WITH CHECK` is constant
`true` — accepts every write),
`SEC029` (role can `SET ROLE` to a `BYPASSRLS` role through
membership — an escalation path that disables every policy),
`SEC035` (UNIQUE constraint not scoped to the tenant discriminator —
a global `UNIQUE(email)` leaks cross-tenant existence via duplicate-key
errors; make it `UNIQUE(tenant_id, email)`),
`SEC040` (permissive `FOR ALL` policy whose `USING` scopes by a
tenant/owner key but whose explicit `WITH CHECK` binds no identity
column at all — a FOR ALL insert is governed by WITH CHECK alone, so a
caller can INSERT a row stamped with another tenant's id; bare FOR
UPDATE is excluded as Postgres re-checks the new row, and the "read
team, write own" asymmetry is not flagged),
`SEC041` (declarative partition child has RLS disabled while its
partitioned parent enforces it, and is granted directly to a non-owner
role — Postgres inherits neither RLS nor grants to partitions, so a query
naming the granted child directly bypasses the parent's policies; the
complement of SEC001, which cedes this case),
`SEC042` (a `SECURITY DEFINER` function whose owner bypasses RLS — a
superuser or `BYPASSRLS` role — is `EXECUTE`-able by `anon`/`PUBLIC`, so an
unauthenticated PostgREST `POST /rpc/fn` caller runs owner-privileged,
RLS-exempt code; function `EXECUTE` defaults to `PUBLIC`, so it fires even
with no explicit `GRANT`; the anon-exposure sharpening of SEC014),
`SEC043` (classic-`INHERITS` child has RLS disabled while an inheritance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgrls/pgrls](https://github.com/pgrls/pgrls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
