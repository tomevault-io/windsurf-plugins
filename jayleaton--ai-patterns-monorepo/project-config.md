---
trigger: always_on
description: This is the **Better-Stack monorepo template**: a production-ready starting point for full-stack apps built with Next.js 16, Better Auth, Drizzle ORM, and Tailwind CSS.
---

# AGENTS.md - Instructions for AI Assistants

## Project Overview

This is the **Better-Stack monorepo template**: a production-ready starting point for full-stack apps built with Next.js 16, Better Auth, Drizzle ORM, and Tailwind CSS.

Guiding principle: we create modern, easy-to-use templates for better DX when creating new software projects efficiently.

When a developer is first defining what they are building, use the project skill `getting-started` (`.claude/skills/getting-started`, `.grok/skills/getting-started`, or `.agents/skills/getting-started` for Codex) instead of improvising a setup interview.

## What makes this template good

People clone or copy this repository to start real projects, then immediately diverge from it. It's important we maintain the things that make it a good starting point as we continue to iterate. Here's a brief list of the things we can never compromise on.

1. **Template first**

This repo is a template, not a canonical upstream. There is no downstream to contribute back to and no fleet of running instances to keep in sync. Every change should keep the template generic, minimal, and free of project-specific cruft. The example flows (like the users API) are documentation-as-code: keep them canonical and small. If a change only makes sense for one downstream project, it does not belong here.

2. **Performance without compromise**

Lots of templates have gotten bogged down with bad tech decisions and "slop". Do not add to that. Default to Server Components, keep client JavaScript to what interaction actually demands, keep data fetching on the server, and be considerate of render and bundle cost in every change.

3. **Network-ready dev servers**

Dev servers here are reachable beyond localhost on purpose. The web app binds `0.0.0.0:3000`, so it works identically at `http://localhost:3000`, on the local network, and over Tailscale at `http://<machine-hostname>:3000`. Never bake `localhost` into code, auth origins, or callbacks — that silently breaks every non-local client. New dev tooling and features must work over the network, not just from the machine that started them.

4. **Monorepo-aware**

One workspace is never the whole story. `apps/web-app`, `apps/mobile-app`, `packages/database`, and `packages/common` move together: a schema change ripples through repositories, services, validators, and UI. Every change gets a decision per workspace, even if the decision is "no change needed here".

The rest of this document is meant to help you navigate the codebase and make changes effectively. Think of these instructions less as "hard rules", more as "good defaults". The developer's preferences should be able to override anything here.

## Shared terminology

We need to be on the same page with terminology. When communicating, use this language:

- **you** means the agent reading this file and changing this template.
- **we, us, and maintainers** mean the people building this template. These are who you are talking to now.
- **user** or **developer** means the person who cloned this template and directs coding agents.
- **app** means a product surface: the Next.js website in `apps/web-app`, or the Expo mobile app in `apps/mobile-app`. Say **web app** or **mobile app** when you need to be specific.
- **packages** mean the shared workspaces: `packages/database` (schema + repositories) and `packages/common` (shared config and types).
- **schema** means a Drizzle table definition in `packages/database/src/schemas.ts`.
- **repository** means the data-access layer over Drizzle, in `packages/database/src/repositories/`.
- **service** means business logic, in `apps/web-app/lib/services/`.
- **validator** means a Zod schema in `apps/web-app/lib/validators/`.
- **server action** means a function in `apps/web-app/actions/` — the only client-triggered path into the API layer.
- **routes config** means `AppRoutes`, `ApiRoutes`, and feature flags in `apps/web-app/lib/config/featureToggles.ts`.

## The three ways to hurt yourself

**Killing by pattern.** Never `pkill -f`, `pgrep | kill`, or kill a PID you found by matching a name, path, or port string. This machine runs several dev servers and Node processes at once, and your own process can match the pattern. Kill only a PID you captured at spawn, or a process you have positively confirmed owns your port.

**Trusting the wrong database.** The Docker Postgres (`localhost:5432`) is the sandbox: seed it, reset it, break it. A remote database (Neon, Supabase, production) is never a playground — never point dev at one, run migrations against one, or copy real user data in or out unless explicitly asked. `docker compose down -v` destroys the local volume; be sure that is intended before you run it.

**Baking in origins.** Never hardcode `http://localhost:3000` in code, auth config, emails, or OAuth callbacks. Dev is reached over localhost, LAN IPs, and Tailscale hostnames; origins must come from `NEXT_PUBLIC_URL` and environment config, or every non-local browser silently breaks.

## Hit every layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jayleaton/ai-patterns-monorepo](https://github.com/jayleaton/ai-patterns-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
