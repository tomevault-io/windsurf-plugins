---
trigger: always_on
description: * **No Pull Requests:** Do not create or submit Pull Requests (PRs) under any circumstances.
---

# AGENTS.md — StudioCMS

# Agent Policies

## Prohibited Actions (All Agents)
* **No Pull Requests:** Do not create or submit Pull Requests (PRs) under any circumstances. 
* **Workflow:** Provide code changes as diffs or full file contents in the chat interface only.
* **Review Only:** You may provide code reviews or suggestions, but you are prohibited from initiating the merge process or opening new PRs.
* **Collaboration:** Collaborate with human developers by providing code snippets, explanations, and guidance, but do not take direct actions in the repository.
  * **Compliance:** Adhere strictly to these constraints to ensure a clear separation of responsibilities between AI agents and human developers.
  * **Human Only:** AI agents are not permitted to generate or modify full code in this repository. All code contributions must be made by human developers based on the guidance provided by AI agents. You may provide code snippets or suggestions, but the final implementation must be done by a human developer.

## Specific Agent Agent Policies
### @coderabbit / @coderabbitai
* Allowed to review, and perform programmed actions as allowed in the current config.

## Project Overview

**StudioCMS** is an MIT-licensed, open-source SSR Astro-native CMS built with TypeScript and Effect-ts. It serves as a headless CMS requiring custom frontend development and must run in server-side rendering mode.

- **Repository**: <https://github.com/withstudiocms/studiocms/>
- **Documentation**: <https://docs.studiocms.dev>
- **Website**: <https://studiocms.dev>
- **Community**: [Discord](https://chat.studiocms.dev)
- **Translations**: [Crowdin Project](https://crowdin.com/project/studiocms)
- **Status**: Early development, not production-ready

## Core Technology Stack

| Technology | Details | Notes |
|---|---|---|
| **Framework** | Astro | Must use SSR mode (`output: 'server'`), never SSG |
| **Language** | TypeScript | Strict typing enforced |
| **Database** | @withstudiocms/kysely | Powered by Kysely - supports libSQL, MySQL, PostgreSQL |
| **Effect System** | Effect-ts | Functional programming patterns |
| **Markdown** | MarkedJS, MarkDoc, native Astro MD | Extension support |
| **Auth** | Plugins (GitHub, Discord, Google, Auth0) + built-in username/password | OAuth requires plugin installation |
| **Package Manager** | pnpm 10.17.0 (preferred) | npm/yarn also supported |
| **Runtime** | Node.js >=22.20.0 only | Bun and Deno are NOT supported |

## Monorepo Structure

```text
packages/
├── studiocms/                    # Core CMS package (main)
├── @studiocms/blog/              # Blog plugin
├── @studiocms/github/            # GitHub OAuth plugin
├── @studiocms/discord/           # Discord OAuth plugin
├── @studiocms/google/            # Google OAuth plugin
├── @studiocms/auth0/             # Auth0 OAuth plugin
├── @studiocms/cloudinary-image-service/
├── @studiocms/s3-storage/
├── @studiocms/html/              # Content renderer plugins
├── @studiocms/markdoc/
├── @studiocms/md/
├── @studiocms/mdx/
├── @studiocms/wysiwyg/
├── @withstudiocms/sdk/           # SDK / internal packages
├── @withstudiocms/kysely/        # Kysely database layer
├── @withstudiocms/effect/
├── @withstudiocms/buildkit/
├── @withstudiocms/auth-kit/      # Authentication utilities
├── @withstudiocms/cli-kit/       # CLI toolkit
├── @withstudiocms/api-spec/      # API specification
├── @withstudiocms/component-registry/
├── @withstudiocms/config-utils/
├── @withstudiocms/internal_helpers/
├── @withstudiocms/template-lang/
└── ...other packages
```

## Dev Environment Setup

### Prerequisites
- Node.js >=22.20.0 (version specified in `.prototools`)
- pnpm 10.17.0 (preferred package manager)
- Database: libSQL (Turso, self-hosted, or local file), MySQL, or PostgreSQL

### Environment Variables
```bash
# Database (Required - Choose ONE option)

# Option 1: libSQL (Turso, local file, or self-hosted)
CMS_LIBSQL_URL=libsql://your-database.turso.io
CMS_LIBSQL_AUTH_TOKEN=your-database-token
# Optional:
# CMS_LIBSQL_SYNC_INTERVAL=
# CMS_LIBSQL_SYNC_URL=

# Option 2: MySQL
CMS_MYSQL_DATABASE=your-database-name
CMS_MYSQL_USER=your-database-user
CMS_MYSQL_PASSWORD=your-database-password
CMS_MYSQL_HOST=your-database-host
CMS_MYSQL_PORT=3306

# Option 3: PostgreSQL
CMS_POSTGRES_DATABASE=your-database-name
CMS_POSTGRES_USER=your-database-user
CMS_POSTGRES_PASSWORD=your-database-password
CMS_POSTGRES_HOST=your-database-host
CMS_POSTGRES_PORT=5432

# Base Authentication (Required)
CMS_ENCRYPTION_KEY="..." # Generate: openssl rand -base64 16

# GitHub OAuth (Optional — requires @studiocms/github plugin)
CMS_GITHUB_CLIENT_ID=
CMS_GITHUB_CLIENT_SECRET=
CMS_GITHUB_REDIRECT_URI=http://localhost:4321/studiocms_api/auth/github/callback

# Discord OAuth (Optional — requires @studiocms/discord plugin)
CMS_DISCORD_CLIENT_ID=
CMS_DISCORD_CLIENT_SECRET=
CMS_DISCORD_REDIRECT_URI=http://localhost:4321/studiocms_api/auth/discord/callback

# Google OAuth (Optional — requires @studiocms/google plugin)
CMS_GOOGLE_CLIENT_ID=
CMS_GOOGLE_CLIENT_SECRET=
CMS_GOOGLE_REDIRECT_URI=http://localhost:4321/studiocms_api/auth/google/callback

# Auth0 OAuth (Optional — requires @studiocms/auth0 plugin)
CMS_AUTH0_CLIENT_ID=
CMS_AUTH0_CLIENT_SECRET=

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [withstudiocms/studiocms](https://github.com/withstudiocms/studiocms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
