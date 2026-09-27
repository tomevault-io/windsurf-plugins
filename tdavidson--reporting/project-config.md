---
trigger: always_on
description: Conventions baked in to keep automated edits safe across this repo. Read before generating migrations or refactoring data-access code.
---

# Repo conventions for AI assistants

Conventions baked in to keep automated edits safe across this repo. Read before generating migrations or refactoring data-access code.

## Styling

`DESIGN.md` is the design system; `app/globals.css` holds the tokens. Read it before writing UI.

The short version, because these are the mistakes that actually get made:

- **Never use raw Tailwind palette classes** — `bg-amber-100`, `text-green-600`, `border-blue-500`. They break per-fund white-labelling, because `themeCssVars()` can't repoint them. Use the tokens: `bg-warning-subtle`, `text-success`, `border-brand-200`, `text-muted-foreground`. `lib/design-tokens.test.ts` fails on new ones; the only exemptions are files using colour *categorically*, allowlisted there with a reason.
- **Numbers use `tabular-nums`, not `font-mono`.** Mono is for content a machine reads literally — code, IDs, OTP inputs. Financial figures are not code. Same rule in the PDF templates.
- **Cards use `rounded-card`**, controls use `rounded-lg`/`md`/`sm`. `--radius` is the control radius (0.25rem), `--radius-card` the card one (0.5rem).
- **`--primary` is the deployment's action colour** (fund-themeable). **`--brand` is Hemrock's** (highlighter yellow, marketing — a fill under ink text, never text itself; accent text is `text-brand-700 dark:text-brand-400`). They are not interchangeable.
- **Accent text needs a dark-mode pair**: `text-brand-700 dark:text-brand-400`. The 700 stop fails contrast on the dark surface.
- **Display weight follows size and face.** Marketing headings (`text-display`/`text-title`) are `font-semibold`; LP-facing document headings (`text-heading`) stay `font-normal`. Both numbers assume `--font-display` is Inter — a serif display face wants 400 throughout.

## Migration conventions

### Every new `create table` migration requires explicit Data API grants

Supabase is moving to an "explicit grants required" model for the Data API. New Supabase projects after 2026-05-30, and all existing projects after 2026-10-30, will create tables in the `public` schema **without** automatic grants to `anon`/`authenticated`/`service_role`. The Data API (supabase-js, PostgREST, GraphQL) can't see those tables until grants are issued explicitly.

This repo is meant to be installable against fresh Supabase projects, so every `create table` migration must include the grants inline. The bulk-backfill migration (`20260513000000_data_api_grants_backfill.sql`) covers tables created before that date — but anything new must carry its own grants or the app breaks on fresh post-2026-05-30 installs.

**Template for any new table:**

```sql
create table public.new_thing (
  id uuid primary key default gen_random_uuid(),
  fund_id uuid not null references funds(id) on delete cascade,
  -- ... columns ...
  created_at timestamptz not null default now()
);

-- 1. Grants — required from 2026-05-30 onward for the Data API to see this table.
--    Default posture: anon = SELECT only (no unauthenticated writes via Data API);
--    authenticated + service_role get full CRUD, with RLS scoping per-row access.
--    Only grant anon writes if the table genuinely needs unauthenticated insert/update.
grant select on public.new_thing to anon;
grant select, insert, update, delete on public.new_thing to authenticated, service_role;

-- 2. RLS — enable even if you think it isn't needed. The schema-wide default is "RLS on".
alter table public.new_thing enable row level security;

-- 3. Policies — at least one per role that should have access. Without policies, RLS
--    blocks every row even when grants are in place.
create policy "Fund members read their fund's rows"
  on public.new_thing for select to authenticated
  using (exists (
    select 1 from fund_members fm
    where fm.fund_id = new_thing.fund_id and fm.user_id = auth.uid()
  ));
-- (add insert/update/delete policies as the table requires)
```

**The template above is for an ordinary table, and "ordinary" is the part to check.** A grant knows
nothing about domains or features, so `grant ... to authenticated` on a table whose route is gated
on a domain hands the data to exactly the members that gate refuses — from the browser console,
with `NEXT_PUBLIC_SUPABASE_ANON_KEY`, no route involved. If a table's only reader is a gated route
holding the service-role key (all of `lp_tax_forms`, `k1_*`, `tax_year_closes`, `received_k1s`),
grant it to `service_role` alone, reads included, and write no `authenticated` policies — RLS then
denies by default if a grant ever creeps back. `tests/tax-data-api-grants.test.ts` pins that set;
`20260714000004_ledger_db_enforcement.sql` is the write-side precedent for the ledger.

**Sequences:** this repo uses `uuid default gen_random_uuid()` for primary keys, so explicit sequences are rare. If you ever add one (`bigserial`, `serial`, `create sequence`), add `grant usage, select on sequence public.<name> to anon, authenticated, service_role;` alongside it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tdavidson/reporting](https://github.com/tdavidson/reporting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
