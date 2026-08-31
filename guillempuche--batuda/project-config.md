---
trigger: always_on
description: Rules and patterns for AI agents working in this codebase.
---

# AGENTS.md — AI Agent Instructions

Rules and patterns for AI agents working in this codebase.
For project overview, stack, and setup see [README.md](README.md).

---

## Never reveal a secret's value

An API key, token, password or connection string must never be printed, echoed, written to a file, pasted into a message, or put in a command's arguments. This holds even when the value is already on the machine, even when a tool offers it, and even when reading it looks like the quickest way to check something.

Two traps have caught this before:

- **A tool that prints values by default.** `infisical secrets` writes every key out in full and has no redacted mode. Its structured output is preceded by a shell banner, so a naive parse comes back empty and tempts you into the plain form. To find out which secrets exist, read their **names**: `infisical secrets --env=<env> | awk -F'│' 'NF>2 {print $2}'`.
- **Arguments, not just output.** `pnpm` echoes the command line it received, so a key passed as a flag ends up in the log. Pass it through the environment instead (`VAR="$SRC" pnpm …`), never as `--api-key <value>`.

Reading a secret is essentially never necessary. `infisical run -- <command>` injects them into the child process, which is what every script and every check should rely on. If something appears to need the value itself, say so and let the person handle it rather than printing it to find out.

If a value does get exposed, stop, say plainly which ones and where, and tell the person to rotate them. A leaked key that nobody is told about is far worse than one that is.

---

## Dev environment

This project uses a **Nix flake** (`flake.nix`) with **direnv** (`.envrc`) to provide Node 24 and pnpm. On entering the directory, direnv activates the flake automatically — no manual `nix develop` needed.

If you need to verify the environment: `node --version` and `pnpm --version`.

Never install Node or pnpm globally or via other package managers for this project.

---

## Testing & debugging the frontend

Use **`agent-browser`** (already installed) to test and debug the local web app.

Dev URLs (via portless): **Batuda web app `https://batuda.localhost`** and API server `https://api.batuda.localhost`. Most debugging is against the web app.

### Dev login

`pnpm cli seed` (any preset) creates the persona set below. Sign in as Alice for the default dev workflow:

| Email                   | Password          | Platform role | Memberships                         |
| ----------------------- | ----------------- | ------------- | ----------------------------------- |
| `admin@taller.cat`      | `batuda-dev-2026` | admin         | Taller (owner), Restaurant (member) |
| `colleague@taller.cat`  | `batuda-dev-2026` | user          | Taller (member)                     |
| `admin@restaurant.demo` | `batuda-dev-2026` | admin         | Restaurant (owner)                  |

Switch to Bob to verify cross-org isolation; switch to Carol to test the per-user privacy predicate on shared inboxes. The personas are exported from `apps/cli/src/commands/seed.ts` as `DEMO_ORGS`, `DEMO_USERS`, `DEMO_MEMBERSHIPS` (with `TEST_USER` aliased to Alice for backward compatibility). If sign-in fails, re-run `pnpm cli db reset && pnpm cli seed` — the seed is idempotent and (re)creates the personas via Better Auth's admin API.

Multi-org users (Alice) without an `activeOrganizationId` hit 403. The session-create hook auto-picks for single-org users; for Alice, set the active org explicitly:

```bash
curl -X POST https://api.batuda.localhost/auth/organization/set-active \
  -H 'content-type: application/json' \
  --cookie-jar cookies.txt --cookie cookies.txt \
  -d '{"organizationSlug":"taller"}'
```

Better Auth lives on the API server at `api.batuda.localhost/auth/*`. The web browser sends cookies cross-origin via `credentials: 'include'`; on SSR, loaders forward the incoming `cookie` header server-to-server. Both flows require the API server (`pnpm dev:server`) to be running.

### Open & navigate

```bash
agent-browser open https://batuda.localhost              # open pipeline dashboard
agent-browser open https://batuda.localhost/companies    # company list
agent-browser open https://batuda.localhost/tasks        # tasks view
agent-browser back                                    # go back
agent-browser reload                                  # reload page
```

### Inspect page state

```bash
agent-browser snapshot                  # accessibility tree with refs (best for AI)
agent-browser screenshot                # screenshot to stdout
agent-browser screenshot /tmp/page.png  # screenshot to file
agent-browser get text "h1"             # read heading text
agent-browser get text "[data-testid='company-name']"
agent-browser get url                   # current URL
agent-browser get title                 # page title
agent-browser get html ".pipeline"      # HTML of an element
```

### Interact with UI

```bash
agent-browser click "button:has-text('Add company')"
agent-browser fill "input[name='name']" "Can Joan"
agent-browser fill "input[name='slug']" "can-joan"
agent-browser select "select[name='status']" "prospect"
agent-browser press Enter
agent-browser check "input[name='is_decision_maker']"
agent-browser scroll down 300

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guillempuche/batuda](https://github.com/guillempuche/batuda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
