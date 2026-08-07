---
trigger: always_on
description: Altay Studio is an automated SaaS platform (the "Control Plane") that rapidly provisions, deploys, and manages isolated business websites. It acts as a website factory by creating an isolated schema in a master Supabase project, cloning a tailored GitHub repository (template), injecting environment variables (including VITE_DB_SCHEMA), and deploying to Vercel for every new client.
---

# Altay Studio AI Context & Governance

## Project Overview
Altay Studio is an automated SaaS platform (the "Control Plane") that rapidly provisions, deploys, and manages isolated business websites. It acts as a website factory by creating an isolated schema in a master Supabase project, cloning a tailored GitHub repository (template), injecting environment variables (including VITE_DB_SCHEMA), and deploying to Vercel for every new client.

---

## ⚠️ Critical: The Provisioning Flow (MUST READ)

Every time a client submits the "Launch My Site" form, the `provision-client` edge function executes these steps **in order**. Getting the order wrong has caused bugs multiple times — do not change it without reading this.

```
User submits form
      │
      ▼
1. AUTHENTICATE — verify the user's JWT via supabase.auth.getUser(token)
      │
      ▼
2. DB RECORD (INIT) — insert into public.businesses with owner_id, slug, schema_name,
                      provisioning_status = 'in_progress', provisioning_step = 'init'
      │
      ▼
3. DB SCHEMA — create a new Postgres schema (e.g. schema_ahmiclinic) in the
               master Supabase project and run the template's schema.sql into it
               (Update DB record: provisioning_step = 'db_schema')
      │
      ▼
4. GITHUB REPO — call POST /repos/{org}/{template-repo}/generate to clone the
                 template into a brand-new repo named {slug}-site
                 (e.g. aaltaay/template-barber → aaltaay/ahmiclinic-site)
                 ⚠️ Wait ~3 seconds after creation before the next step.
                 (Update DB record: github_repo, provisioning_step = 'github_repo')
      │
      ▼
4b. INJECT CONFIG — GET then PUT tenant.config.json into the new repo via
                    GitHub Contents API. Overwrites the template default with
                    tenant-specific colors, fonts, feature flags, and pages.
                    Non-fatal if it fails (env vars are the primary source).
      │
      ▼
5. VERCEL PROJECT — call POST /v10/projects?teamId={VERCEL_TEAM_ID} to create
                    a new Vercel project named {slug}, linking it to the NEW repo.
                    Inject all VITE_ env vars at this step.
                 (Update DB record: vercel_project_id, provisioning_step = 'vercel_deploy')
      │
      ▼
6. CLOUDFLARE DNS — DNS is handled automatically via a global wildcard CNAME 
                (*.altaystudio.com → cname.vercel-dns.com) on Cloudflare. 
                No per-client API calls are required for DNS.
                 (Update DB record: provisioning_step = 'done')
      │
      ▼
7. UPDATE DB — write vercel_deployment_url back onto the business record, 
               mark provisioning_status = "completed", provisioning_step = "done"
```

### Rollback Strategy
- If **any step fails** → Catch error, update DB record with `provisioning_status = 'failed'` and `provisioning_error = err.message`. This allows the Admin Dashboard to see the failure.
- External resources (like a partially created GitHub repo or Vercel project) should either be rolled back manually via the Admin Dashboard or via a robust retry mechanism. We currently keep the DB record as an error log.

### Template → Repo Name Convention
| Template repo      | Business type | New client repo name   |
|--------------------|---------------|------------------------|
| template-barber    | barber        | `{slug}-site`          |
| template-clinic    | clinic        | `{slug}-site`          |

Template is resolved from the `GITHUB_TEMPLATE_BARBER` / `GITHUB_TEMPLATE_CLINIC` secrets (NOT hardcoded). The `GITHUB_ORG` secret controls the GitHub org (default: `aaltaay`).

## The Admin Control Plane & Diagnostics

The platform includes an Admin Dashboard (`/admin`) for monitoring and debugging provisioning flows. It interfaces with two main Edge Functions:
1. `provision-client`: Handles the initial creation.
2. `admin-action`: A secure proxy function that takes an `action` and `business_id` to perform live diagnostics or cleanup.
   - **`diagnose`**: Connects to the database to check if the schema exists, pings GitHub to verify the repo exists, and hits Vercel API to check the project.
   - **`cleanup`**: Drops the schema (`DROP SCHEMA CASCADE`), deletes the GitHub repo, and deletes the Vercel project, allowing the admin to start over cleanly.

---

## Key Architectural Decisions & Gotchas
*   **Archival Architecture:** Adopted a "Safe Archive" philosophy. When an admin deletes a business record, the system relies on archiving resources instead of fully destroying them to prevent accidental data loss. The Edge Function renames the Postgres schema and the GitHub repository by prefixing them with `archived_` and a timestamp. Vercel projects are permanently deleted to free up domains, and the local business record is removed to clear the dashboard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaltaay/altay-studio-public](https://github.com/aaltaay/altay-studio-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
