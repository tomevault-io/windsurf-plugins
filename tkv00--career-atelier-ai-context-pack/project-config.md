---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# AGENTS.md

Instructions for AI coding agents working in this repository.

This file follows the [AGENTS.md](https://agents.md) convention and is read by
Codex, Gemini CLI, Cursor, Copilot's coding agent, Zed, Aider, and others.

<br>

## What this project is

Career Atelier is a self-hosted job-application workspace. Seven AI agents run
on the user's **own** ChatGPT, Claude, and Gemini CLI subscriptions rather than
on metered API keys.

It has two halves:

| Path | Runs where | Holds |
|---|---|---|
| `web/` | Vercel | Data. **Never AI credentials.** |
| `runner/` | The user's machine | Credentials, and the CLI processes |

The web app never invokes a model. It writes a row into a job queue; the runner
claims it, builds a context pack, runs the right CLI, and writes results back.

<br>

## Setting this up for a user

Run these in order. Steps 1 and 4 need a human — do not pretend otherwise.

### 1. Sign in to Supabase (human required)

```bash
supabase login                   # opens a browser; only they can finish it
```

If `supabase` is missing, follow the [official OS-specific installation guide](https://supabase.com/docs/guides/local-development/cli/getting-started) and make it available on PATH.

Do not ask them for a project ref or an anon key. The wizard finds both.

Never ask for, accept, or write down the `service_role` key. This project does
not use it anywhere, and `web/lib/env.ts` fails the build if a key like it is
present.

### 2. Run the wizard

```bash
node scripts/setup.mjs --yes
```

`--yes` makes it fully non-interactive, which is what you want with no tty.
It checks tooling, reuses their existing Supabase project or creates
`career-atelier` and waits for it to become healthy, reads the anon key from
the CLI, applies every migration, and writes `web/.env.local` and
`runner/.env`.

`--new-project <name>` forces a fresh project, `--region` defaults to
`ap-northeast-2`, and `--project-ref` with `--anon-key` skips discovery when
the user hands you the values.

For an existing configured installation, `npm start` fingerprints the local
migration set before launching services. When it changes after an update, the
launcher runs the wizard's migration-only path against the project already in
the local env files; it never creates a replacement project during this path.

### 3. Install dependencies

```bash
cd web
npm install
cd ../runner
npm install
```

### 4. Hand back to the human

Two things you cannot do:

- **First sign-up.** Have them run `npm start` from the repository root on Windows, macOS, or Linux. It prepares dependencies and starts the web app and runner together. Then the
  human opens http://localhost:3000 and creates an account with their own email
  and a password. **The first account to sign up becomes the owner of that
  instance and every later signup is rejected**, so this must be them.
- **Runner login and approval.** The same `npm start` terminal requests login if its session is missing or expired. It needs the
  email and password they chose in the web signup form. The password input is hidden.
  Supabase dashboard and database passwords are different credentials. They approve the
  device in the dashboard's runner list.

Ctrl+C stops both local services. For a deployed web app, `npm run runner` starts only the local runner. Existing subdirectory commands remain available for development and unattended use.

Agents also need their own CLI subscriptions signed in (`codex login`,
`claude auth login`, `agy`). Those are the human's accounts; do not attempt to
authenticate as them.

<br>

## Verifying your changes

Run this from the repository root before you report anything:

```bash
npm run verify
```

It runs, in order: version consistency, the project rules, the tooling and
runner tests, and `web`'s typecheck, lint, and build. To run only the rules
while you work — they are fast and need no install:

```bash
npm run rules                          # everything checkable from the tree
npm run rules -- --base origin/main    # adds the history-aware checks CI runs
npm run rules -- --list                # what the rules are
```

Each rule prints the file, the problem, and the fix. They are the conventions
below turned into executable checks, so an agent that cannot run the app can
still prove it did not break the invariants. Every rule is described in
[docs/AGENT-RULES.md](docs/AGENT-RULES.md), which is generated from
`scripts/lib/rules.mjs` — change the rule, then run `npm run rules:docs`.

The web checks on their own, if that is all you touched:

```bash
cd web
npx tsc --noEmit
npm run lint
npm run build
```

All of these must pass. **They are not sufficient.** This project's bug history is
mostly defects that passed every static check and only appeared when someone ran
the thing: a context pack destructuring a key that did not exist, a CLI flag
documented to take a file path that actually takes a JSON string, a schema that
was valid JSON but rejected by the provider's API.

When you report what you did, say what you actually exercised. If you could not
run something, say that instead of implying you did.

<br>

## Conventions

**Migrations are append-only.** Add `supabase/migrations/00NN_name.sql`; never
edit an applied file. After a schema change:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tkv00/Career-Atelier-AI-Context-Pack](https://github.com/tkv00/Career-Atelier-AI-Context-Pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
