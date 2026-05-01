---
trigger: always_on
description: This document is a quick guide for any contributors or AI agents that touch the `koborin.ai` repository.
---

# Agents Guide

This document is a quick guide for any contributors or AI agents that touch the `koborin.ai` repository.

## Language Policy

- **Conversation**: Always communicate with the user in **Japanese**.
- **Code**: Write all code, comments, variable names, and commit messages in **English**.

## Mission

- Personal site + technical garden for `koborin.ai`.
- Astro with Starlight (documentation-focused theme) for MDX content under `app/src/content/docs/`.
- Google Cloud Run (dev / prod) fronted by a single global HTTPS load balancer.
- Infrastructure managed via Pulumi (Go) with three stacks: `shared`, `dev`, `prod`.
- CI/CD and Pulumi preview/up executed only through GitHub Actions using Workload Identity Federation.

## Repository Layout

| Path | Purpose |
| --- | --- |
| `.gcloudignore` | Excludes files from Cloud Build upload (infra/, docs/, etc.). |
| `app/` | Astro + Starlight app (TypeScript, MDX, Vitest). |
| `app/cloudbuild.yaml` | Cloud Build configuration for Docker build from project root. |
| `app/src/content/docs/` | MDX documentation pages. Mark drafts with `draft: true` in frontmatter. |
| `app/src/content/config.ts` | Content Collections schema (uses Starlight's `docsSchema`). |
| `app/src/utils/llms.ts` | Shared logic for llms.txt generation. |
| `app/src/pages/llms*.txt.ts` | Astro endpoints that generate llms.txt files at build time. |
| `app/src/pages/rss.xml.ts` | RSS feed endpoint for English articles. |
| `app/src/pages/ja/rss.xml.ts` | RSS feed endpoint for Japanese articles. |
| `app/nginx/nginx.conf` | nginx configuration for static file serving (port 8080). |
| `infra/` | Pulumi Go stacks (`shared`, `dev`, `prod`). |
| `infra/stacks/shared.go` | Shared resources: APIs, Artifact Registry, HTTPS LB, Workload Identity. |
| `infra/stacks/dev.go` | Dev Cloud Run service. |
| `infra/stacks/prod.go` | Prod Cloud Run service. |
| `docs/` | Specifications, e.g. contact flow, o11y notes. |
| `docs/assets/{article}/` | Mermaid sources and generated images for each spec document. |
| `.claude-plugin/` | Plugin marketplace manifest (`marketplace.json`). |
| `plugins/` | Published Claude Code plugins (each with `plugin.json` + `skills/`). |
| `.github/workflows/` | CI/CD definitions. |

## Infrastructure Rules

1. **Preview/Up**: never run Pulumi applies locally. All infra changes go through GitHub Actions with Workload Identity Federation.
2. **State backend**: GCS bucket with Pulumi's automatic stack-based state management. Backend URL: `gs://<BUCKET_NAME>/pulumi`.
3. **Stacks**: Pulumi stacks (`shared`, `dev`, `prod`) are managed via `pulumi stack select`. Each stack has its own state file.
4. **Environments**:

   - `shared`: APIs, Artifact Registry, static IP, Managed SSL cert, HTTPS LB (NEG, Backend Service, URL Map, Target Proxy, Forwarding Rule), IAP configuration for dev, Workload Identity for GitHub Actions.
   - `dev`: Cloud Run service `koborin-ai-web-dev`.
   - `prod`: Cloud Run service `koborin-ai-web-prod`.

5. **Architecture Design**:

   - `shared` stack creates the entire HTTPS load balancer including Serverless NEGs and Backend Services for both dev/prod.
   - NEGs reference Cloud Run services by name (string), so Cloud Run services can be created later in dev/prod stacks without circular dependencies.
   - Dev Backend Service has IAP enabled with allowlist, prod has no IAP.
   - Dev Backend Service adds `X-Robots-Tag: noindex, nofollow` response header.

6. **Configuration Management**:

   - All configuration values are set dynamically via `pulumi config set` in GitHub Actions.
   - Secrets (OAuth credentials, passphrase) are stored in GitHub Secrets and passed at runtime.
   - Stack configuration files (`Pulumi.*.yaml`) are gitignored - CI/CD sets all values.

7. **IaC Philosophy - Code as Documentation**:

   - IaC differs fundamentally from application code: **the code itself is the design document**.
   - Prioritize readability and explicitness over abstraction.
   - A reviewer should be able to understand the entire infrastructure by reading the stack files alone.
   - Only extract to configuration when values genuinely vary across environments or need to be injected at runtime.

8. **File Organization**:

   - `infra/main.go`: Entry point that loads the appropriate stack based on stack name.
   - `infra/config.go`: Configuration helper functions.
   - `infra/stacks/*.go`: Stack definitions (shared, dev, prod).
   - `infra/Pulumi.yaml`: Project configuration.
   - `infra/go.mod` / `infra/go.sum`: Go module dependencies.

## Application Rules

1. **MDX workflow**:
   - Author pages under `app/src/content/docs/`. Use YAML frontmatter with `title`, `description`.
   - Mark drafts with `draft: true` in frontmatter to exclude from navigation.
   - Starlight automatically generates navigation from the directory structure and sidebar config in `astro.config.mjs`.
2. **Adding new content**:
   - Create `.mdx` file under `app/src/content/docs/` (or subdirectory for categories like `blog/`, `guides/`).
   - Add frontmatter: `title` (required), `description` (required), `publishedAt` (required for articles, `YYYY-MM-DD`), `draft` (optional, boolean).
   - Update `app/src/sidebar.ts` to add navigation entry:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koborin-ai/site](https://github.com/koborin-ai/site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
